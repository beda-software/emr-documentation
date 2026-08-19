---
sidebar_position: 7
---

# ViewChart component

The `ViewChart` component renders a time-series or categorical chart backed by Aidbox `ViewDefinition` or `Library` (SQL on FHIR SQLQuery) resources. It fetches rows from the given data source, sorts them, turns them into a chart configuration, and renders the result — handling loading and error states along the way. Use it for patient dashboard widgets that chart lab results, questionnaire scores, or any other tabular data Aidbox can produce.

Source: [`src/uberComponents/ViewChart/index.tsx`](https://github.com/beda-software/fhir-emr/blob/main/src/uberComponents/ViewChart/index.tsx#L15)

Import:

```tsx
import { ViewChart } from 'src/uberComponents/ViewChart';
import type { ReferenceChartRow, ViewChartConfig, ViewChartProps } from 'src/uberComponents/ViewChart';
```

## What ViewChart does

1. Calls `useViewChartRows` to fetch rows from `source`, passing `parameters` as run/query parameters. The fetch re-runs when `source.type`, `source.reference`, or `parameters` change.
2. Sorts the loaded rows with `sort` (defaults to `sortByAxisLabel`, which orders by `axis_label`).
3. Renders `RenderRemoteData` around the result: a spinner while loading, an `Alert` on failure, and the chart once data has loaded.
4. Resolves `chart` into a `ViewChartConfig` — either used as-is or computed by calling it with the loaded rows — and builds a `Chart` element from `config.transform(data)`.
5. Renders that element directly, or via `renderChart` if provided, so callers can wrap the chart in their own card, row count, or layout.

## Props

```tsx
type ViewChartProps<TRow extends ReferenceChartRow> = {
    source: ViewChartDataSource;
    parameters?: ViewDefinitionRunParameter[];
    sort?: (a: TRow, b: TRow) => number;
    chart?: ViewChartConfig<TRow> | ((rows: TRow[]) => ViewChartConfig<TRow>);
    onPointClick?: (datum: ChartDatumBase) => void;
    renderChart?: (chart: ReactNode, config: ViewChartConfig<TRow>, data: TRow[]) => ReactNode;
};
```

| Prop | Required | Description |
| ---- | -------- | ----------- |
| `source` | Yes | `{ type: 'ViewDefinition' \| 'Library', reference: string }` — which resource to run and its id, e.g. `{ type: 'ViewDefinition', reference: 'ViewDefinition/creatinine-observations' }`. `Library` is the way to run a custom SQL query. |
| `parameters` | No | FHIR `Parameters.parameter` entries passed to `$run` (ViewDefinition) or `$sqlquery-run` (Library, nested under a `parameters` input parameter). Typically includes the patient reference. |
| `sort` | No | Client-side row comparator applied after fetch. Defaults to `sortByAxisLabel`. |
| `chart` | No | A `ViewChartConfig`, or a function from loaded rows to one. Defaults to `buildReferenceChart`, which derives title, reference-range bands, and y-axis domain from the rows themselves. |
| `onPointClick` | No | Called with the clicked datum — useful for navigating to the source document (see the HMB example below). |
| `renderChart` | No | Wraps the rendered chart element. Receives the chart element, the resolved config (for `config.title`, etc.), and the raw rows (for counts or empty states). Defaults to rendering the chart unwrapped. |

### `ReferenceChartRow`

Rows returned by the data source must match this shape — it's the common column set `ViewDefinition` and `Library` sources are expected to project:

```tsx
interface ReferenceChartRow {
    id: string;
    axis_label: string;
    title: string | null;
    reference_range: (ObservationReferenceRange | string)[] | null;
    value_code: string | null;
    value_integer: number | null;
    value_quantity: number | null;
}
```

`id` and `axis_label` drive default sorting and x-axis labels; `value_code` / `value_integer` / `value_quantity` are alternate value slots depending on the resource being charted; `reference_range` feeds the default reference-band shading in `buildReferenceChart`.

## Examples

### CreatinineDashboard — a single chart with a computed config

[`src/components/DashboardCard/creatinine.tsx`](https://github.com/beda-software/fhir-emr/blob/main/src/components/DashboardCard/creatinine.tsx) renders one `ViewChart` against a fixed `ViewDefinition`, alongside a form for recording new creatinine readings:

```tsx
export function CreatinineDashboard({ patient }: Props) {
    const [refreshKey, setRefreshKey] = useState(0);
    const chart = buildCreatinineChart(patient.gender);

    return (
        <S.Card>
            {/* ...header omitted... */}
            <ViewChart<ReferenceChartRow>
                key={refreshKey}
                source={{ type: 'ViewDefinition', reference: 'ViewDefinition/creatinine-observations' }}
                parameters={
                    patient.id ? [{ name: 'patient', valueReference: { reference: `Patient/${patient.id}` } }] : []
                }
                chart={chart}
                renderChart={(chartElement, _config, data) => (
                    <div style={{ flex: 1 }}>
                        {data.length > 0 ? <div>{t`Total ${data.length}`}</div> : null}
                        {chartElement}
                    </div>
                )}
            />
            <QuestionnaireResponseForm
                initialQuestionnaireResponse={{
                    resourceType: 'QuestionnaireResponse',
                    questionnaire: 'creatinine',
                    subject: { reference: `Patient/${patient.id}` },
                }}
                questionnaireLoader={questionnaireIdLoader('creatinine')}
                onSuccess={() => setRefreshKey((key) => key + 1)}
            />
        </S.Card>
    );
}
```

Key points:

- `chart` is a **function of `patient.gender`**, not of the loaded rows — the normal creatinine range differs by gender, and gender isn't part of the fetched data. `buildCreatinineChart` closes over it and returns a `ViewChartConfig` factory.
- `renderChart` is used only to prepend a row count above the chart; the chart itself is still rendered by `ViewChart`.
- Bumping `refreshKey` (as a React `key`) after the form submits re-mounts `ViewChart`, forcing a re-fetch so the new reading appears immediately.

### HMBDiagnosticDashboard — multiple charts from a shared config array

[`src/containers/PatientDetails/HMBDiagnostic/HMBDiagnosticDashboard.tsx`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/PatientDetails/HMBDiagnostic/HMBDiagnosticDashboard.tsx) drives several `ViewChart` instances from one config array, mixing `ViewDefinition` and `Library` sources:

```tsx
export function HMBDiagnosticDashboard({ patient }: { patient: Patient }) {
    const navigate = useNavigate();
    const [refreshKey, setRefreshKey] = useState(0);

    const onPointClick = (datum: ChartDatumBase) => {
        const { qrId } = datum as HMBChartDatum;
        navigate(`/patients/${patient.id}/documents/${qrId}`);
    };

    return (
        <S.Grid>
            {getHMBCharts().map((entry) => (
                <ViewChart<ReferenceChartRow>
                    key={`${entry.id}-${refreshKey}`}
                    source={entry.source}
                    parameters={patient.id ? entry.parameters(patient.id) : []}
                    chart={entry.config}
                    onPointClick={onPointClick}
                    renderChart={(chart, config) => (
                        <DashboardCard title={config.title} icon={entry.icon}>
                            {chart}
                        </DashboardCard>
                    )}
                />
            ))}
        </S.Grid>
    );
}
```

`getHMBCharts()` (in [`config.tsx`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/PatientDetails/HMBDiagnostic/config.tsx)) returns one entry per chart, each with its own `source`, `parameters`, `icon`, and `config`:

```tsx
{
    id: 'flow-volume',
    source: { type: 'ViewDefinition', reference: 'ViewDefinition/hmb-flow-volume' },
    icon: <BarChartOutlined />,
    parameters: viewDefinitionPatientParameters,
    config: { variant: 'bar', transform: toFlowVolumeWithAxis(flow) /* ... */ },
},
{
    id: 'pain-severity-score',
    source: { type: 'Library', reference: 'Library/hmb-pain-severity-score' },
    icon: <HeartOutlined />,
    parameters: sqlQueryPatientParameters,
    config: buildPainSeverityAndScoreChart, // config as a function of rows
},
```

Key points:

- `ViewDefinition` sources here are filtered server-side with a `patient` FHIR reference parameter; the `Library` source instead takes the bare patient id as a `valueString`, matching how its SQL query reads the `:patient` parameter (see [ViewDefinition and Library](#viewdefinition-and-library) below).
- `onPointClick` is used for navigation: each row's `id` is carried through as `qrId` in the transformed datum, so clicking a point opens the `QuestionnaireResponse` document that produced it.
- `renderChart` here wraps every chart in the shared `DashboardCard`, reading the title from the resolved `config` rather than hardcoding it per entry.
- As in the creatinine example, `key={entry.id}-${refreshKey}` forces a re-fetch of all charts after a new questionnaire response is submitted.

## Using ViewChart in a patient dashboard

The patient dashboard (`src/containers/PatientDetails/Dashboard/config.ts`) is a `DashboardInstance` — an object with `top` / `left` / `right` / `bottom` arrays of `WidgetInfo`, each `{ widget, query? }`. `ViewChart`-based widgets slot in the same way as any other dashboard card, as long as they're wrapped in a component matching `WidgetProps` (`{ patient, widgetInfo }`). See [Dashboard page](./dashboard-page.md) for how the dashboard is wired up and the other widget patterns it supports.

[`CreatinineDashboardContainer`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/PatientDetails/PatientOverviewDynamic/containers/CreatinineDashboardContainer/index.tsx) is that wrapper for the creatinine example above — it just forwards `patient` to `CreatinineDashboard`:

```tsx
export function CreatinineDashboardContainer({ patient }: ContainerProps) {
    return <CreatinineDashboard patient={patient} />;
}
```

It's registered in the `bottom` area of the dashboard config ([`config.ts`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/PatientDetails/Dashboard/config.ts#L97)):

```tsx
export const patientDashboardConfig: DashboardInstance = {
    top: [
        // AppointmentCardContainer, GeneralInformationDashboardContainer,
        // StandardCardContainerFabric(prepareConditions), ... (query-driven cards)
    ],
    left: [],
    right: [],
    bottom: [
        {
            widget: CreatinineDashboardContainer,
        },
    ],
};
```

Unlike the query-driven `top` widgets (which declare a FHIR `search` and get pre-fetched resources), a `ViewChart`-based widget fetches its own data — it needs no `query` entry, only `widget`. Follow this pattern to add your own `ViewChart` dashboards: build a small `WidgetProps`-compatible container, place it in whichever dashboard area fits, and let `ViewChart` handle the fetch and render internally.

## ViewDefinition and Library

`ViewChart`'s `source.type` selects which Aidbox operation supplies rows:

- **`ViewDefinition`** — a [SQL on FHIR](https://www.health-samurai.io/docs/aidbox/modules/sql-on-fhir) `ViewDefinition` resource, projected into flat rows via the [`$run` operation](https://www.health-samurai.io/docs/aidbox/modules/sql-on-fhir/operation-run) (`POST /ViewDefinition/:id/$run`). `ViewChart` calls it with `parameters` plus `{ name: '_format', valueCode: 'json' }`. Use it when the data lives directly on a FHIR resource and can be expressed as a `where`/`select` view. Example — [`ViewDefinition/creatinine-observations`](https://github.com/beda-software/fhir-emr/blob/main/resources/init-seeds/ViewDefinition/creatinine-observations.yaml), which projects one row per creatinine `Observation`:

  ```yaml
  resourceType: ViewDefinition
  id: creatinine-observations
  url: ViewDefinition/creatinine-observations
  resource: Observation
  where:
    - path: code.coding.where(system = 'http://loinc.org' and code = '2160-0').exists()
  select:
    - column:
        - name: id
          path: getResourceKey()
          type: string
        - name: axis_label
          path: effective.ofType(dateTime)
          type: dateTime
        - name: title
          path: "'Creatinine'"
          type: string
        - name: value_quantity
          path: value.ofType(Quantity).value.first()
          type: decimal
  ```

- **`Library`** — a [SQL on FHIR SQLQuery](https://build.fhir.org/ig/HL7/sql-on-fhir/StructureDefinition-SQLQuery.html) encoded as a `Library` resource, run via the [`$sqlquery-run` operation](https://build.fhir.org/ig/HL7/sql-on-fhir/OperationDefinition-SQLQueryRun.html) (`POST /Library/:id/$sqlquery-run`). This is the recommended way to write a custom SQL query — use it when a single FHIR resource type can't express the shape you need, e.g. joining rows from more than one source, as in the HMB example, where pain severity and pain score come from two different `ViewDefinition`-backed views (`ViewDefinition/hmb-pain-severity`, `ViewDefinition/hmb-pain-score`) and are combined by `id`. The `Library`'s `relatedArtifact` entries declare which `ViewDefinition`s the query depends on and give each a `label` that the SQL text uses as its table name (`FROM severity`/`FULL JOIN score`, not the raw `sof.*` schema). Parameters are declared under `parameter` and substituted as `:name` placeholders; `ViewChart` sends `parameters` as a nested `Parameters` resource under the `parameters` input parameter of the request body (rather than as flat query-string params). Example — [`Library/hmb-pain-severity-score`](https://github.com/beda-software/fhir-emr/blob/main/resources/init-seeds/Library/hmb-pain-severity-score.yaml):

  ```yaml
  resourceType: Library
  id: hmb-pain-severity-score
  status: active
  name: hmb_pain_severity_score
  title: Period Pain Severity & Score
  type:
    coding:
      - system: https://sql-on-fhir.org/ig/CodeSystem/LibraryTypesCodes
        code: sql-query
  relatedArtifact:
    - type: depends-on
      resource: ViewDefinition/hmb-pain-severity
      label: severity
    - type: depends-on
      resource: ViewDefinition/hmb-pain-score
      label: score
  parameter:
    - name: patient
      use: in
      type: string
  content:
    - contentType: application/sql
      extension:
        - url: https://sql-on-fhir.org/ig/StructureDefinition/sql-text
          valueString: >-
            SELECT
              COALESCE(severity.id, score.id) AS id,
              COALESCE(severity.axis_label, score.axis_label) AS axis_label,
              'Period Pain Severity & Score' AS title,
              NULL AS reference_range,
              severity.value_code AS value_code,
              score.value_integer AS value_integer,
              NULL AS value_quantity
            FROM severity
            FULL JOIN score ON severity.id = score.id
            WHERE COALESCE(severity.patient_id, score.patient_id) = :patient
      data: <base64-encoded copy of the same SQL text>
  ```

  A `ViewDefinition` referenced by `relatedArtifact` needs its own `url` (e.g. `url: ViewDefinition/hmb-pain-severity`) so the `Library` can resolve it by that dependency.

Whichever type you use, the returned columns must line up with `ReferenceChartRow` (or a superset of it, as `HMBChartMeta`/`HMBChartDatum` do) — `id`, `axis_label`, `title`, `reference_range`, and the relevant `value_*` column — since that's what `ViewChart`'s default sort and `buildReferenceChart` config expect. See the [SQL on FHIR reference](https://www.health-samurai.io/docs/aidbox/modules/sql-on-fhir/defining-flat-views-with-view-definitions) and the [SQL on FHIR SQLQuery spec](https://build.fhir.org/ig/HL7/sql-on-fhir/StructureDefinition-SQLQuery.html) docs for the full `ViewDefinition`/`Library` syntax.

In a [custom EMR build](./custom-emr-build.md), add your `ViewDefinition`/`Library` YAML files to `contrib/fhir-emr/resources/init-seeds/` so Aidbox loads them on startup, the same way [questionnaire actions](./questionnaire-actions.md) load `Questionnaire`/`Mapping` resources.

## Related documentation

- [Dashboard page](./dashboard-page.md) — `Dashboard`/`DashboardInstance` config, `Dashboards` rendering, and the other widget patterns (`StandardCardContainerFabric`, bespoke query-driven widgets)
- [Resource detail page](./resource-detail-page.md) — tabbed FHIR resource detail layout, another place chart-like widgets are commonly embedded
- [Questionnaire actions](./questionnaire-actions.md) — Questionnaire + Mapping pairs, including how init-seed resources are loaded
- [Custom EMR build](./custom-emr-build.md) — project template, including the Aidbox init-seeds volume
