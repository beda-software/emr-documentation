---
sidebar_position: 2
---


# Features

- Appointment and Encounters (visits management, scheduling)
- Electronic Medical Records
  - based on Questionnaire and QuestionnaireResponse resources
  - Questionnaire population, initial and calculated expressions
  - extraction FHIR data from QuestionnaireResponse on save
- EMR Questionnaire form builder
- HealthcareService management
- Invoice management
- Medication management
  - Warehouse management
  - Prescriptions management
- Patient medical information
- Patients management
- Practitioners management
- Role-based functionality (Admin, Receptionist, Practitioner, Patient)
- Telemedicine
- Treatment notes

# Benefits

-   Fully FHIR compatible:
    -   all app data are stored as FHIR resources
    -   any app data are available via FHIR API
-   Extremely flexible:
    -   use extensions and profiles to adjust FHIR data model
-   Fast to build forms and CRUD
    -   all forms in the app are just Questionnaire resources
-   Build the app with no-code
    -   app provides UI Questionnaire builder for creating Questionnaires
