---
sidebar_position: 8
---

# Dashboard page

The patient dashboard is the widget grid shown on the patient overview tab. A `Dashboard` config maps roles (or `default`) to a `DashboardInstance` — four widget lists (`top`, `left`, `right`, `bottom`) — and the `Dashboards` component renders each list as a sequence of widget components.

Sources:

- [`src/components/Dashboard/contexts.tsx`](https://github.com/beda-software/fhir-emr/blob/main/src/components/Dashboard/contexts.tsx)
- [`src/components/Dashboard/types.ts`](https://github.com/beda-software/fhir-emr/blob/main/src/components/Dashboard/types.ts)
- [`src/components/Dashboard/index.tsx`](https://github.com/beda-software/fhir-emr/blob/main/src/components/Dashboard/index.tsx)
- [`src/containers/PatientDetails/Dashboard/config.ts`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/PatientDetails/Dashboard/config.ts)

Import:

```tsx
import { Dashboards } from 'src/components/Dashboard';
import { PatientDashboardProvider, useDashboard } from 'src/components/Dashboard/contexts';
import type { Dashboard, DashboardInstance, WidgetInfo, WidgetProps, Query } from 'src/components/Dashboard/types';
```

## How it's wired up

1. [`src/dashboard.config.ts`](https://github.com/beda-software/fhir-emr/blob/main/src/dashboard.config.ts) builds the top-level `Dashboard` object, currently just `{ default: patientDashboardConfig }` from `patientDashboardConfig` in [`config.ts`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/PatientDetails/Dashboard/config.ts).
2. [`main.tsx`](https://github.com/beda-software/fhir-emr/blob/main/src/main.tsx) wraps the app in `<PatientDashboardProvider dashboard={dashboard}>`, putting that config on `PatientDashboardContext` for the whole tree.
3. [`PatientOverview`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/PatientDetails/PatientOverviewDynamic/index.tsx) calls `useDashboard()` to read the resolved `DashboardInstance`, then renders one `Dashboards` per area:

   ```tsx
   const patientDashboard = useDashboard();

   <S.Container>
       <Dashboards widgets={patientDashboard.top} patient={patient} />
       <S.Cards>
           <S.Column>
               <Dashboards widgets={patientDashboard.left} patient={patient} />
           </S.Column>
           <S.Column>
               <Dashboards widgets={patientDashboard.right} patient={patient} />
           </S.Column>
       </S.Cards>
       <Dashboards widgets={patientDashboard.bottom} patient={patient} />
   </S.Container>
   ```

`Dashboards` itself just maps a `WidgetInfo[]` to mounted widget components, passing `patient` and the originating `widgetInfo` through:

```tsx
export function Dashboards({ patient, widgets }: Props) {
    return (
        <>
            {widgets.map((widgetInfo, index) => {
                const WidgetComponent = widgetInfo.widget;
                return <WidgetComponent key={index} patient={patient} widgetInfo={widgetInfo} />;
            })}
        </>
    );
}
```

`top` and `bottom` render full-width in document order; `left` and `right` render as two columns between them — that's a layout convention of `PatientOverview`'s styles, not something `Dashboards` enforces itself.

## `useDashboard` and role-based dashboards

```tsx
export interface Dashboard {
    default: DashboardInstance;
}

export function useDashboard() {
    const patientDashboard = useContext(PatientDashboardContext);
    // TODO select dashboard based on the role
    return patientDashboard.default;
}
```

`Dashboard` is keyed by role today only in shape — `useDashboard` always returns `patientDashboard.default`, with a `TODO` marking role-based selection as unimplemented. `dashboard.config.ts` already shows the intended extension point, commented out:

```tsx
export const dashboard: Dashboard = {
    default: patientDashboardConfig,
    // [Role.Admin]: {},
    // [Role.Practitioner]: {},
};
```

Until role-based selection lands, every signed-in user sees the same `default` dashboard regardless of role.

## `WidgetInfo` and `Query`

```tsx
export type DashboardAreas = 'top' | 'right' | 'left' | 'bottom';
export type DashboardInstance = Record<DashboardAreas, WidgetInfo[]>;

export interface WidgetInfo {
    widget: React.FunctionComponent<WidgetProps>;
    query?: Query;
}

export interface WidgetProps {
    patient: Patient;
    widgetInfo: WidgetInfo;
}

export interface ContainerProps {
    patient: Patient;
    widgetInfo: WidgetInfo;
}

export interface Query {
    resourceType: FhirResource['resourceType'];
    search: (patient: Patient) => SearchParams;
}
```

Each entry in a `DashboardInstance` area is a `{ widget, query? }` pair:

- `widget` — a component matching `WidgetProps` (`{ patient, widgetInfo }`, aliased as `ContainerProps` in widget containers). `Dashboards` mounts it directly.
- `query` — optional. When present, it's the widget's contract for what to fetch: a `resourceType` and a `search(patient)` function returning FHIR search params. `query` is only data attached to the config entry — nothing in `Dashboards` or `useDashboard` executes it. It's up to the `widget` component to read `widgetInfo.query` and fetch accordingly (see [`StandardCardContainerFabric`](#query-driven-widgets-standardcardcontainerfabric) and [`AppointmentCardContainer`](#bespoke-query-driven-widgets-appointmentcardcontainer) below).

A widget with no `query` fetches its own data internally, the way [`ViewChart`](./view-chart.md)-based widgets do.

## `patientDashboardConfig`

[`src/containers/PatientDetails/Dashboard/config.ts`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/PatientDetails/Dashboard/config.ts) is the `DashboardInstance` used as `dashboard.default`:

```tsx
export const patientDashboardConfig: DashboardInstance = {
    top: [
        {
            widget: AppointmentCardContainer,
            query: {
                resourceType: 'Appointment',
                search: (patient: Patient) => ({
                    patient: patient.id,
                    status: ['arrived,booked'],
                }),
            },
        },
        {
            widget: GeneralInformationDashboardContainer,
        },
        {
            query: {
                resourceType: 'Condition',
                search: (patient: Patient) => ({
                    patient: patient.id,
                    _sort: ['-_recorded-date'],
                    _revinclude: ['Provenance:target'],
                    _count: 7,
                }),
            },
            widget: StandardCardContainerFabric(prepareConditions),
        },
        // ...AllergyIntolerance, MedicationStatement, Immunization, Procedure follow the same shape
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

This shows the three widget patterns in the codebase:

| Pattern | Example | `query` | Data fetching |
| ------- | ------- | ------- | -------------- |
| Query-driven, generic card | `StandardCardContainerFabric(prepareConditions)` | Yes | `useStandardCard` runs `query.search(patient)` against `query.resourceType`, then formats the result with a `PrepareFunction` |
| Query-driven, bespoke widget | `AppointmentCardContainer` | Yes | Its own hook (`useAppointmentCard`) runs `widgetInfo.query` itself |
| Self-fetching widget | `GeneralInformationDashboardContainer`, `CreatinineDashboardContainer` | No | Widget (or the `ViewChart` it renders) fetches internally, ignoring `widgetInfo.query` |

## Query-driven widgets: `StandardCardContainerFabric`

Most `top`-area cards (conditions, allergies, medications, immunizations, procedures) share one generic renderer built by `StandardCardContainerFabric<T>(prepareFunction, cardProps?)`. It returns a `ContainerProps` component that:

1. Requires `widgetInfo.query` — renders an error `<div>` if missing.
2. Calls `useStandardCard(patient, query, prepareFunction)`, which runs `getFHIRResources(query.resourceType, { ...query.search(patient), _count: countNumber })`, extracts the bundle, and calls `prepareFunction(resources, bundle, total, to)` to build an `OverviewCard`.
3. Renders `StandardCard` with the resulting card via `RenderRemoteData` (spinner while loading).

A `PrepareFunction` turns raw FHIR resources into an `OverviewCard`:

```tsx
export type PrepareFunction<T extends Resource> =
    | ((resources: T[], bundle: Bundle<T>, total: number, to?: string) => OverviewCard<T>)
    | ((resources: T[]) => OverviewCard<T[]>);

export interface OverviewCard<T extends Resource | Resource[]> {
    title: string;
    key: string;
    icon: React.ReactNode;
    data: T[];
    total?: number;
    columns: { key: string; title: string; render: (r: T) => React.ReactNode; width?: string | number }[];
    getKey: (r: T) => string;
}
```

[`prepareConditions`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/PatientDetails/PatientOverviewDynamic/components/StandardCard/prepare.tsx) is one such function — given `Condition[]` and the search `Bundle`, it returns a title, icon, and column definitions (with `render` typically wrapping the value in `LinkToEdit` to open an edit questionnaire).

To add a new query-driven card of the same shape:

1. Write a `PrepareFunction<T>` for your resource type in `prepare.tsx` (or your own file).
2. Add an entry to `patientDashboardConfig` (or a custom config) with `widget: StandardCardContainerFabric(yourPrepareFunction)` and a `query` describing the resource and search params.

## Bespoke query-driven widgets: `AppointmentCardContainer`

When a widget's rendering doesn't fit the generic table shape (`AppointmentCard` renders custom cards, not `StandardCard` columns), it still follows the same `query`-driven contract but with its own hook instead of `useStandardCard`:

```tsx
export function AppointmentCardContainer(props: ContainerProps) {
    const { widgetInfo } = props;

    if (!widgetInfo.query) {
        return <div>Error: no query parameter for the widget.</div>;
    }

    return <AppointmentCardWrapper {...props} />;
}

function AppointmentCardWrapper({ patient, widgetInfo }: ContainerProps) {
    const { response } = useAppointmentCard(patient, widgetInfo.query!);

    return (
        <RenderRemoteData remoteData={response} renderLoading={Spinner}>
            {({ appointments }) => appointments.map((appointment, index) => (
                <AppointmentCard key={index} appointment={appointment} />
            ))}
        </RenderRemoteData>
    );
}
```

Use this pattern — read `widgetInfo.query` in your own hook — when you need custom rendering but still want the resource type and search params to live declaratively in the dashboard config rather than hardcoded in the widget.

## Self-fetching widgets

Widgets that don't need `query` fetch their own data and ignore `widgetInfo.query` entirely:

- `GeneralInformationDashboardContainer` calls `useGeneralInformationDashboard(patient)` internally and renders `GeneralInformationDashboard`.
- `CreatinineDashboardContainer` forwards `patient` to a [`ViewChart`](./view-chart.md)-based component, which fetches rows from a `ViewDefinition`/`Library` itself.

Prefer this pattern when the widget's data source isn't a single FHIR search (a `ViewChart`/SQL on FHIR chart, an aggregate, or a hook that composes several calls), since forcing it into `query.search(patient)` would just move the same logic into a function that has to run inside another hook anyway.

## Adding a widget to the dashboard

1. Build a component matching `WidgetProps`/`ContainerProps` (`{ patient, widgetInfo }`), reusing `StandardCardContainerFabric` if a generic card fits, or a bespoke container otherwise.
2. Add `{ widget, query? }` to the appropriate area (`top`, `left`, `right`, `bottom`) of `patientDashboardConfig` in `config.ts`.
3. In a [custom EMR build](./custom-emr-build.md), override `src/containers/PatientDetails/Dashboard/config.ts` (or `src/dashboard.config.ts` for role-based dashboards) to add, remove, or reorder widgets without touching the base component code.

## Related documentation

- [ViewChart component](./view-chart.md) — chart widgets backed by `ViewDefinition`/`Library`, and how they slot into `patientDashboardConfig`
- [Resource detail page](./resource-detail-page.md) — tabbed FHIR resource detail layout, another place `StandardCard`-style widgets are used
- [Custom EMR build](./custom-emr-build.md) — project template and override points for `dashboard.config.ts`
