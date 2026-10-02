# Exercise 4 — Application Job

This exercise creates a recurring Application Job that sends customers email notifications with the Sales Order processing outcome (priority, action applied, net amount). It covers the ABAP class implementing the job logic, the job catalog entry and job template and the IAM/authorization setup.

> **Note:** Replace `PW#` with the namespace of your system (`PW1`, `PW2`, or `PW3`) and `##` with your two-digit partner/group number wherever it appears (e.g., `09`).

---

## Create Email Notification Class

1. Create a class to send the email notification:
   - **Name:** `/PW#/P##_CL_EMAIL_JOB`
   - **Description:** P## Application Job to send Email Notifications

2. Replace the content with the following code:

   ```abap
   CLASS /pw#/p##_cl_email_job DEFINITION
     PUBLIC
     FINAL
     CREATE PUBLIC.

     PUBLIC SECTION.
       INTERFACES if_apj_rt_run.

     PROTECTED SECTION.
     PRIVATE SECTION.
       TYPES: tt_so_lg TYPE STANDARD TABLE OF /pw#/p##_log WITH DEFAULT KEY.
       METHODS:
         get_pending_entries
           RETURNING VALUE(rt_entries) TYPE tt_so_lg,
         send_emails
           IMPORTING it_entries     TYPE tt_so_lg
           RETURNING VALUE(rt_sent) TYPE tt_so_lg,
         mark_email_sent
           IMPORTING it_entries TYPE tt_so_lg.

   ENDCLASS.


   CLASS /pw#/p##_cl_email_job IMPLEMENTATION.

     METHOD if_apj_rt_run~execute.
       DATA(lt_entries) = get_pending_entries( ).
       CHECK lt_entries IS NOT INITIAL.
       DATA(lt_sent) = send_emails( lt_entries ).
       CHECK lt_sent IS NOT INITIAL.
       mark_email_sent( lt_sent ).
     ENDMETHOD.


     METHOD get_pending_entries.
       SELECT lg~*
         FROM /pw#/p##_log AS lg
         WHERE lg~email_sent = @abap_false
           AND lg~sold_to_party IS NOT INITIAL
         INTO CORRESPONDING FIELDS OF TABLE @rt_entries.
     ENDMETHOD.


     METHOD send_emails.
       DATA(lv_date) = cl_abap_context_info=>get_system_date( ).

       IF it_entries IS NOT INITIAL.
         SELECT DISTINCT
                entries~sold_to_party AS businesspartner,
                coalesce( wa~defaultemailaddress, std_email~emailaddress ) AS defaultemailaddress
           FROM @it_entries AS entries
           INNER JOIN i_businesspartner WITH PRIVILEGED ACCESS AS bp
             ON entries~sold_to_party = bp~businesspartner
           LEFT OUTER JOIN i_workplaceaddress WITH PRIVILEGED ACCESS AS wa
             ON bp~businesspartner = wa~businesspartner
           LEFT OUTER JOIN i_buspartaddress WITH PRIVILEGED ACCESS AS bp_adr
             ON bp~businesspartner = bp_adr~businesspartner
           LEFT OUTER JOIN i_addressemailaddress_2 WITH PRIVILEGED ACCESS AS std_email
             ON bp_adr~addressid = std_email~addressid
             AND std_email~emailaddressiscurrentdefault = @abap_true
           INTO TABLE @DATA(lt_emails).
       ENDIF.

       SELECT value_low, text
         FROM /pw#/p##_i_priority_vh
         WHERE language = @sy-langu
         INTO TABLE @DATA(lt_priority_texts).

       SELECT value_low, text
         FROM /pw#/p##_i_action_vh
         WHERE language = @sy-langu
         INTO TABLE @DATA(lt_action_texts).

       LOOP AT it_entries INTO DATA(ls_entry).
         DATA(lv_email)    = VALUE #( lt_emails[ businesspartner = ls_entry-sold_to_party ]-defaultEmailAddress OPTIONAL ).
         DATA(lv_priority) = VALUE #( lt_priority_texts[ value_low = ls_entry-priority ]-text OPTIONAL ).
         DATA(lv_action)   = VALUE #( lt_action_texts[ value_low = ls_entry-action_key ]-text OPTIONAL ).

         IF lv_priority IS INITIAL. lv_priority = ls_entry-priority. ENDIF.
         IF lv_action IS INITIAL.   lv_action   = ls_entry-action_key. ENDIF.

         CHECK lv_email IS NOT INITIAL.

         TRY.
             DATA(lo_mail) = cl_bcs_mail_message=>create_instance( ).
             lo_mail->add_recipient( CONV #( lv_email ) ).
             lo_mail->set_subject( |P## Sales Order { ls_entry-sales_order } — Action Notification| ).

             DATA(lv_html) =
               |<html><body style="font-family:Arial,sans-serif;color:#333;">| &&
               |<div style="max-width:600px;margin:0 auto;padding:20px;">| &&
               |<div style="background:#1B4F8A;color:white;padding:16px;border-radius:4px 4px 0 0;">| &&
               |  <h2 style="margin:0;">Sales Order Processing Notification</h2>| &&
               |</div>| &&
               |<div style="background:#f9f9f9;padding:24px;border-radius:0 0 4px 4px;">| &&
               |  <p>Dear Customer,</p>| &&
               |  <p>Your Sales Order has been processed. Below are the details:</p>| &&
               |  <table style="width:100%;border-collapse:collapse;margin:16px 0;">| &&
               |    <tr style="background:#e8f0fb;"><td style="padding:8px 12px;font-weight:bold;">Sales Order</td>| &&
               |      <td style="padding:8px 12px;">{ ls_entry-sales_order }</td></tr>| &&
               |    <tr><td style="padding:8px 12px;font-weight:bold;">Priority</td>| &&
               |      <td style="padding:8px 12px;">{ lv_priority }</td></tr>| &&
               |    <tr style="background:#e8f0fb;"><td style="padding:8px 12px;font-weight:bold;">Action Applied</td>| &&
               |      <td style="padding:8px 12px;">{ lv_action }</td></tr>| &&
               |    <tr><td style="padding:8px 12px;font-weight:bold;">Net Amount</td>| &&
               |      <td style="padding:8px 12px;">{ ls_entry-net_amount } { ls_entry-currency }</td></tr>| &&
               |  </table>| &&
               |  <p>Thank you for your business.</p>| &&
               |  <p style="color:#666;font-size:11px;">This is an automated message. Please do not reply.</p>| &&
               |</div></div></body></html>|.

             lo_mail->set_main( cl_bcs_mail_textpart=>create_text_html( lv_html ) ).

             DATA(lo_status) = lo_mail->send( ).

             " Check recipient email status — only mark sent if accepted
             lo_status->get_email_status(
               IMPORTING
                 et_recipients_statuses = DATA(lt_recipient_status)
             ).

             DATA(lv_accepted) = abap_false.
             LOOP AT lt_recipient_status INTO DATA(ls_status).
               IF ls_status-status = 'S'.
                 lv_accepted = abap_true.
               ENDIF.
             ENDLOOP.

             IF lv_accepted = abap_true.
               APPEND ls_entry TO rt_sent.
             ENDIF.

           CATCH cx_bcs_mail.
             CONTINUE.
         ENDTRY.

       ENDLOOP.
     ENDMETHOD.


     METHOD mark_email_sent.
       DATA(lt_update) = it_entries.
       LOOP AT lt_update ASSIGNING FIELD-SYMBOL(<ls>).
         <ls>-email_sent = abap_true.
       ENDLOOP.
       UPDATE /pw#/p##_log FROM TABLE @lt_update.
       COMMIT WORK.
     ENDMETHOD.

   ENDCLASS.
   ```

