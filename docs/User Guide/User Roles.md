# User roles
## Short overview
|Role|Description|
|----|-----------|
|Admin|Kind of superuser in the system. Can manage system, resources, practitioners, etc.|
|Receptionist|Can schedule/cancel appointments for the patient and manage their invoices.|
|Practitioner|Can manage their own appointments, questionnaires, and patient care.|
|Patient|Can see their own invoices and medical information related to them.|
## Modules by role
| Module         | Admin | Receptionist | Practitioner | Patient |URL|
|---------------------|-|-|-------|--------------|--------------|---------|
| [Invoices](Invoices)            |   ✅   |       ✅      |       ❌      |    ✅    |https://emr.beda.software/invoices|
| [Services](HealthcareServiceManagement)            |   ✅   |       ❌      |       ❌      |    ❌    |https://emr.beda.software/healthcare-services|
| Encounters          |   ✅   |       ❌      |       ✅      |    ❌    |https://emr.beda.software/encounters|
| Patients            |   ✅   |       ❌      |       ✅      |    ❌    |https://emr.beda.software/patients|
| Practitioners       |   ✅   |       ❌      |       ❌      |    ❌    |https://emr.beda.software/practitioners|
| Questionnaires      |  ✅   |       ❌      |       ✅      |    ❌    |https://emr.beda.software/questionnaires| 
| [Scheduling](Scheduling)          |   ❌   |       ✅      |       ❌      |    ❌    |https://emr.beda.software/scheduling|
| Patient's dashboard |   ❌   |       ❌      |       ✅      |    ✅    |https://emr.beda.software/ for the Patient|
## Demo Credentials
|Role|Login|Password|
|----|-----|--------|
|Admin|admin|password|
|Receptionist|receptionist|password|
|Practitioner|practitioner1/practitioner2|password|
|Patient|patient1/patient2|password|
