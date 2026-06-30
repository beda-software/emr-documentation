# Resource detail page

`ResourceDetailPage` is the standard layout for FHIR resource detail views in a custom Beda EMR build. It loads a record from the server, wraps the page in clinical context, renders a tabbed header, and mounts tab content as nested routes.

Source: [`ResourceDetailPage`](https://github.com/beda-software/fhir-emr/blob/main/src/uberComponents/ResourceDetailPage/index.tsx#L54) in Beda EMR.

Import:

```tsx
import { ResourceDetailPage, Tab } from '@beda.software/emr/dist/uberComponents/ResourceDetailPage/index';
```

When working inside the Beda EMR submodule, use the `src/` path instead:

```tsx
import { ResourceDetailPage, Tab } from 'src/uberComponents/ResourceDetailPage';
```

## What it provides

`ResourceDetailPage` composes three building blocks:

1. **[`RenderBundleResourceContext`](https://github.com/beda-software/fhir-emr/blob/main/src/components/RenderBundleResourceContext/index.tsx)** — reads route params, runs a FHIR search, extracts the primary resource from the bundle, and wraps children in `ClinicalContext`.
2. **[`PageContainer`](https://github.com/beda-software/fhir-emr/blob/main/src/components/BaseLayout/PageContainer/index.tsx)** — page shell with title, optional header slots, and `layoutVariant="with-tabs"`.
3. **[`PageTabs`](https://github.com/beda-software/fhir-emr/blob/main/src/uberComponents/ResourceDetailPage/index.tsx#L13)** — tab navigation synced with the current URL.

```tsx
export function ResourceDetailPage<R extends Resource>(props: DetailPageProps<R>) {
    const { getTitle, getTitleLeftElement, getTitleRightElement, tabs, maxWidth } = props;

    return (
        <RenderBundleResourceContext<R> {...props}>
            {(context) => (
                <PageContainer
                    title={getTitle(context)}
                    titleLeftElement={getTitleLeftElement ? getTitleLeftElement(context) : undefined}
                    titleRightElement={getTitleRightElement ? getTitleRightElement(context) : undefined}
                    layoutVariant="with-tabs"
                    headerContent={<PageTabs tabs={tabs} />}
                    maxWidth={maxWidth}
                >
                    <Routes>
                        {tabs.map(({ path, component }) => (
                            <React.Fragment key={path}>
                                <Route path={'/' + path} element={component(context)} />
                                <Route path={'/' + path + '/*'} element={component(context)} />
                            </React.Fragment>
                        ))}
                    </Routes>
                </PageContainer>
            )}
        </RenderBundleResourceContext>
    );
}
```

Each tab's `component` render function receives `{ resource, bundle, reload }`. Use `reload()` after mutations to refresh the loaded bundle.

## Route setup

Register the detail page on a wildcard route so tab paths resolve under the record id:

```tsx
<Route path="/medications/:id/*" element={<MedicationManagementDetail />} />
```

`getSearchParams` maps route params to FHIR search parameters. The usual pattern for a record loaded by id:

```tsx
getSearchParams={({ id }) => ({ _id: id })}
```

`RenderBundleResourceContext` calls `useParams()` internally, so the parent route must expose `:id`.

## Props

```tsx
interface DetailPageProps<R extends Resource> {
    resourceType: R['resourceType'];
    getSearchParams: (params: Readonly<Record<string, string | string[] | undefined>>) => SearchParams;
    getTitle: (context: RecordType<WithId<R>>) => string | ReactElement;
    getTitleLeftElement?: (context: RecordType<WithId<R>>) => string | ReactElement;
    getTitleRightElement?: (context: RecordType<WithId<R>>) => string | ReactElement;
    tabs: Array<Tab<WithId<R>>>;
    extractPrimaryResource?: (bundle: Bundle) => WithId<R>;
    getClinicalContext?: (context: RecordType) => ParametersParameter[];
    maxWidth?: number | string;
}
```

| Prop | Required | Description |
| ---- | -------- | ----------- |
| `resourceType` | Yes | FHIR resource type to load and extract from the search bundle. |
| `getSearchParams` | Yes | Maps React Router params (from `useParams()`) to FHIR search params. |
| `getTitle` | Yes | Page title from the loaded `{ resource, bundle }`. |
| `getTitleLeftElement` | No | Optional element rendered to the left of the title (for example, a status badge). |
| `getTitleRightElement` | No | Optional element rendered to the right of the title (for example, action buttons). |
| `tabs` | Yes | Tab definitions — label, path segment, and content component. |
| `extractPrimaryResource` | No | Custom bundle extractor. Defaults to the first resource matching `resourceType`. |
| `getClinicalContext` | No | Parameters merged into page-level `ClinicalContext`. See [Clinical context](./clinical-context.md#resource-detail-pages-resourcedetailpage). |
| `maxWidth` | No | Max content width passed to `PageContainer` (for example, `"100%"`). |

## Tabs

```tsx
type Tab<R extends Resource, Extra = unknown> = {
    label: string;
    path?: string;
    component: (context: RecordType<R>) => JSX.Element;
} & Extra;
```

| Field | Description |
| ----- | ----------- |
| `label` | Tab label shown in the header. |
| `path` | URL segment relative to the record route. Use `''` or omit for the default tab (overview). |
| `component` | Render function receiving `{ resource, bundle, reload }`. |

`PageTabs` derives the active tab from the current pathname and navigates with React Router `Link`. Each tab registers two routes — `/{path}` and `/{path}/*` — so a tab can host its own nested `<Routes>` (for example, a documents list with sub-routes for create and edit).

**URL examples** for route `/patients/:id/*`:

| Tab `path` | Resolved URL |
| ---------- | ------------ |
| `''` | `/patients/123` |
| `documents` | `/patients/123/documents` |
| `documents/new/allergies` | `/patients/123/documents/new/allergies` (nested routes inside the tab) |

## Pairing with list pages

Detail pages are typically opened from a [`ResourceListPage`](./resource-list-page.md) via a `navigationAction` row action. Register both routes in `EMR`:

```tsx
<Route path="/medications" element={<MedicationManagementList />} />
<Route path="/medications/:id/*" element={<MedicationManagementDetail />} />
```

## Examples

### Medication knowledge detail

[`MedicationManagementDetail`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/MedicationManagementDetail/index.tsx) — a single overview tab with a nested `ResourceListPageContent` for related `Medication` batches:

```tsx
const tabs: Array<Tab<MedicationKnowledge>> = [
    {
        path: '',
        label: t`Overview`,
        component: (context) => <MedicationKnowledgeOverview resource={context.resource} />,
    },
];

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
```

### Patient detail with multiple tabs

[beda.fhirlab.net](https://github.com/beda-software/beda.fhirlab.net) uses `ResourceDetailPage` for a custom patient chart with overview, documents, and apps tabs. The documents tab mounts its own nested routes for list, create, and detail views:

```tsx
const tabs: Array<Tab<WithId<Patient>>> = [
    { path: '', label: 'Overview', component: ({ resource }) => <PatientOverview patient={resource} /> },
    { path: 'documents', label: 'Documents', component: ({ resource }) => <Documents patient={resource} /> },
    { path: 'apps', label: 'Apps', component: ({ resource }) => <PatientApps patient={resource} /> },
];

<PatientDashboardProvider dashboard={dashboard}>
    <ResourceDetailPage<Patient>
        resourceType="Patient"
        getSearchParams={({ id }) => ({ _id: id })}
        getTitle={({ resource, bundle }) => getName(resource, { bundle })!}
        tabs={tabs}
        maxWidth="100%"
    />
</PatientDashboardProvider>
```

Full example: [`src/containers/PatientsUberList/detail.tsx`](https://github.com/beda-software/beda.fhirlab.net/blob/main/src/containers/PatientsUberList/detail.tsx).

## Clinical context

By default, `ResourceDetailPage` exposes the loaded primary resource in page-level `ClinicalContext` via `getRecordClinicalContextDefault`. Override with `getClinicalContext` when questionnaires need different parameter names or additional resources.

Tab components and nested lists inside tabs inherit this context automatically. For nested `ResourceListPageContent` with their own questionnaire actions, provide a separate `getClinicalContext` on the list — see [Clinical context — Resource detail pages](./clinical-context.md#resource-detail-pages-resourcedetailpage).

## Related documentation

- [Resource list page](./resource-list-page.md) — standalone and nested FHIR resource lists
- [EMR component](./emr-component.md) — register detail routes in `authenticatedRoutes`
- [Clinical context](./clinical-context.md) — `getClinicalContext` on detail and nested list pages
- [Custom EMR build](./custom-emr-build.md) — project setup and customization entry points
