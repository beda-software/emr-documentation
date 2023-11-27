---
sidebar_position: 14
---

# Form Builder
## General information
All forms in the system are [FHIR Questionnaires](http://hl7.org/fhir/R4B/questionnaire.html). You can adjust them by chnaging Questionnaire definition. You can do it on fly via calls to FHIR server or update seeds and upload them during startup.  
For technical users we provides SDC IDE, it is a comprehansive tool that helps you with creating or editin Questionnaires in yaml format. Less technical users can use chat driven interface to generate questionanires. It is a good approach to form prototyping.

### Chat based interface
To create a new questonnaire or edit an existing one, you need to open `Questionnaire` section on the left handside menu. Then click on `+ Add questionnaire` button. You will see a chat-driver questionanire builder user interface. You can use free text to describe the questionanire you would like to create. Here is an example of sutable text:  
```
A questionnaire to capture immunization details.
```
The ai driven backend will generate a questionanire based on this descriprion. You will see a form representation. You can click on any element to see its details and adjust them. You also can delete element or change an order via drag and drop. If you would like to add more fields or change their behaviour plese use chat interface.  
Here are two more examples on how you may want to adjust immunization questionnaire:  
```
The vaccine field shall be a select that uses the hl7 valueset.
```
```
Add yes/no question: Are there any side effects?
```
```
Add a question named 'Choose side effect'. It is a select that contains the top 10 common vaccination side effects SNOMED encoded as embedded options.
Show the question 'Choose side effect' only when the answer to the question 'Are there any side effects?'  is yes. 

```
By entering this command to chat interface you can adjust questionnaire behaviour and see how it affect the form. You can get back to the prious state of the form at any time. Once you satisfied eith the reqult you can save the form by clicking on `Save questionnaire` button on the rigth top corner. During the saveing process you will need to specify `Subject Type`. This field will add questionnaire to a list of `Patient` and/or `Encounter` documents. If you choose `Draft` status the questionnaire will be hidden from `Create doucment` list.


### Advance questionnaire tailoring with SDC IDE
Beda EMR leverage SDC IDE application that provides debuger experience for Questionnaire workflows. Any form could be launched in SCD IDE by clicking on `Edit in SDC IDE` button in the actions column of Questionanires list.
SDC IDE constist of 6 windows.
![SDC IDE interface](/img/sdc-ide.png)
**Launch Context**  
You need to choose initial launch parameters for the questionnaire. Questionnaire need some context like patient or encounter and author. Some questionanire may require more specific context like specific Condition or Appointemnt. You can set up a desired launch context in this window.   
**Questionnaire FHIR resource**  
You can edit the questionnaire in yaml format, all changes will be represented in real-time at the right top window that renders the form based on its definition. In some cases you have to manually save the questionnaire to see that changes in the form.  
**QuestionnaireResponse FHIR resource**  
This window represent the form state. You can chnage values of the form in the right top window and see how the resulted QuestionnaireResponse looks like.  
**Mapping**  
In this window you can define a mapper from QuestionnaireResponse to FHIR Bundle transaction that istantiate FHIR resources from form values. You can use two extraction engines:  
* [JUTE](https://github.com/healthSamurai/jute.clj)  
* [FHIRPath mapping language](https://github.com/beda-software/FHIRPathMappingLanguage)  
To select a specific type of mapping language please set type attribute to `JUTE` or `FHIRPathMappingLanguage`. `JUTE` is used by default.
The result of the mapper you will see in the right bottom corner. It updates on fly once you change form falues in the rigth top corner. You have to manually save mapper to see the changes from a an updated version of the mapper.  
#### FHIRPath debuggin
FHIR Questinnaires uses FHIR path expresions for many purposes. Also the whole extraction process is based on quering data with FHIRPath. So SDC IDE provides you with ability to test FHIRPath expressions. To test the expression you can use rigth mouse click on any FHIRPath exresion in `Questionnaire FHIR Resource` window. You will see a popup where you can play with expression on observe its result in the real time. You you clean the expression input you will see a list of all FHIRPath variables available. By clicking on `save` button, the system will replace previously selected fhirpath expresion with a version from the popup.
#### Mapper generator
For a brand new questionnaire the Mapper window will be empty. You can choose an existing mapper, use ai based mapper generator, or create an empty mapper and implement it manually `Add blank mapper`.
![Mapper](/img/mapping.png)
For ai-based mapper generator you can provide addition requrements to the mapper. Here is an example:  
```
Extract to immunization resource.
```
Once mapper is generated you can edit it in yaml format in mapper window.
