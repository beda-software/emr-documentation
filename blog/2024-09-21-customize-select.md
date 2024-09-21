---
slug: 2024-09-21-customize-select
title: Customize data select
authors: ilya
tags: [release-notes, updates]
---

## Select customization for date

Some times you need to apply detailed styles to a form component.  
For example you would like to get a form for selecting a booking slot like this:
<img src="http://localhost:3000/img/blog/2024-09-21-customize-select/form.png"/>
As you can see slots are represented as radio buttons with a specific formating.

Let's see how we can implement such interface with FHIR Questionnaire:
```yaml
- text: Select slot
  type: reference
  linkId: slot
  required: true
  extension:
    - url: >-
        http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-choiceColumn
      extension:
        - url: path
          valueString: Slot.start
        - url: forDisplay
          valueBoolean: true
    - url: >-
        http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-answerExpression
      valueExpression:
        language: application/x-fhir-query
        expression: Slot?schedule={{%Schedule.id}}&status=free&_assoc=slot
    - url: >-
        http://hl7.org/fhir/StructureDefinition/questionnaire-referenceResource
      valueCode: Slot
```
This samle uses three extensions fro [FHIR SDC IG](https://build.fhir.org/ig/HL7/sdc/)
First of all we are using [answerExpression](https://build.fhir.org/ig/HL7/sdc/StructureDefinition-sdc-questionnaire-answerExpression.html) 
to load a list of avalable slots. We are using embeded FHIRPath to filter Slots by specific schedule id.
The `Schedule` resource is defined as [launch context](https://build.fhir.org/ig/HL7/sdc/StructureDefinition-sdc-questionnaire-launchContext.html) for the questionanire.
[referenceResource](https://build.fhir.org/ig/HL7/fhir-extensions/StructureDefinition-questionnaire-referenceResource-definitions.html) applies additonal constraon for the resource type we can use in a reference.
And finally, [choiceColumn](https://build.fhir.org/ig/HL7/sdc/rendering.html#choiceColumn) is used to extract a specific field from FHIR resource and format it.
In our case we are selcting `start` field of `Slot` resource.

Once we define this three extesions the from will look like this:
<img src="http://localhost:3000/img/blog/2024-09-21-customize-select/default-form.png"/>
This form is fully finctional, but it uses select widget for slot reference field.
Select is a default control for this kinf of field in Beda EMR.

Let's add (itemControl)[https://build.fhir.org/ig/HL7/sdc/rendering.html#itemControl] that will change the apperacne of the reference widget.
We will use reference-radio-button widget. The list of all other avaialbe widgets you can find in (Beda EMR storybook)[https://64b7c5c51809d460dc448e6b-jipisdnzdn.chromatic.com/?path=/story/questionnaire-questions-choice--default].
```yaml
- url: http://hl7.org/fhir/StructureDefinition/questionnaire-itemControl
  valueCodeableConcept:
    coding:
      - code: reference-radio-button
```
It will look close to the design:
<img src="http://localhost:3000/img/blog/2024-09-21-customize-select/radio-select-form.png"/>
but it missess the formating option `Wednesday • 18 Sep • 4:00 PM`.

Unfortunatly FHIRPath doesn't support formation for dates.
However fhirpath.js allow you to extend it with user invocation nodes: https://github.com/HL7/fhirpath.js?tab=readme-ov-file#user-defined-functions

So we can define a custom function for date formating like this:
```typescript
import { parseFHIRDateTime } from '@beda.software/fhir-react';

const formatDateUserInvocationTable: UserInvocationTable = {
    formatDate: {
        fn: (inputs: string[], format: string) => {
            return inputs.map((i) => parseFHIRDateTime(i).format(format));
        },
        arity: { 0: [], 1: ['String'] },
    },
};
```

After that we should modify choice FHIRPath in choiceColumn extension to leverage this new custom function
```yaml
- url: >-
    http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-choiceColumn
  extension:
    - url: path
      valueString: Slot.start.formatDate('dddd • D MMM • h:mm A')
    - url: forDisplay
    valueBoolean: true
```
After this final update we get a form that fully match the desired design:
<img src="http://localhost:3000/img/blog/2024-09-21-customize-select/form.png"/>

It may be challenging to achive a pixel perfect match of design and implemntation.
In Beda EMR is our goal to provide a conviniet way of acheeving it and don't limit developers and designers.
With this approcah we can build any kind of interfaces for modern healthcare applications.
