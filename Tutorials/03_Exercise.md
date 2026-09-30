# Exercise 3 — Business Logic Creation

This exercise implements the business logic that runs when a Sales Order is created or changed: a strategy pattern with one action class per priority level (Standard, Review, Manager, Escalation, Critical), a factory to resolve the right strategy and an event handler that listens to Sales Order RAP events and dispatches the action.

> **Note:** Replace `PW#` with the namespace of your system (`PW1`, `PW2`, or `PW3`) and `##` with your two-digit partner/group number wherever it appears (e.g., `09`).

---

## Create Interface for Action Classes

1. Right-click on the package and choose **New → ABAP Interface**.

2. Provide the following information and then click **Next**:
   - **Name:** `/PW#/P##_IF_ACTION`
   - **Description:** P## Interface for Sales Order Action

3. Click **Finish**.

4. Replace with the following code:

   ```abap
   INTERFACE /pw#/p##_if_action
     PUBLIC .
     METHODS execute
       IMPORTING
         iv_salesorder  TYPE vbeln
         iv_amount      TYPE /pw#/p##_log-net_amount
         iv_currency    TYPE waers
         iv_priority    TYPE /pw#/p##_log-priority
         iv_action      TYPE /pw#/p##_log-action_key
         iv_soldtoparty TYPE /pw#/p##_log-sold_to_party.
   ENDINTERFACE.
   ```

5. Save and activate.

---

## Create Action Classes

### Standard Action

1. Right-click on the package and choose **New → ABAP Class**.

2. Provide the following information and then click **Next**:
   - **Name:** `/PW#/P##_CL_ACT_STANDARD`
   - **Description:** P## Action: Standard Processing

3. Click **Finish**.

4. Replace with the following code:

   ```abap
   CLASS /pw#/p##_cl_act_standard DEFINITION
     PUBLIC
     FINAL
     CREATE PUBLIC .

     PUBLIC SECTION.
       INTERFACES /pw#/p##_if_action.
     PROTECTED SECTION.
     PRIVATE SECTION.
   ENDCLASS.


   CLASS /pw#/p##_cl_act_standard IMPLEMENTATION.

     METHOD /pw#/p##_if_action~execute.

       MODIFY ENTITIES OF /pw#/r_p##_log
         ENTITY /pw#/rP##Log
         CREATE FIELDS ( SalesOrder LogDate LogTime NetAmount Currency Priority SoldtoParty ActionKey )
         WITH VALUE #( ( %cid        = iv_salesorder
                         SalesOrder  = iv_salesorder
                         LogDate     = cl_abap_context_info=>get_system_date( )
                         LogTime     = cl_abap_context_info=>get_system_time( )
                         NetAmount   = iv_amount
                         Currency    = iv_currency
                         SoldtoParty = iv_soldtoparty
                         Priority    = iv_priority
                         ActionKey   = iv_action ) )
         REPORTED DATA(reported)
         FAILED   DATA(failed).

     ENDMETHOD.

   ENDCLASS.
   ```

5. Save and activate.

### Review Action

Similarly create the following class and replace with the code provided.

1. Create class:
   - **Name:** `/PW#/P##_CL_ACT_REVIEW`
   - **Description:** P## Action: Review Processing

2. Replace with the following code:

   ```abap
   CLASS /pw#/p##_cl_act_review DEFINITION
     PUBLIC
     FINAL
     CREATE PUBLIC .

     PUBLIC SECTION.
       INTERFACES /pw#/p##_if_action.
     PROTECTED SECTION.
     PRIVATE SECTION.
   ENDCLASS.

   CLASS /pw#/p##_cl_act_review IMPLEMENTATION.
     METHOD /pw#/p##_if_action~execute.
       MODIFY ENTITIES OF /pw#/r_p##_log
         ENTITY /pw#/rP##Log
         CREATE FIELDS ( SalesOrder LogDate LogTime NetAmount Currency Priority SoldtoParty ActionKey )
         WITH VALUE #( ( %cid        = iv_salesorder
                         SalesOrder  = iv_salesorder
                         LogDate     = cl_abap_context_info=>get_system_date( )
                         LogTime     = cl_abap_context_info=>get_system_time( )
                         NetAmount   = iv_amount
                         Currency    = iv_currency
                         SoldtoParty = iv_soldtoparty
                         Priority    = iv_priority
                         ActionKey   = iv_action ) )
         REPORTED DATA(reported)
         FAILED   DATA(failed).
     ENDMETHOD.
   ENDCLASS.
   ```

