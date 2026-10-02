# Exercise 2 — Log Table & RAP Object Generation

This exercise creates a Sales Order Log table that records each automated action performed by the system, then uses the ABAP Repository Generator to scaffold the RAP business object and OData UI service on top of it.

> **Note:** Replace `PW#` with the namespace of your system (`PW1`, `PW2`, or `PW3`) and `##` with your two-digit partner/group number wherever it appears (e.g., `09`).

---

## Create Data Elements

### Log Date

1. Create the following data element for log date and provide the field label as **Created On**:
   - **Name:** `/PW#/P##_LOG_DATE`
   - **Description:** P## Log Date
   - **Category:** Predefined Type
   - **Data Type:** DATS
   - **Field Labels:**

     | Label   | Text       |
     | :------ | :--------- |
     | Short   | Created On |
     | Medium  | Created On |
     | Long    | Created On |
     | Heading | Created On |

2. Save and activate.

### Log Time

1. Create the following data element for log time and provide the field label as **Created At**:
   - **Name:** `/PW#/P##_LOG_TIME`
   - **Description:** P## Log Time
   - **Category:** Predefined Type
   - **Data Type:** TIMS
   - **Field Labels:**

     | Label   | Text       |
     | :------ | :--------- |
     | Short   | Created At |
     | Medium  | Created At |
     | Long    | Created At |
     | Heading | Created At |

2. Save and activate.

### Sold-to Party

1. Create the following data element for sold-to party:
   - **Name:** `/PW#/P##_SOLDTOPARTY`
   - **Description:** P## Sold-to Party
   - **Category:** Predefined Type
   - **Data Type:** CHAR
   - **Length:** 10
   - **Field Labels:**

     | Label   | Text          |
     | :------ | :------------ |
     | Short   | ID            |
     | Medium  | Sold-to Party |
     | Long    | Sold-to Party |
     | Heading | Sold-to Party |

2. Save and activate.

---

## Create Log Table

1. Right-click on the Package name. Choose **New → Other ABAP Repository Object** and search for and select **Database Table**.

2. Create a new Table:
   - **Name:** `/PW#/P##_LOG`
   - **Description:** P## Sales Order Log

   Click **Next**.

3. Click **Finish**.

4. Replace your code as follows:

   This table records one row per automated action: the Sales Order number, timestamp, net amount, currency, priority, resolved action, sold-to party and an `email_sent` flag.

   ```cds
   @EndUserText.label : 'Sales Order Log'
   @AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
   @AbapCatalog.tableCategory : #TRANSPARENT
   @AbapCatalog.deliveryClass : #A
   @AbapCatalog.dataMaintenance : #RESTRICTED
   define table /pw#/p##_log {

     key client            : abap.clnt not null;
     key sales_order       : vbeln not null;
     key log_date          : /pw#/p##_log_date not null;
     key log_time          : /pw#/p##_log_time not null;
     net_amount            : /pw#/p##_amount;
     currency              : waers;
     priority              : /pw#/p##_priority;
     action_key            : /pw#/p##_action;
     sold_to_party         : /pw#/p##_soldtoparty;
     email_sent            : abap_boolean;
     created_by            : abp_creation_user;
     last_changed_by       : abp_lastchange_user;
     last_changed_at       : abp_lastchange_tstmpl;
     local_last_changed_at : abp_locinst_lastchange_tstmpl;

   }
   ```

5. Save and activate.

---

## Generate RAP Objects

1. Right-click on the Table and choose **Generate ABAP Repository Objects**.

2. Choose **OData UI Service**. Click **Next**.

   ![RAP BO Generation 1](Images/RAP%20BO%20Generation%201.png)

3. Provide your Package name `/PW#/P##_SO_EXT`. Click **Next**.

4. Click **Next** on the Artifacts name page.

   ![RAP BO Generation 2](Images/RAP%20BO%20Generation%202.png)

5. Click **Next** and **Finish**.

6. Open generated Service Definition '/PW#/UI_P##_LOG_O4' present under 'Business Services' and add alias 'as P##_LOG'.

   ![Service Definition](Images/Service%20Definition.png)

7. Save and activate.

### Modify Generated R View

