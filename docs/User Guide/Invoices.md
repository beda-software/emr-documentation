# Invoices
## General information
### Definition
The demonstration of the simple billing system in the EMR. Admin users can set up service, service base price, and tax. Receptionist users can create an encounter with this service. When the encounter is finished, the invoice will be created in an automated manner. Right after that receptionist can check the details of the invoice and change the status of the invoice. Invoice lists and details are also available for the patient user.
### Attributes
In this documentation, attributes are described from the user's perspective. If you want to understand how these models are implemented from developers' perspective, please visit the next links:

1. [ChargeItem](https://hl7.org/fhir/R4/chargeitem.html)
2. [ChargeItemDefinition](https://hl7.org/fhir/R4/chargeitemdefinition.html)
3. [Invoice](https://hl7.org/fhir/R4/invoice.html)
#### Invoice
|Attribute|Description|
|-|-|
|Patient|The subject of the invoice|
|Practitioner|Participant of the invoice|
|Date|Date and time when the invoice was issued|
|Status|Status of the invoice. Can be issued, balanced, canceled|
#### Line item
|Attribute|Description|
|-|-|
|Item|Item name|
Quantity|Quantity||
|Rate|The base price of the item|
|Tax|Tax of the item|
|Amount|Amount (rate + tax)|
## Actions
### Details
Every invoice has a details page where the user can check additional information about the invoice, such as patient and practitioner names, issued dates, status, and amount. Also, details include information about every line item in the invoice: name of the service, quantity, rate, tax, amount
![Invoice details](/img/invoice_open.gif)
### Pay
The receptionist can change the invoice status from issued to balanced (paid). To do this user should click on the "payment" button and confirm his action. Right after this, the list of invoices will be refreshed and the user can see the result of the action.
![Invoice details](/img/invoice_pay.gif)
### Cancel
The receptionist can change the invoice status from issued to cancelled. To do this user should click on the "cancel" button and confirm his action. Right after this, the list of invoices will be refreshed and the user can see the result of the action.
![Invoice details](/img/invoice_cancel.gif)
### Search
Users can use filters to find the correct invoice. Available filters are practitioner, patient, and status. For the patient available practitioner and status filter.
![Invoice details](/img/invoice_search.gif)
### Create new
Currently, invoices can be created only after these steps:

1. Encounter was created;
2. Encounter was completed.

If these steps are reproduced, you will see a new encounter in issued status in the invoices list.

To control rate and tax values users can use the [Services](HealthcareServiceManagement) page (for admin only)
![Invoice details](/img/invoice_create.gif)