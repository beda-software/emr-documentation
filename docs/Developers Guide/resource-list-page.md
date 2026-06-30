---
sidebar_position: 5
---

# Resource list page

`ResourceListPage` and `ResourceListPageContent` are the standard building blocks for FHIR resource list views in a custom Beda EMR build. They load paginated search results, render a filterable table, and wire up header, row, and batch actions — including questionnaire modals and navigation links.

Sources:

- [`ResourceListPage`](https://github.com/beda-software/fhir-emr/blob/main/src/uberComponents/ResourceListPage/index.tsx#L60)
- [`ResourceListPageContent`](https://github.com/beda-software/fhir-emr/blob/main/src/uberComponents/ResourceListPageContent/index.tsx#L36)

Import:

```tsx
import {
    ResourceListPage,
    navigationAction,
    questionnaireAction,
    customAction,
} from '@beda.software/emr/dist/uberComponents/ResourceListPage/index';

import { ResourceListPageContent } from '@beda.software/emr/dist/uberComponents/ResourceListPageContent/index';
```

When working inside the Beda EMR submodule, use the `src/` paths instead:

```tsx
import { ResourceListPage, navigationAction, questionnaireAction } from 'src/uberComponents/ResourceListPage';
import { ResourceListPageContent } from 'src/uberComponents/ResourceListPageContent';
```

## `ResourceListPage` vs `ResourceListPageContent`

Both components share the same data-loading, filtering, sorting, pagination, and action logic via [`useResourceListPage`](https://github.com/beda-software/fhir-emr/blob/main/src/uberComponents/ResourceListPage/hooks.ts). The difference is layout and where you use them.

| | `ResourceListPage` | `ResourceListPageContent` |
| - | ------------------ | ------------------------- |
| Shell | Full page with [`PageContainer`](https://github.com/beda-software/fhir-emr/blob/main/src/components/BaseLayout/PageContainer/index.tsx) — title, optional back button, header actions | Content block inside [`PageContainerContent`](https://github.com/beda-software/fhir-emr/blob/main/src/components/BaseLayout/PageContainer/PageContainerContent/index.tsx) (level 2) |
| Use case | Top-level routes registered in `EMR` (`/patients`, `/encounters`, …) | Nested inside a detail page tab, patient chart section, or other existing layout |
| Extra props | `headerTitle`, `backButtonVisible` | None — inherits surrounding page chrome |
| Header questionnaire context | Merges `defaultLaunchContext` with `getClinicalContext(undefined)` | Uses `defaultLaunchContext` only |

Use `ResourceListPage` for standalone list routes. Use `ResourceListPageContent` when the list is part of a larger page — typically a tab on [`ResourceDetailPage`](./resource-detail-page.md).

## What it provides

Both components compose:

1. **[`useResourceListPage`](https://github.com/beda-software/fhir-emr/blob/main/src/uberComponents/ResourceListPage/hooks.ts)** — FHIR search with pagination (`usePager`), debounced filter params, row selection for batch actions, and a `reload()` callback.
2. **[`SearchBar`](https://github.com/beda-software/fhir-emr/blob/main/src/components/SearchBar/index.tsx)** — filters from `getFilters()`, synced with table column filters.
3. **[`Table`](https://github.com/beda-software/fhir-emr/blob/main/src/components/Table/index.tsx)** — paginated data grid with optional row selection, sorting, and an auto-generated Actions column.
4. **Action helpers** — `questionnaireAction`, `navigationAction`, and `customAction` for header, row, and batch buttons.

`ResourceListPage` additionally wraps everything in `PageContainer` with a page title and optional back navigation.

## Route setup

Register a list page on a flat route in `EMR`:

```tsx
<Route path="/encounters" element={<EncounterList />} />
<Route path="/patients/:id/*" element={<PatientDetails />} />
```

Pair list and detail routes so row `navigationAction` links resolve:

```tsx
<Route path="/medications" element={<MedicationManagementList />} />
<Route path="/medications/:id/*" element={<MedicationManagementDetail />} />
```

See [EMR component](./emr-component.md) for route registration and [Resource detail page](./resource-detail-page.md) for the detail side.

## Props

Shared props come from `ResourceListProps`. Each row in the table is a `RecordType<R>`:

```tsx
type RecordType<R extends Resource> = { resource: R; bundle: Bundle; children?: RecordType<R>[] };
```

`bundle` is the full FHIR search response for the current page. Use it to resolve `_include`d resources (for example, `Patient` on an `Encounter` row).

### `ResourceListPage`-only props

```tsx
type ResourceListPageProps<R extends Resource> = ResourceListProps<R> & {
    headerTitle: string;
    backButtonVisible?: boolean;
    maxWidth?: number | string;
    getTableColumns: (manager: TableManager) => ColumnsType<RecordType<R>>;
};
```

| Prop | Required | Description |
| ---- | -------- | ----------- |
| `headerTitle` | Yes | Page title shown in `PageContainer`. |
| `backButtonVisible` | No | When `true`, shows a back button that calls `navigate(-1)`. |
| `getTableColumns` | Yes | Column definitions (without the Actions column). Receives `{ reload }` via `TableManager`. |


### Shared props (`ResourceListProps`)

| Prop | Required | Description |
| ---- | -------- | ----------- |
| `resourceType` | Yes | Primary FHIR resource type for rows (for example, `"Encounter"`). |
| `searchParams` | No | Default FHIR search parameters (`_sort`, `_count`, `_include`, …). Filter and sort values from the UI are merged on top. |
| `extractPrimaryResources` | No | Custom function to pick primary rows from the search bundle. Default: all entries matching `resourceType`. |
| `extractChildrenResources` | No | Returns child resources for tree rows (for example, organizations linked via `partOf`). |
| `uniqueOrderSortSearchParam` | No | Tie-breaker sort param appended to `_sort` for stable pagination. Default: `-_lastUpdated`. Set to `null` to disable. |
| `getFilters` | No | Returns [`SearchBarColumn`](https://github.com/beda-software/fhir-emr/blob/main/src/components/SearchBar/types.ts) definitions for the search bar and table filters. |
| `getSorters` | No | Returns sorter definitions mapping column ids to FHIR `_sort` params. |
| `getRecordActions` | No | Per-row actions. When provided, an Actions column is appended automatically. |
| `getHeaderActions` | No | Buttons in the page header (create, import, …). |
| `getBatchActions` | No | Actions shown when rows are selected. Enables row checkboxes. Receives a `Bundle` of selected resources. |
| `defaultLaunchContext` | No | `ParametersParameter[]` merged into all questionnaire launch contexts. |
| `getClinicalContext` | No | Per-scope clinical context for questionnaire actions. See [Clinical context](./clinical-context.md#resource-list-pages-resourcelistpage). |
| `getReportColumns` | No | **Experimental.** Summary report above the table from the current search bundle. |
| `maxWidth` | No | Max content width. |
| `tableProps` | No | Extra props passed to the underlying Ant Design `Table` (excluding pagination, columns, data, etc.). |

`TableManager` passed to `getTableColumns` and `getRecordActions`:

```tsx
interface TableManager {
    reload: () => void;
}
```

Call `reload()` after a mutation to refresh the current page.

## Table columns

### `ResourceListPageContent` — Ant Design columns

`ResourceListPageContent` expects standard Ant Design `ColumnsType<RecordType<R>>`. Define cell content in `render`:

```tsx
getTableColumns={() => [
    {
        title: t`Lot number`,
        key: 'lotNumber',
        render: (_: unknown, record) => getMedicationLotNumber(record.resource) ?? '',
    },
    {
        title: t`Expiration date`,
        key: 'expirationDate',
        render: (_: unknown, record) => getMedicationExpirationDate(record.resource) ?? '',
    },
]}
```

The Actions column is still generated automatically when `getRecordActions` is provided.

## Actions

Import action builders from `ResourceListPage`:

```tsx
import { navigationAction, questionnaireAction, customAction } from 'src/uberComponents/ResourceListPage';
```

### Questionnaire actions

Opens a modal with [`QuestionnaireResponseForm`](https://github.com/beda-software/fhir-emr/blob/main/src/components/QuestionnaireResponseForm/index.tsx). On success, shows a notification and calls `reload()`.

Define the form as a `Questionnaire` + `Mapping` pair on the FHIR server, then reference the questionnaire id here. See [Questionnaire actions](./questionnaire-actions.md) for the full authoring workflow.

```tsx
questionnaireAction(<Trans>Edit</Trans>, 'healthcare-service-edit')

questionnaireAction(<Trans>Add patient</Trans>, 'patient-create', {
    icon: <PlusOutlined />,
    extra: {
        modalProps: { width: 800 },
    },
})
```

`extra.qrfProps` and `extra.modalProps` are web-specific options (`WebExtra`).

### Navigation actions

Navigates with React Router. The row `resource` is passed in `location.state`:

```tsx
navigationAction(<Trans>Chart</Trans>, `/patients/${record.resource.id}`)
```

Use this to open a [`ResourceDetailPage`](./resource-detail-page.md) route.

### Custom actions

Render any React node in the Actions column or batch bar:

```tsx
customAction(<MyCustomButton onDone={manager.reload} record={record} />)
```

### Action scopes

| Scope | Callback | Typical use |
| ----- | -------- | ----------- |
| Header | `getHeaderActions` | Create, import |
| Row | `getRecordActions(record, { reload })` | Edit, open detail, video call |
| Batch | `getBatchActions(selectedBundle)` | Bulk update, bulk delete |

Batch actions appear above the table when `getBatchActions` returns at least one action. Row checkboxes are enabled automatically.

## Filters and sorting

**Filters** — return `SearchBarColumn[]` from `getFilters()`. Values are debounced (300 ms) and mapped to FHIR search params via each column's `searchParam` or `id`.

**Sorters** — return `SorterColumn[]` from `getSorters()`:

```tsx
const getSorters = () => [
    { id: 'date', searchParam: 'date', label: 'Date' },
];
```

Column header clicks update `_sort`. The component appends `uniqueOrderSortSearchParam` (default `-_lastUpdated`) as a tie-breaker so pagination stays stable.

Set initial sort in `searchParams`:

```tsx
searchParams={{ _sort: '-date,_id', _count: 10 }}
```

## Data loading

`useResourceListPage` calls `usePager` with `resourceType` and merged search params. Each page returns a bundle; primary resources are extracted and wrapped as `{ resource, bundle }`.

Use `_include` / `_include:iterate` in `searchParams` to load related resources for column rendering:

```tsx
searchParams={{
    '_include:iterate': [
        'Encounter:subject',
        'Encounter:participant:PractitionerRole',
        'Encounter:participant:Practitioner',
        'PractitionerRole:practitioner:Practitioner',
    ],
    _sort: '-date,_id',
    _count: 10,
}}
```

Role-based scoping is a common pattern — filter `searchParams` by the current user before passing them to the list:

```tsx
const roleSearchParams = matchCurrentUserRole({
    [Role.Practitioner]: (practitioner) => ({ participant: practitioner.id }),
    [Role.Admin]: () => ({}),
    // ...
});

<ResourceListPage
    searchParams={{ ...roleSearchParams, _sort: '-date,_id', _count: 10 }}
    // ...
/>
```

## Pairing with detail pages

List pages typically link to detail pages via `navigationAction`:

```tsx
const getRecordActions = (record: RecordType<Encounter>) => [
    navigationAction(<Trans>Open</Trans>, `/patients/${patientId}/encounters/${record.resource.id}`),
];
```

Detail pages can embed `ResourceListPageContent` inside a tab for related resources. See [Resource detail page — Nested lists](./resource-detail-page.md#nested-lists-inside-tabs).

## Examples

### Encounter list — includes, custom format, navigation

[`EncounterList`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/EncounterList/index.tsx) — standalone page with `_include:iterate`, custom column `format` functions, filters, sorters, and row navigation:

```tsx
<ResourceListPage
    headerTitle={t`Encounters`}
    resourceType="Encounter"
    searchParams={{
        '_include:iterate': [
            'Encounter:subject',
            'Encounter:participant:PractitionerRole',
            'Encounter:participant:Practitioner',
            'PractitionerRole:practitioner:Practitioner',
        ],
        _sort: '-date,_id',
        _count: 10,
    }}
    getTableColumns={getTableColumns}
    getRecordActions={(record) => [
        navigationAction(<Trans>Open</Trans>, `/patients/${patientId}/encounters/${record.resource.id}`),
        navigationAction(<Trans>Video call</Trans>, `/encounters/${record.resource.id}/video`),
    ]}
    getFilters={getFilters}
    getSorters={getSorters}
/>
```

### Healthcare service list — FHIRPath columns and questionnaire actions

[`HealthcareServiceList`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/HealthcareServiceList/index.tsx) — declarative FHIRPath getters with header and row questionnaire actions:

```tsx
<ResourceListPage
    headerTitle={t`Healthcare Services`}
    resourceType="HealthcareService"
    searchParams={{ _sort: '-_lastUpdated,_id', _count: 10 }}
    getFilters={getFilters}
    getSorters={getSorters}
    getTableColumns={getTableColumns}
    getRecordActions={getRecordActions}
    getHeaderActions={getHeaderActions}
/>
```

### Patient list — Consent-based search with custom clinical context

[`PatientList`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/PatientList/index.tsx) — when the list searches `Consent` but questionnaires expect `Patient`, override `getClinicalContext`:

```tsx
<ResourceListPage<Consent>
    headerTitle={t`Patients`}
    resourceType="Consent"
    getClinicalContext={(record) => {
        if (!record) {
            return getResourceClinicalContext('Patient', {} as FhirResource);
        }
        const patient = getPatientFromConsent(record.resource, record.bundle);
        return patient ? getResourceClinicalContext('Patient', patient) : [];
    }}
    // ...
/>
```

### Nested list inside a detail tab

[`MedicationManagementDetail`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/MedicationManagementDetail/index.tsx) — `ResourceListPageContent` on the overview tab lists related `Medication` batches:

```tsx
<ResourceListPageContent<Medication>
    resourceType="Medication"
    searchParams={{ code, status: 'active' }}
    getHeaderActions={() => [questionnaireAction(t`Add batch`, 'medication-batch-create')]}
    getTableColumns={() => [
        {
            title: t`Lot number`,
            key: 'lotNumber',
            render: (_: unknown, record) => getMedicationLotNumber(record.resource) ?? '',
        },
    ]}
    getClinicalContext={(record) => [
        ...getResourceClinicalContext('Medication', record?.resource ?? ({} as FhirResource)),
        ...getResourceClinicalContext('MedicationKnowledge', resource, ['CurrentMedicationKnowledge']),
    ]}
/>
```

[`PatientEncounter`](https://github.com/beda-software/fhir-emr/blob/main/src/components/PatientEncounter/index.tsx) — encounters list embedded in a patient chart with `defaultLaunchContext` for the current patient:

```tsx
<ResourceListPageContent<Encounter>
    resourceType="Encounter"
    searchParams={{ subject: patient.id, _sort: '-date,_id', _count: 10 }}
    getTableColumns={getTableColumns}
    getRecordActions={getRecordActions}
    getHeaderActions={getHeaderActions}
    defaultLaunchContext={[{ name: 'Patient', resource: patient }]}
/>
```

## Clinical context

Questionnaire actions merge launch context from three sources: ancestor `ClinicalContext`, `defaultLaunchContext`, and `getClinicalContext(record)`.

| Action scope | `getClinicalContext` argument | Default |
| ------------ | ----------------------------- | ------- |
| Header (create) | `undefined` | Empty array |
| Row action | `{ resource, bundle }` | Primary resource (PascalCase + lowercase names) |
| Batch (multi-select) | One call per selected row | Parameters concatenated per row |

On `ResourceListPage`, header questionnaire actions automatically merge `getClinicalContext(undefined)`. On `ResourceListPageContent`, pass patient or parent resource context via `defaultLaunchContext` or `getClinicalContext`.

Full details and examples: [Clinical context — Resource list pages](./clinical-context.md#resource-list-pages-resourcelistpage).

## Related documentation

- [Questionnaire actions](./questionnaire-actions.md) — define Questionnaire + Mapping pairs for header, row, and batch buttons
- [Resource detail page](./resource-detail-page.md) — tabbed FHIR resource detail layout; embed `ResourceListPageContent` in tabs
- [Clinical context](./clinical-context.md) — `getClinicalContext` for questionnaire launch parameters
- [EMR component](./emr-component.md) — register list routes in `authenticatedRoutes`
- [Custom EMR build](./custom-emr-build.md) — project setup and customization entry points
