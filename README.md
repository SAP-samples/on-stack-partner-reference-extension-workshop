# On-Stack Partner Reference Extension Workshop

## Description

This repository contains the materials for extending **SAP S/4HANA Cloud Public Edition** using the **ABAP RESTful Application Programming Model (RAP)** and the **ABAP Cloud development model**. Based on the Sales Order Priority extension scenario, this workshop guides you through how to build a partner extension that adds priority-based processing logic, business configuration, and notifications to the standard Sales Order business object. It also covers how to ship it as a partner software component product.

## Overview

In this session you will create your own on-stack ABAP partner extension called **Sales Order Priority Extension**, learn how to package it as a software component, deploy it through the partner build pipeline, and publish it as a product.

Imagine you're a partner consultant, and your job is to extend the standard SAP Sales Order process with custom **priority handling**. Depending on the priority assigned to a sales order and its net amount, the system must automatically determine an **action** (Standard, Review, Manager, Escalation, or Critical), log it, and notify the sold-to party via email. Critical and Escalation actions also block delivery automatically.

As your customer runs its business on **SAP S/4HANA Cloud Public Edition**, you use the **ABAP Cloud development model** to add this logic in a clean-core, upgrade-safe way. The whole extension is packaged into a software component that you can build, version, and ship to multiple downstream systems through the partner pipeline.

## Requirements

To follow the exercises in this repository, you need:

- Access to an **SAP S/4HANA Cloud Public Edition** development system with the ABAP Development Tools (ADT) for Eclipse installed.
- A registered **partner namespace** (`/PW#/` in this workshop, where `#` is your system number — `PW1`, `PW2`, or `PW3`).
- Access to the **Manage Software Components**, **Custom Business Configurations**, **Export Customizing Transports**, **Maintain Business Roles**, **Import Collection**, **Build Product Version**, and **Publish Product** Fiori apps.
- Access to the **Landscape Portal**.
- A **two-digit partner/group number** (`##`) that is used throughout the exercises as a placeholder in all object names.

> **Note:** The software component, package, priority domain & data element, append structure, extension views, and priority value help are **pre-provisioned for you** before the workshop. You start directly from **Exercise 1**.

## Objectives

At the end of this workshop, you will be able to:

- Create a custom business configuration (CUBCO) entry with a RAP-generated business object and service binding, and set up the required IAM app, business catalog, and business role so a configuration expert can maintain priority-to-action rules from the **Custom Business Configurations** app
- Create a Sales Order log table, generate its OData UI service RAP BO, and enhance the generated views with associations and UI annotations
- Implement the strategy pattern (action classes, factory, and event handler) that reacts to Sales Order `created` / `changed` RAP events and automatically dispatches the right action based on the configured rules
- Schedule an application job that sends customers an HTML email notification summarizing the priority, action, and net amount for each processed sales order
- Ship the extension as a partner product by releasing transports, building a product version through the partner pipeline, and publishing it on the active channel

## Exercises

### Exercise 1 — Business Configuration Maintenance Object

Build the priority-to-action configuration table and ship it as a CUBCO entry.

- [Exercise 1 — Business Configuration Maintenance Object](Tutorials/01_Exercise.md)

### Exercise 2 — Log Table & RAP Object Generation

Create the Sales Order log table and scaffold its OData UI service.

- [Exercise 2 — Log Table & RAP Object Generation](Tutorials/02_Exercise.md)

### Exercise 3 — Business Logic Creation

Implement the strategy / factory / event handler that reacts to Sales Order events.

- [Exercise 3 — Business Logic Creation](Tutorials/03_Exercise.md)

### Exercise 4 — Application Job

Send email notifications to customers via a scheduled application job.

- [Exercise 4 — Application Job](Tutorials/04_Exercise.md)

### Exercise 5 — Deploy an SAP Fiori App Using Business Application Studio

Build and deploy a Fiori Elements app from BAS and expose it in the Launchpad.

- [Exercise 5 — Deploy an SAP Fiori App Using Business Application Studio](Tutorials/05_Exercise.md)

### Exercise 6 — SAPUI5 Adaptation (Optional)

Adapt a standard SAP Fiori application using a SAPUI5 Adaptation Project.

- [Exercise 6 — SAPUI5 Adaptation (Optional)](Tutorials/06_Exercise.md)

### Exercise 7 — End-to-End Scenario Test

Run the complete scenario end to end and verify each part works as expected.

- [Exercise 7 — End-to-End Scenario Test](Tutorials/07_Exercise.md)

### Exercise 8 — Release, Build & Publish Product

Release transports, build the product version, and publish.

- [Exercise 8 — Release, Build & Publish Product](Tutorials/08_Exercise.md)

---

> **Note:** Replace `PW#` with the namespace of your system (`PW1`, `PW2`, or `PW3`) and `##` with your two-digit partner/group number wherever it appears (e.g., `09`).