1. Open the generated R view `/PW#/R_P##_LOG`. To find it in the **Project Explorer**, expand your package `/PW#/P##_SO_EXT` → **Core Data Services** → **Data Definitions** and double-click `/PW#/R_P##_LOG`. Then add the following associations:

   These associations join the log to the Priority and Action value-help views (for their texts) and to the business user (for the creator's name), so the app can display descriptive texts and the creator instead of raw codes.

   ```cds
   association [1..*] to /PW#/P##_I_Priority_VH as _PriorityText on _PriorityText.domain_name = '/PW#/P##_PRIORITY'
                                                  and $projection.Priority = _PriorityText.value_low
   association [1..*] to /PW#/P##_I_Action_VH as _ActionText on _ActionText.domain_name = '/PW#/P##_ACTION'
                                                  and $projection.ActionKey = _ActionText.value_low
   association [1..1] to I_BusinessUserBasic as _CreatedByUser
       on $projection.CreatedBy = _CreatedByUser.UserID
   ```

2. Expose the above created associations:

   Exposing them makes these associations available to the consumption (projection) layer.

   ```cds
   _PriorityText,
   _ActionText,
   _CreatedByUser
   ```

   Now, the R view looks as shown below:

   ![R View 2](Images/R%20View%202.png)

3. Save and activate.

### Modify Generated Consumption View

1. Open the generated Consumption view `/PW#/C_P##_LOG`.

   Replace the block containing fields with the following code:

   ```cds
   key SalesOrder,
   key LogDate,
   key LogTime,
   NetAmount,
   Currency,
   @ObjectModel.text.element: [ 'PriorityText' ]
   Priority,
   @ObjectModel.text.element: [ 'ActionText' ]
   ActionKey,
   SoldToParty,
   EmailSent,
   @Semantics: {
     user.createdBy: true
   }
   @ObjectModel.text.element: [ 'Name' ]
   CreatedBy,
   @Semantics: {
     user.lastChangedBy: true
   }
   LastChangedBy,
   @Semantics: {
     systemDateTime.lastChangedAt: true
   }
   LastChangedAt,
   @Semantics: {
     systemDateTime.localInstanceLastChangedAt: true
   }
   LocalLastChangedAt,
   _ActionText.text as ActionText : localized,
   _PriorityText.text as PriorityText : localized,
   _CreatedByUser.PersonFullName as Name,
   _BaseEntity
   
   ```

2. Save and activate.

### Modify Metadata Extension

1. Open the Metadata Extension `/PW#/C_P##_LOG`. To find it in the **Project Explorer**, expand your package `/PW#/P##_SO_EXT` → **Core Data Services** → **Metadata Extensions** and double-click `/PW#/C_P##_LOG`. Then replace it with the following code:

    ```cds
    @Metadata.layer: #CORE
    @UI.headerInfo.title.type: #STANDARD
    @UI.headerInfo.title.value: 'SalesOrder'
    @UI.headerInfo.description.type: #STANDARD
    @UI.headerInfo.description.value: 'SalesOrder'
    annotate view /PW#/C_P##_LOG with
    {
      @UI.facet: [ {
        label: 'General Information',
        id: 'GeneralInfo',
        purpose: #STANDARD,
        position: 10 ,
        type: #IDENTIFICATION_REFERENCE
      } ]
      @UI: {
        identification: [{ position: 10 }],
        lineItem:       [{ position: 10 }],
        selectionField: [{ position: 10 }]
      }
      SalesOrder;

      @UI: {
        identification: [{ position: 20 }],
        lineItem:       [{ position: 20 }],
        selectionField: [{ position: 20 }]
      }
      LogDate;

      @UI: {
        identification: [{ position: 30 }],
        lineItem:       [{ position: 30 }],
        selectionField: [{ position: 30 }]
      }
      LogTime;

      @UI: {
        identification: [{ position: 40 }],
        lineItem:       [{ position: 40 }],
        selectionField: [{ position: 40 }]
      }
      NetAmount;

      @UI: {
        identification: [{ position: 50 }],
        lineItem:       [{ position: 50 }],
        selectionField: [{ position: 50 }]
      }
      Currency;

      @UI: {
        identification: [{ position: 60 }],
        lineItem:       [{ position: 60 }],
        selectionField: [{ position: 60 }]
      }
      Priority;

      @UI: {
        identification: [{ position: 70 }],
        lineItem:       [{ position: 70 }],
        selectionField: [{ position: 70 }]
      }
      ActionKey;

      @UI: {
        identification: [{ position: 80 }],
        lineItem:       [{ position: 80 }],
        selectionField: [{ position: 80 }]
      }
      SoldToParty;

      @UI.hidden: true
      EmailSent;

      @UI.hidden: true
      CreatedBy;

      @UI.hidden: true
      LastChangedBy;

      @UI.hidden: true
      LastChangedAt;

      @UI.hidden: true
      LocalLastChangedAt;

      @UI.hidden: true
      _BaseEntity;
    }
    ```

2. Save and activate.

### Publish Service Binding

1. Publish the generated service binding `/PW#/UI_P##_LOG_O4`.
2. Once published, choose the available entity set and click **Preview** to open the Fiori preview app.

---

**Next:** You built the Sales Order Log table and generated its RAP business object and OData UI service to store each automated action. In [Exercise 3 — Business Logic Creation](03_Exercise.md), you'll implement the action classes, factory and event handler that write to this log when a Sales Order is created or changed.
