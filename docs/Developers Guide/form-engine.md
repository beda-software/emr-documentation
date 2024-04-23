# Form render engine
This article describes how to use SDC form engine https://github.com/beda-software/fhir-questionnaire outside of Beda EMR.

1. Install fhir-questionnaire as submodule to your project and specify it as a workspace in your package.json https://github.com/beda-software/fhir-questionnaire?tab=readme-ov-file#installation-instructions

2. Implement widgets for each question type. Here are examples of widgets: https://github.com/beda-software/fhir-emr/tree/master/src/components/BaseQuestionnaireResponseForm/widgets We encourage you to use https://storybook.js.org/ to test each widget independently.

3. Create your own `BaseQuestionnaireResponseForm` component. fhir-questionnaire provides generic https://github.com/beda-software/fhir-questionnaire/blob/a4a06b6bf948c5c82ef2b54e5fef37c8aad0ce49/components/QuestionnaireResponseForm/index.tsx#L109 implementation that contains multiple parameters, including maps to link widgets with `QuestionnaireItem.type`.  The best practice is to create a wrapper over `QuestionnaireResponseForm` that proivdes a set of defaults sutable for all your usecases.
Here is an example of such configuration:

```typescript
import { QuestionnaireResponseFormSaveResponse } from '@beda.software/fhir-questionnaire/components/QuestionnaireResponseForm/questionnaire-response-form-data';
import { serviceProvider } from 'services/auth';

export function BaseQuestionnaireResponseForm(props: {
    questionnaireId: string;
    launchContextParameters?: ParametersParameter[];
    initialQuestionnaireResponse?: Partial<FHIRQuestionnaireResponse>;
    onSuccess?: (response: QuestionnaireResponseFormSaveResponse) => void;
    onFailure?: (response: Error) => void;
}) {
    return (
        <QuestionnaireResponseForm
            questionnaireLoader={questionnaireIdWOAssembleLoader(props.questionnaireId)}
            launchContextParameters={props.launchContextParameters}
            initialQuestionnaireResponse={props.initialQuestionnaireResponse}
            readOnly={false}
            onSuccess={props.onSuccess}
            onFailure={props.onFailure}
            serviceProvider={serviceProvider}
            widgetsByQuestionType={{
                dateTime: QuestionDateTime,
                integer: QuestionInteger,
                choice: InlineChoice,
                string: QuestionString,
            }}
            widgetsByQuestionItemControl={{
                'phone': QuestionPhone,
                'slider': QuestionSlider,
            }}
            FormWrapper={({ items, handleSubmit }) => {
                return (
                    <form onSubmit={handleSubmit}>{items}</form>
                );
            }}
        />
    );
}

```
