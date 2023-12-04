---
sidebar_position: 14
---

# Prescription Management
## Definition
Prescription management is a crucial component of our system, involving several steps:
1. **Create New Medication Knowledge**: This provides foundational information about various medications.
2. **Batch Creation of New Medications**: This step involves adding information about medications available at the clinic, including quantity, batch number, and expiration date.
3. **Create a Prescription**: This functionality, available on the encounter page, allows practitioners to generate prescriptions.
4. **Confirm or Cancel Prescriptions**: This can be done on the prescriptions page.

## Attributes
This section of the documentation describes attributes from the user's perspective. For an understanding from a developer's perspective, please refer to the following links:
1. [MedicationRequest](https://hl7.org/fhir/R4/medicationrequest.html)

### MedicationRequest
| Attribute     | Description                               |
|---------------|-------------------------------------------|
| Medications   | Medication to be added to the prescription|

## Actions
### Create
Practitioners can create a prescription by filling out a new document titled "Medication Prescription." In this document, the practitioner selects medication from the clinic's warehouse. Once a medication prescription is created and saved, the selected medication becomes unavailable in the medication list.

![Medication prescription create](/img/medication-prescription-create.gif)

### Confirm
Receptionists can confirm prescriptions and dispense the medication to the patient. The prescribed medication remains unavailable for other uses in the system.

![Medication prescription confirm](/img/medication-prescription-confirm.gif)

### Cancel
Receptionists have the ability to cancel medication prescriptions. In such cases, the medication becomes available again in the medication list.

![Medication prescription cancel](/img/medication-prescription-cancel.gif)

### Filter
Receptionists can filter the prescription list using the following criteria:
1. Practitioner
2. Patient
3. Status

![Medication prescription filter](/img/prescriptions-filters.gif)