3. Ensure that your partner/group number is updated in the email subject in method - `send_emails`.

4. Save and Activate.

   > **Note:** Replace `##` with your two-digit partner/group number everywhere it appears in the code above — including the email subject (`P## Sales Order ...`), the table type, the `SELECT` statements and the `UPDATE` statement — so all object names and text resolve to your objects.

---

## Application Job Creation

### Create Job Catalog Entry

After implementing the ABAP class, a job catalog entry must be created.

An application job catalog entry contains:

- The name of the ABAP class that implements the business logic
- Information about how selection fields are rendered in the scheduling dialog
- Additional settings required for scheduling and execution

1. Right-click on the package and choose **New → Other ABAP Repository Object**.

2. Search for and choose **Application Job Catalog Entry**.

3. Provide:
   - **Name:** `/PW#/P##_EMAIL_JOB_CE`
   - **Description:** P## Application Job Catalog Entry
   - **Class with Execute Method:** `/PW#/P##_CL_EMAIL_JOB`

4. Click **Next**, then click **Finish**.

5. Activate the created job catalog entry.

### Create Job Template

An application job template refers to an application job catalog entry and contains values for some or all selection fields.

A job template can be considered a variant of a job catalog entry. It may:

- Contain no values (initial values are used)
- Contain partial or full parameter values

A job catalog entry can have multiple job templates.

1. Right-click on the package and choose **New → Other ABAP Repository Object**.

2. Search for and choose **Application Job Template**.

3. Provide:
   - **Name:** `/PW#/P##_EMAIL_JOB_JT`
   - **Description:** P## Application Job Template
   - **Job Catalog Entry:** `/PW#/P##_EMAIL_JOB_CE`

4. Click **Next**, then click **Finish**.

5. Activate the created job template.

---

## Setting Up the Authorizations

When you create a job catalog entry and a job template as explained above, an object of type **IAM App** is created automatically. It has the name `<job catalog entry name>_SAJC`. This IAM app contains the start authorization for all job templates that refer to this job catalog entry.

1. Open the IAM app `/PW#/P##_EMAIL_JOB_CE_SAJC`. To find it in the **Project Explorer**, expand your package `/PW#/P##_SO_EXT` → **Identity and Access Management** → **IAM Apps** and double-click `/PW#/P##_EMAIL_JOB_CE_SAJC`. In the overview section of the IAM App, click **Create a new Business Catalog and assign the App to it**.

   ![IAM Job](Images/IAM%20job.png)

2. Enter the following details:
   - **Name:** `/PW#/P##_EMAIL_JOB_BC`
   - **Description:** P## Email Business Catalog

   Click **Next**.

3. Click **Finish**.

4. The wizard for creating a Business Catalog App Assignment opens automatically. Provide package name '/PW#/P##_SO_EXT'.

5. Click **Next** and then click **Finish**.

6. Open Business Catalog '/PW#/P##_EMAIL_JOB_BC', click **Publish Locally**. Wait for a few minutes until the status changes to 'Published'.

---

## Assign Business Catalog to the Business Role Template

1. Open Business Role Template '/PW#/P##_BRT_SO_EXT' created in Exercise 1. It is present in your package in 'Identity and Access Management' section.

2. Click **Add** and provide Business Catalog name: `/PW#/P##_EMAIL_JOB_BC`.

   ![Add Business Catalog to existing BRT](Images/Add%20BC%20in%20existing%20BRT.png)

3. Click **Next**.

4. Select the Transport and click **Finish**.

---

**Next:** You created the Application Job that emails customers their Sales Order outcome, including the job class, catalog entry, template and authorization setup. In [Exercise 5 — Deploy an SAP Fiori App Using Business Application Studio](05_Exercise.md), you'll build and deploy the Fiori Elements audit log app on top of the OData service from Exercise 2.
