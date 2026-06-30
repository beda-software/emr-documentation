# Clinical context

Clinical context is the working environment of a practitioner in Beda EMR: the signed-in user, the patient whose chart is open, the active encounter, and other FHIR resources that questionnaires and mappers need at runtime.

- [`@beda.software/fhir-questionnaire`](https://github.com/beda-software/fhir-questionnaire) — React context provider, hooks, and merge helpers ([PR #16](https://github.com/beda-software/fhir-questionnaire/pull/16))
- [Beda EMR](https://github.com/beda-software/fhir-emr) — wiring context through routes, list pages, detail pages, and patient documents ([PR #854](https://github.com/beda-software/fhir-emr/pull/854), [issue #848](https://github.com/beda-software/fhir-emr/issues/848))

Requires `@beda.software/fhir-questionnaire` `^0.1.0-alpha.13` or newer.

## Data model

Clinical context is an array of FHIR `ParametersParameter` entries. Each entry has a `name` and a `resource` (or other value types supported by Parameters).

```typescript
import { ParametersParameter } from 'fhir/r4b';

const context: ParametersParameter[] = [
    { name: 'Patient', resource: patient },
    { name: 'Encounter', resource: encounter },
    { name: 'Author', resource: practitioner },
];
```

Parameter names are case-sensitive. Beda EMR commonly exposes both PascalCase and lowercase aliases for the same resource (for example `Patient` and `patient`) via the `getResourceClinicalContext` helper.

## fhir-questionnaire API

### `ClinicalContext` provider

Wrap part of the UI tree to share launch context with nested forms and components:

```tsx
import { ClinicalContext } from '@beda.software/fhir-questionnaire';

<ClinicalContext
    context={[
        { name: 'Patient', resource: patient },
        { name: 'Encounter', resource: encounter },
    ]}
>
    {children}
</ClinicalContext>
```

Nested providers compose. By default, a child parameter with the same `name` **replaces** the parent value for lookups. Pass `append` to keep all values and append instead:

```tsx
<ClinicalContext context={[patient('outer')]}>
    <ClinicalContext append context={[patient('inner')]}>
        {children}
    </ClinicalContext>
</ClinicalContext>
```

### Reading context

```tsx
import {
    getFirstParameter,
    getParameters,
    useClinicalContext,
} from '@beda.software/fhir-questionnaire';

function MyComponent() {
    const { parameters } = useClinicalContext();

    const patient = getFirstParameter(parameters, 'Patient')?.resource;
    const allPatients = getParameters(parameters, 'Patient');
}
```

| Function | Returns | Behavior |
| -------- | ------- | -------- |
| `getParameters(params, name)` | `ParametersParameter[]` | All matches, **child-first** (deepest / most recent first) |
| `getFirstParameter(params, name)` | `ParametersParameter \| undefined` | The deepest match for that name |

When no provider is mounted, `useClinicalContext()` returns `{ parameters: [] }`.

### Launch context for questionnaires

`QuestionnaireResponseForm` automatically merges clinical context with explicit `launchContextParameters`:

```
effective launch context = mergeLaunchContextParameters(clinicalContext, launchContextParameters)
```

Later parameters override earlier ones for the same name. Existing code that passes `launchContextParameters` or `sdcQrfProps.launchContextParameters` continues to work; those values are merged on top of the inherited context.

## Beda EMR helpers

### `getResourceClinicalContext`

Creates parameter entries under a logical name plus common aliases:

```typescript
import { getResourceClinicalContext } from 'src/utils';

getResourceClinicalContext('Patient', patient);
// → [{ name: 'Patient', resource }, { name: 'patient', resource }]

getResourceClinicalContext('Encounter', encounter, ['CurrentEncounter']);
// → Encounter, encounter, CurrentEncounter
```

For create flows that need a typed but empty resource (for example a Questionnaire used for both create and update), pass an empty resource object:

```typescript
getResourceClinicalContext('Patient', {} as FhirResource);
getResourceClinicalContext('MedicationKnowledge', {} as FhirResource, ['CurrentMedicationKnowledge']);
```

### `getRecordClinicalContextDefault`

Default mapper for list and detail pages — exposes the row's primary resource under its `resourceType` and lowercase name:

```typescript
import { getRecordClinicalContextDefault } from 'src/uberComponents/ResourceListPage/utils';

// For a Patient row → Patient + patient parameters
getRecordClinicalContextDefault(record);
```

## Where context is provided in Beda EMR

### Authenticated session (`EMR` container)

After login, the whole authenticated app is wrapped in `ClinicalContext` with the current user's role resource. See [EMR component](./emr-component.md) for how the container fits into a custom build.

- `User` — role resource (Practitioner, Patient, etc.)
- `{resourceType}` — same resource under its FHIR type name
- `Author` — same resource for questionnaire authorship

Override via the `getAuthenticatedClinicalContext` prop on `EMR`:

```tsx
<EMR
    getAuthenticatedClinicalContext={() => [
        ...getResourceClinicalContext('User', myUser),
        ...getResourceClinicalContext('Author', myUser),
    ]}
/>
```

Default implementation: `getAuthenticatedClinicalContextDefault()` in `src/containers/EMR/defaultAuthenticatedClinicalContext.ts`.

### Bundle-backed pages (`RenderBundleResourceContext`)

Pages that load a FHIR search bundle (patient chart, medication detail, document print, etc.) wrap their content in an additional `ClinicalContext` layer.

Props:

| Prop | Purpose |
| ---- | ------- |
| `resourceType` | Primary resource type to extract from the bundle |
| `getSearchParams` | Search params from route params |
| `extractPrimaryResource` | Optional custom extractor from bundle |
| `getClinicalContext` | Optional function `(record) => ParametersParameter[]`; defaults to `getRecordClinicalContextDefault` |

The render prop receives `{ resource, bundle, reload }`.

### Resource list pages (`ResourceListPage`)

`getClinicalContext` controls what is passed to questionnaire actions:

| Action scope | `record` argument | Default behavior |
| ------------ | ----------------- | ---------------- |
| Header (create) | `undefined` | Empty array (`getRecordClinicalContextDefault(undefined)`) |
| Row action | `{ resource, bundle }` | Primary resource under type name + lowercase |
| Batch (multi-select) | One call per selected row | Parameters from each row are concatenated; a `Bundle` parameter is also added |

Header and row launch contexts are merged with `defaultLaunchContext` and ancestor `ClinicalContext` via `mergeLaunchContextParameters`.

**Example — patient list built on `Consent`:**

The list row resource is `Consent`, but questionnaires expect `Patient`. Provide a custom mapper:

```tsx
<ResourceListPage<Consent>
    resourceType="Consent"
    getClinicalContext={(record) => {
        if (!record) {
            // Header "create" action — empty Patient for populate
            return getResourceClinicalContext('Patient', {} as FhirResource);
        }
        const patient = getPatientFromConsent(record.resource, record.bundle);
        return patient ? getResourceClinicalContext('Patient', patient) : [];
    }}
/>
```

**Example — standard patient list:**

```tsx
<ResourceListPage<Patient>
    getClinicalContext={(record) => getRecordClinicalContextDefault(record)}
/>
```

### Resource detail pages (`ResourceDetailPage`)

`ResourceDetailPage` is a layout wrapper around `RenderBundleResourceContext`. It loads the detail record from FHIR, wraps the page in `ClinicalContext`, and renders tab routes. All props shared with `RenderBundleResourceContext` — including `getClinicalContext` — are passed through via `{...props}`.

```tsx
// src/uberComponents/ResourceDetailPage/index.tsx (simplified)
export function ResourceDetailPage<R extends Resource>(props: DetailPageProps<R>) {
    return (
        <RenderBundleResourceContext<R> {...props}>
            {(context) => (
                <PageContainer title={getTitle(context)} /* ... */>
                    <Routes>
                        {tabs.map(({ path, component }) => (
                            <Route path={'/' + path} element={component(context)} />
                        ))}
                    </Routes>
                </PageContainer>
            )}
        </RenderBundleResourceContext>
    );
}
```

#### `getClinicalContext` on the detail page

Add `getClinicalContext` to `DetailPageProps`. The function receives `{ resource, bundle }` for the loaded detail record and returns parameters merged into the page-level `ClinicalContext`:

| Scope | `record` argument | Default behavior |
| ----- | ----------------- | ---------------- |
| Detail page | `{ resource, bundle }` from the loaded bundle | Primary resource under its `resourceType` and lowercase name (`getRecordClinicalContextDefault`) |

Tab components receive the same `context` object in their render function and automatically inherit this clinical context — including ancestor context from the authenticated session.

#### Nested lists inside tabs

Tabs often embed `ResourceListPageContent` with their own questionnaire actions. Those nested lists accept a separate `getClinicalContext` prop (same semantics as `ResourceListPage`) for header and row actions inside the tab.

The detail-page context and the nested list context compose: list actions merge their parameters with inherited ancestor context via `mergeLaunchContextParameters`.

**Example — medication knowledge detail** (`MedicationManagementDetail`):

The outer detail page exposes the loaded `MedicationKnowledge` record. A tab lists related `Medication` batches and needs both the batch row and the parent knowledge record in launch context:

```tsx
// Page level — context for the whole detail view
<ResourceDetailPage<MedicationKnowledge>
    resourceType="MedicationKnowledge"
    getSearchParams={({ id }) => ({ _id: id })}
    getTitle={({ resource, bundle }) => getMedicationName(resource, { bundle }) ?? ''}
    getClinicalContext={(record) => [
        ...getResourceClinicalContext('MedicationKnowledge', record.resource, [
            'CurrentMedicationKnowledge',
        ]),
    ]}
    tabs={tabs}
/>

// Tab content — nested list with its own context for create/row actions
function MedicationKnowledgeOverview({ resource }: { resource: MedicationKnowledge }) {
    return (
        <ResourceListPageContent<Medication>
            resourceType="Medication"
            searchParams={{ code, status: 'active' }}
            getHeaderActions={() => [questionnaireAction(t`Add batch`, 'medication-batch-create')]}
            getClinicalContext={(record) => [
                ...getResourceClinicalContext('Medication', record?.resource ?? ({} as FhirResource)),
                ...getResourceClinicalContext('MedicationKnowledge', resource, ['CurrentMedicationKnowledge']),
            ]}
        />
    );
}
```

For the header **create** action (`record` is `undefined`), the nested list passes an empty `Medication` resource so populate works for both create and update questionnaires. For **row** actions, `record.resource` is the batch's `Medication`.

This replaces the previous pattern of passing `defaultLaunchContext={[{ name: 'CurrentMedicationKnowledge', resource }]}` on nested lists — parent context is now derived from `getClinicalContext` instead.

## Using context in custom components

### Forms that require a patient

`PatientDocument` resolves the patient from clinical context:

```tsx
import { getFirstParameter, mergeLaunchContextParameters, useClinicalContext } from '@beda.software/fhir-questionnaire';

const { parameters: clinicalParams } = useClinicalContext();
const mergedParams = mergeLaunchContextParameters(clinicalParams, props.launchContextParameters ?? []);
const patient = getFirstParameter(mergedParams, 'Patient')?.resource;
```

Wrap the form in `ClinicalContext` in tests and standalone pages:

```tsx
<ClinicalContext
    context={[
        { name: 'Patient', resource: patient },
        { name: 'Author', resource: practitioner },
    ]}
>
    <PatientDocument questionnaireId="allergies" onSuccess={onSuccess} />
</ClinicalContext>
```

### Standalone questionnaire routes

`PatientQuestionnaire` wraps `PatientDocument` with patient and author context:

```tsx
<ClinicalContext
    context={[
        ...getResourceClinicalContext('Patient', patient, ['patient']),
        ...getResourceClinicalContext('Author', author, ['author']),
    ]}
>
    <PatientDocument questionnaireId={questionnaireId} />
</ClinicalContext>
```

## Migration notes

If you maintain a custom EMR build on top of Beda EMR:

1. **Remove direct `patient` / `author` props** from `PatientDocument` — provide them via `ClinicalContext` instead.
2. **Replace URL-based encounter wiring** — use nested `ClinicalContext` on encounter routes rather than passing `encounterId` into forms.
3. **List page create actions** — implement `getClinicalContext` when the listed resource type differs from what questionnaires expect, or when create flows need an empty typed resource.
4. **Keep `launchContextParameters`** for action-specific overrides; they merge with inherited context rather than replacing it entirely.

## Related documentation

- [EMR component](./emr-component.md) — application shell, routing, and menu customization
- [Form render engine](./form-engine.md) — integrating `@beda.software/fhir-questionnaire` in custom projects
- [Custom EMR build](./custom-emr-build.md) — project setup and customization entry points
- [fhir-questionnaire ClinicalContext README](https://github.com/beda-software/fhir-questionnaire#clinicalcontext) — package-level API reference
