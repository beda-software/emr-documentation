---
sidebar_position: 13
---

# Medications management
## Manage knowledge base
### Definition
Users are empowered to manage the medication knowledge base by adding new MedicationKnowledge data. This resource is crucial for subsequent steps involving the management of existing medications and medication requests.
### Attributes
This documentation presents attributes from the user's perspective. For an understanding from a developer's perspective, please refer to the following links:
1. [MedicationKnowledge](https://hl7.org/fhir/R4/medicationknowledge.html)
#### MedicationKnowledge
|Attribute|Description|
|-|-|
|Name|Name of the medication|
|Code|Unique code identifying this medication|
|Packaging Type|Details about the medication's packaging|
|Dose Form|Form in which the medication dose is administered|
|Amount Unit|Unit of measure for the drug amount in the package|
|Amount Value|Numeric value representing the drug amount in the package|
|Ingredient|Specifies whether it's an active or inactive ingredient|
|Numerator Unit|Unit of measure for the ingredient's quantity|
|Numerator Value|Numeric value of the ingredient's quantity|
|Denominator Unit|Unit of measure for another ingredient's quantity (if applicable)|
|Denominator Value|Numeric value of another ingredient's quantity (if applicable)|
|Price|Pricing information for the medication|
### Actions
#### Create
![New medication knowledge](/img/new-medication-knowledge.gif)
## Manage warehouse
### Definition
Users can manage existing medications, including the creation of multiple medications from a single batch through a unified form.
### Attributes
This documentation presents attributes from the user's perspective. For a developer's perspective, refer to:
1. [Medication](https://hl7.org/fhir/R4/medication.html)
#### Medication
|Attribute|Description|
|-|-|
|Medication Name|Name of the medication, prefilled from the knowledge base|
|Items Number|Total number of medication items|
|Batch Number|Identification number for the batch|
|Expiration Date|Expiry date of the medication|
### Actions
#### Create
![New medication](/img/medication-batch-create.gif)
