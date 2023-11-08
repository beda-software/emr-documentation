---
sidebar_position: 3
---

# Architecture

The core component of EMR is SDC IG. FHIR Questionnaires are used to describe all forms in the system. For Questionnaire it is important to customize the appearance and behavior of item controls. Storybook is used to isolate item control, develop, and test it in a separate environment. It is important to provide an easy way to build item controls with modern mainstream tools like TypeScript, React, Styled components, etc.

Dashboard, chatting, and interface widgets are React components that interact with FHIR API. At the current stage of the project, they are not configurable. 
If customization is needed these dashboards and widgets will be reimplemented from scratch to meet end-user needs.

Each user action goes through `$extract` SDC operation that converts it into a Bundle transaction that alters resources in FHIR storage. For example, side effects like email notifications could be created in the mapper. For example, when you need to send an email, you can add it as a custom resource into a Bundle transaction. Another approach for side effect management is FHIR Subscriptions. They are used for backend services to react to data changes and perform some actions like sending a request to an insurance company when a claim is created.
Subscriptions may trigger a more complex logic in 3rd-party modules. These modules create FHIR resources during their work.

We prefer to stick as close as possible to tools driven by the FHIR community. For example, we are using TestScript to test and validate Questionnaire behavior. 
However, in some cases, we have to step out from the standard. An example is the FHIR Mapping language, we are going to add extraction with FHIR Mapping language but the preferable way is our custom solution https://github.com/beda-software/fhirpathmappinglanguage.

## Services
Beda EMR requires several services to be deployed:
Component|Designation
---------|-----------
FHIR server|Any FHIR server that will provide full-features FHIT API and persistence layer
[Beda EMR](https://github.com/beda-software/fhir-emr)|Frontend that turns FHIR server into EMR
[Scheduling](https://github.com/beda-software/aidbox-scheduling-node-app)|Inspired by Argo Scheduling
[SDC IDE](https://github.com/beda-software/sdc-ide)|Development environment for Questionnaires and Mappers
[FHIR SDC](https://github.com/beda-software/fhir-sdc)|Backend modules for SDC, include population and extraction
[mHealth](https://github.com/beda-software/fhir-mhealth)|iOS application that loads activity data via Apple Health
[data sequence](https://github.com/beda-software/fhir-datasequence)|A backend that saves activities from Apple Health into TimescaleDB

Please check [docker compose configuration](https://github.com/beda-software/fhir-emr/blob/master/compose.yaml) for more details.