3. Save and activate.

### Manager Action

1. Create class:
   - **Name:** `/PW#/P##_CL_ACT_MANAGER`
   - **Description:** P## Action: Manager Processing

2. Replace with the following code:

   ```abap
   CLASS /pw#/p##_cl_act_manager DEFINITION
     PUBLIC
     FINAL
     CREATE PUBLIC .
     PUBLIC SECTION.
       INTERFACES /pw#/p##_if_action.
     PROTECTED SECTION.
     PRIVATE SECTION.
   ENDCLASS.

   CLASS /pw#/p##_cl_act_manager IMPLEMENTATION.
     METHOD /pw#/p##_if_action~execute.
       MODIFY ENTITIES OF /pw#/r_p##_log
         ENTITY /pw#/rP##Log
         CREATE FIELDS ( SalesOrder LogDate LogTime NetAmount Currency Priority SoldtoParty ActionKey )
         WITH VALUE #( ( %cid        = iv_salesorder
                         SalesOrder  = iv_salesorder
                         LogDate     = cl_abap_context_info=>get_system_date( )
                         LogTime     = cl_abap_context_info=>get_system_time( )
                         NetAmount   = iv_amount
                         Currency    = iv_currency
                         SoldtoParty = iv_soldtoparty
                         Priority    = iv_priority
                         ActionKey   = iv_action ) )
         REPORTED DATA(reported)
         FAILED   DATA(failed).
     ENDMETHOD.
   ENDCLASS.
   ```

3. Save and activate.

### Critical Action

1. Create class:
   - **Name:** `/PW#/P##_CL_ACT_CRITICAL`
   - **Description:** P## Action: Critical Processing

2. Replace with the following code:

   ```abap
   CLASS /pw#/p##_cl_act_critical DEFINITION
     PUBLIC
     FINAL
     CREATE PUBLIC .

     PUBLIC SECTION.
       INTERFACES /pw#/p##_if_action.
     PROTECTED SECTION.
     PRIVATE SECTION.
   ENDCLASS.

   CLASS /pw#/p##_cl_act_critical IMPLEMENTATION.
     METHOD /pw#/p##_if_action~execute.
       MODIFY ENTITIES OF I_SalesOrderTP
           ENTITY SalesOrder
           UPDATE FIELDS ( DeliveryBlockReason )
           WITH VALUE #( ( SalesOrder          = iv_salesorder
                           DeliveryBlockReason = '01' ) )
           REPORTED DATA(rep_block)
           FAILED   DATA(fail_block).

       MODIFY ENTITIES OF /pw#/r_p##_log
         ENTITY /pw#/rP##Log
         CREATE FIELDS ( SalesOrder LogDate LogTime NetAmount Currency Priority SoldtoParty ActionKey )
         WITH VALUE #( ( %cid        = iv_salesorder
                         SalesOrder  = iv_salesorder
                         LogDate     = cl_abap_context_info=>get_system_date( )
                         LogTime     = cl_abap_context_info=>get_system_time( )
                         NetAmount   = iv_amount
                         Currency    = iv_currency
                         SoldtoParty = iv_soldtoparty
                         Priority    = iv_priority
                         ActionKey   = iv_action ) )
         REPORTED DATA(reported)
         FAILED   DATA(failed).
     ENDMETHOD.
   ENDCLASS.
   ```

3. Save and activate.

### Escalation Action

1. Create class:
   - **Name:** `/PW#/P##_CL_ACT_ESCALATION`
   - **Description:** P## Action: Escalation Processing

