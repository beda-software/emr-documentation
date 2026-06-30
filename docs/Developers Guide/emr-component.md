# EMR component

The `EMR` component is the main application shell of [Beda EMR](https://github.com/beda-software/fhir-emr). Use it as the root of a custom EMR build when you want to compose your own routes, sidebar menu, and authentication flow while reusing Beda EMR's layout, session handling, and built-in pages.

Source: [`src/containers/EMR/index.tsx`](https://github.com/beda-software/fhir-emr/blob/main/src/containers/EMR/index.tsx)

Import:

```tsx
import { EMR } from '@beda.software/emr/containers';
```

## What EMR provides

On mount, `EMR`:

1. Reads the stored auth token and restores the user session (optionally via `populateUserInfoSharedState`).
2. Wraps the tree in `MenuLayout` and `FooterLayout` providers.
3. Renders either the **anonymous** or **authenticated** route tree inside `BrowserRouter`.

### Anonymous users

Unauthenticated visitors get a minimal route set:

- Your `anonymousRoutes` (typically `/signin` and OAuth callback routes).
- Built-in `/auth` route that parses the OAuth hash fragment and stores the access token.
- A catch-all redirect to `/signin`.

### Authenticated users

Signed-in users with at least one role receive:

- `ClinicalContext` for the session (see [Clinical context](./clinical-context.md)).
- Built-in routes:
  - `/print-patient-document/:id/:qrId` — document print view.
  - `/appointment/book` — public appointment booking (also available when anonymous via `App`).
- Your `authenticatedRoutes` rendered inside `BaseLayout` (sidebar + header).
- A catch-all redirect to the **first item** in the current `menuLayout()`.

Users with no roles see `UserWithNoRolesComponent` (defaults to `DefaultUserWithNoRoles`).

## Props

```tsx
interface EMRProps {
    menuLayout: MenuLayoutValue;
    authenticatedRoutes?: ReactElement;
    anonymousRoutes?: ReactElement;
    populateUserInfoSharedState?: () => Promise<RemoteDataResult<User>>;
    UserWithNoRolesComponent?: () => ReactElement;
    footer?: ReactElement;
    getAuthenticatedClinicalContext?: () => ParametersParameter[];
}
```

| Prop | Required | Description |
| ---- | -------- | ----------- |
| `menuLayout` | Yes | Function returning sidebar items for the current user role. First item's `path` becomes the default landing route. |
| `authenticatedRoutes` | No | `<Route>` elements available after login, rendered inside `BaseLayout`. |
| `anonymousRoutes` | No | `<Route>` elements for unauthenticated users (sign-in, OAuth callbacks). |
| `populateUserInfoSharedState` | No | Custom hook to load user info and populate shared state (e.g. role details). Called during session restore when a token exists. |
| `UserWithNoRolesComponent` | No | Component shown when the user is authenticated but has no roles assigned. |
| `footer` | No | Custom footer element. Falls back to `defaultFooterLayout`. |
| `getAuthenticatedClinicalContext` | No | Override session-level clinical context parameters. See [Clinical context](./clinical-context.md). |

### `menuLayout`

`MenuLayoutValue` is a function that returns an array of sidebar entries:

```tsx
type MenuLayoutValue = () => Array<{
    label: string;
    path: string;
    icon: React.ReactElement;
}>;
```

Use [`matchCurrentUserRole`](https://github.com/beda-software/fhir-emr/blob/main/src/utils/role.ts) to return different menus per role:

```tsx
import { matchCurrentUserRole, Role } from '@beda.software/emr/utils';
import { PatientsIcon } from '@beda.software/emr/icons';

const menuLayout: MenuLayoutValue = () =>
    matchCurrentUserRole({
        [Role.Admin]: () => [
            { label: t`Patients`, path: '/patients', icon: <PatientsIcon /> },
        ],
        [Role.Practitioner]: () => [
            { label: t`Patients`, path: '/patients', icon: <PatientsIcon /> },
        ],
        [Role.Patient]: () => [],
        [Role.Receptionist]: () => [],
    });
```

The sidebar reads from `MenuLayout` context. `EMR` calls `menuLayout()` when rendering authenticated routes to determine the default redirect path.

## Examples

### beda.fhirlab.net — custom routes with default Beda EMR containers

[beda.fhirlab.net](https://github.com/beda-software/beda.fhirlab.net) mounts `EMR` with custom list and detail pages while reusing Beda EMR theme, dashboard, and icons:

```tsx
<EMR
    menuLayout={menuLayout}
    populateUserInfoSharedState={populateUserInfoSharedState}
    anonymousRoutes={
        <>
            <Route path="/signin" element={<SignIn />} />
        </>
    }
    authenticatedRoutes={
        <>
            <Route path="/patients-ph" element={<PatientUberList />} />
            <Route path="/patients-ph/:id/*" element={<PatientDetails />} />
            <Route path="/encounters-ph" element={<EncountersUberList />} />
            {/* ... additional custom resource list routes ... */}
        </>
    }
/>
```

Full example: [`src/main.tsx` (line 170)](https://github.com/beda-software/beda.fhirlab.net/blob/main/src/main.tsx#L170).

This project also wraps `EMR` with providers for i18n, value set expansion, patient dashboard, and theme — patterns you can copy for localization and terminology customization.

### emr.au-core — environment-specific routes and menus

[emr.au-core](https://github.com/beda-software/emr.au-core) uses `EMR` with routes and sidebar items that change based on the connected FHIR server (Digital Health, Epic, Orion Health, etc.):

```tsx
<EMR
    authenticatedRoutes={renderRoutes()}
    anonymousRoutes={
        <>
            <Route path="/signin" element={<SignIn onSwitchService={setAuthProvider} />} />
            <Route path="/auth" element={<CodeGrantAuth />} />
            <Route path="/auth-aidbox" element={<ImplicitGrantAuth />} />
        </>
    }
    populateUserInfoSharedState={sharedUserInitService}
    menuLayout={getMenuLayout()}
/>
```

Full example: [`src/containers/App/index.tsx` (line 78)](https://github.com/beda-software/emr.au-core/blob/main/src/containers/App/index.tsx#L78).

`renderRoutes()` returns different `<Route>` sets per environment — for example, Digital Health admin pages vs. a default patient list. `getMenuLayout()` switches between `menuLayout` and `digitalHealthMenuLayout` using the same role-matching pattern.

Menu definitions live in [`src/containers/App/layout.tsx`](https://github.com/beda-software/emr.au-core/blob/main/src/containers/App/layout.tsx).

## Minimal setup

A minimal custom build needs initialization, a sign-in route, at least one authenticated route, and a menu:

```tsx
import { Route } from 'react-router-dom';
import { EMR } from '@beda.software/emr/containers';
import { PatientsIcon } from '@beda.software/emr/icons';
import { matchCurrentUserRole, Role } from '@beda.software/emr/utils';

import '@beda.software/emr/dist/services/initialize';
import '@beda.software/emr/dist/style.css';

import { SignIn } from './containers/SignIn';
import { PatientList } from './containers/PatientList';

const menuLayout = () =>
    matchCurrentUserRole({
        [Role.Admin]: () => [
            { label: 'Patients', path: '/patients', icon: <PatientsIcon /> },
        ],
        [Role.Practitioner]: () => [
            { label: 'Patients', path: '/patients', icon: <PatientsIcon /> },
        ],
        [Role.Patient]: () => [],
        [Role.Receptionist]: () => [],
    });

export function App() {
    return (
        <EMR
            menuLayout={menuLayout}
            anonymousRoutes={
                <Route path="/signin" element={<SignIn />} />
            }
            authenticatedRoutes={
                <Route path="/patients" element={<PatientList />} />
            }
        />
    );
}
```

For project scaffolding, see [Custom EMR build](./custom-emr-build.md).

## Reusing Beda EMR pages

You do not have to build every page from scratch. Import containers from `@beda.software/emr/containers` and register them as routes — the same way `App` does for encounters, patients, questionnaires, and scheduling:

```tsx
import {
    EncounterList,
    PatientDetails,
    PatientList,
    PractitionerList,
} from '@beda.software/emr/containers';

<Route path="/encounters" element={<EncounterList />} />
<Route path="/patients" element={<PatientList />} />
<Route path="/patients/:id/*" element={<PatientDetails />} />
<Route path="/practitioners" element={<PractitionerList />} />
```

Mix Beda EMR containers with your own list pages (`ResourceListPage`) and detail views (`ResourceDetailPage`) as needed. See [Resource detail page](./resource-detail-page.md) for tabbed detail layout, routing, and props.

## Related documentation

- [Custom EMR build](./custom-emr-build.md) — project template and quick start
- [Resource detail page](./resource-detail-page.md) — tabbed FHIR resource detail layout
- [Clinical context](./clinical-context.md) — session and page-level FHIR context for questionnaires
- [Form render engine](./form-engine.md) — using `@beda.software/fhir-questionnaire` outside Beda EMR