2. Replace with the following code:

   ```abap
   CLASS /pw#/p##_cl_act_escalation DEFINITION
     PUBLIC
     FINAL
     CREATE PUBLIC .

     PUBLIC SECTION.
       INTERFACES /pw#/p##_if_action.
     PROTECTED SECTION.
     PRIVATE SECTION.
   ENDCLASS.

   CLASS /pw#/p##_cl_act_escalation IMPLEMENTATION.
     METHOD /pw#/p##_if_action~execute.
       MODIFY ENTITIES OF I_SalesOrderTP
           ENTITY SalesOrder
           UPDATE FIELDS ( DeliveryBlockReason )
           WITH VALUE #( ( SalesOrder          = iv_salesorder
                           DeliveryBlockReason = '01' ) )
           REPORTED DATA(rep_block)
           FAILED   DATA(fail_block).

       MODIFY ENTITIES OF /pw#/r_p##_log
         ENTITY /pw#/rP##Log
         CREATE FIELDS ( SalesOrder LogDate LogTime NetAmount Currency Priority SoldtoParty ActionKey )
         WITH VALUE #( ( %cid        = iv_salesorder
                         SalesOrder  = iv_salesorder
                         LogDate     = cl_abap_context_info=>get_system_date( )
                         LogTime     = cl_abap_context_info=>get_system_time( )
                         NetAmount   = iv_amount
                         Currency    = iv_currency
                         SoldtoParty = iv_soldtoparty
                         Priority    = iv_priority
                         ActionKey   = iv_action ) )
         REPORTED DATA(reported)
         FAILED   DATA(failed).
     ENDMETHOD.
   ENDCLASS.
   ```

3. Save and activate.

---

## Create Factory Class

Create the factory class to return the correct class instance based on the action key.

1. Create class:
   - **Name:** `/PW#/P##_CL_ACTION_FACTORY`
   - **Description:** P## Factory for Sales Order Action Strategy

2. Replace with the following code:

   ```abap
   CLASS /pw#/p##_cl_action_factory DEFINITION
     PUBLIC
     FINAL
     CREATE PUBLIC .
     PUBLIC SECTION.
       CLASS-METHODS get_strategy
         IMPORTING iv_action_key      TYPE /pw#/p##_log-action_key
         RETURNING VALUE(ro_strategy) TYPE REF TO /pw#/p##_if_action.
     PROTECTED SECTION.
     PRIVATE SECTION.
   ENDCLASS.

   CLASS /pw#/p##_cl_action_factory IMPLEMENTATION.
     METHOD get_strategy.
       CASE iv_action_key.
         WHEN 'STD'.
           ro_strategy = NEW /pw#/p##_cl_act_standard( ).
         WHEN 'REV'.
           ro_strategy = NEW /pw#/p##_cl_act_review( ).
         WHEN 'MGR'.
           ro_strategy = NEW /pw#/p##_cl_act_manager( ).
         WHEN 'ESC'.
           ro_strategy = NEW /pw#/p##_cl_act_escalation( ).
         WHEN 'CRT'.
           ro_strategy = NEW /pw#/p##_cl_act_critical( ).
         WHEN OTHERS.
           ro_strategy = NEW /pw#/p##_cl_act_standard( ).
       ENDCASE.
     ENDMETHOD.
   ENDCLASS.
   ```

3. Save and activate.

---

## Create Event Handler Class

1. Create the event handler class:
   - **Name:** `/PW#/P##_SO_EVENT_HANDLER`
   - **Description:** P## Sales Order Event Handler

2. Replace the **Global class** with the following code:

   ```abap
   CLASS /pw#/p##_so_event_handler DEFINITION
     PUBLIC
     FINAL
     FOR EVENTS OF I_SalesOrderTP.
     PUBLIC SECTION.
     PROTECTED SECTION.
     PRIVATE SECTION.
   ENDCLASS.


   CLASS /pw#/p##_so_event_handler IMPLEMENTATION.
   ENDCLASS.
   ```

3. Go to **Local Types** in the class and paste the following code:

   ```abap
   CLASS lhe_event DEFINITION INHERITING FROM cl_abap_behavior_event_handler.
     PRIVATE SECTION.
       METHODS on_created FOR ENTITY EVENT
          created FOR SalesOrder~created.
       METHODS on_updated FOR ENTITY EVENT
          updated FOR SalesOrder~Changed.
   ENDCLASS.

   CLASS lhe_event IMPLEMENTATION.

     METHOD on_created.
       CHECK created IS NOT INITIAL.

       SELECT SalesOrder,
              TotalNetAmount,
              TransactionCurrency,
              /pw#/p##_priority_sdh,
              SoldToParty,
              OvrlItmGeneralIncompletionSts,
              DeliveryBlockReason
         FROM I_SalesOrderTP
         FOR ALL ENTRIES IN @created
         WHERE SalesOrder = @created-SalesOrder
         INTO TABLE @DATA(lt_so).

       LOOP AT lt_so ASSIGNING FIELD-SYMBOL(<so>).
         CHECK <so>-/pw#/p##_priority_sdh IS NOT INITIAL.
         CHECK <so>-OvrlItmGeneralIncompletionSts = 'C' and <so>-DeliveryBlockReason IS INITIAL.

         DATA(lv_amount) = <so>-TotalNetAmount.

         SELECT SINGLE action_key FROM /pw#/p##_cnfg
           WHERE priority    = @<so>-/pw#/p##_priority_sdh
             AND amount_from <= @lv_amount
             AND ( amount_to = 0 OR amount_to > @lv_amount )
           INTO @DATA(lv_action_key).

         IF sy-subrc <> 0 OR lv_action_key IS INITIAL.
           lv_action_key = 'STD'.
         ENDIF.

         DATA(lo_strategy) = /pw#/p##_cl_action_factory=>get_strategy( lv_action_key ).

         lo_strategy->execute(
             iv_salesorder  = <so>-SalesOrder
             iv_amount      = lv_amount
             iv_currency    = <so>-TransactionCurrency
             iv_priority    = <so>-/pw#/p##_priority_sdh
             iv_action      = lv_action_key
             iv_soldtoparty = <so>-SoldToParty ).
       ENDLOOP.
     ENDMETHOD.

     METHOD on_updated.
       CHECK updated IS NOT INITIAL.

       SELECT SalesOrder,
              TotalNetAmount,
              TransactionCurrency,
              /pw#/p##_priority_sdh,
              SoldToParty,
              OvrlItmGeneralIncompletionSts,
              DeliveryBlockReason
         FROM I_SalesOrderTP
         FOR ALL ENTRIES IN @updated
         WHERE SalesOrder = @updated-SalesOrder
         INTO TABLE @DATA(lt_so).

       LOOP AT lt_so ASSIGNING FIELD-SYMBOL(<so>).
         CHECK <so>-/pw#/p##_priority_sdh IS NOT INITIAL.
         CHECK <so>-OvrlItmGeneralIncompletionSts = 'C' and <so>-DeliveryBlockReason IS INITIAL.

         DATA(lv_amount) = <so>-TotalNetAmount.

         SELECT SINGLE action_key FROM /pw#/p##_cnfg
           WHERE priority    = @<so>-/pw#/p##_priority_sdh
             AND amount_from <= @lv_amount
             AND ( amount_to = 0 OR amount_to > @lv_amount )
           INTO @DATA(lv_action_key).

         IF sy-subrc <> 0 OR lv_action_key IS INITIAL.
           lv_action_key = 'STD'.
         ENDIF.

         DATA(lo_strategy) = /pw#/p##_cl_action_factory=>get_strategy( lv_action_key ).

         lo_strategy->execute(
             iv_salesorder  = <so>-SalesOrder
             iv_amount      = lv_amount
             iv_currency    = <so>-TransactionCurrency
             iv_priority    = <so>-/pw#/p##_priority_sdh
             iv_action      = lv_action_key
             iv_soldtoparty = <so>-SoldToParty ).
       ENDLOOP.
     ENDMETHOD.

   ENDCLASS.
   ```

4. Save and activate.

---

**Next:** You implemented the business logic (action classes, factory and event handler) that reacts to Sales Order events and records each outcome in the log. In [Exercise 4 — Application Job](04_Exercise.md), you'll create a recurring Application Job that emails customers their Sales Order processing outcome.
