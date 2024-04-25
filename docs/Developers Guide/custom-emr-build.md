# Custom EMR build
Beda EMR is designed to be a framework for building EHR and EMR solutions on top of it. This article describes how you can build your own custom version of Beda EMR suitable for your needs.

1. Initialize the project. We recommend using [vitejs](https://vitejs.dev/) and [yarn](https://yarnpkg.com/), but you can use any other tool for building the frontend.
```
yarn create vite@latest  your_awesome_emr -- --template react-ts
```

2. Add https://github.com/beda-software/fhir-emr as a submodule to your repository.
Use relative name `../../beda-software/fhir-emr` for repositories hosted on GitHub, or absolute URL in other cases.
```
git submodule add ../../beda-software/fhir-emr ./contrib/fhir-emr/
```

In tsconfig.json specify `src/*` and `shared/*` to be seached in fhir-emr submodule. 
*Now fhir-emr uses relative paths from src/ it will be fixed in the future to use @fhir-emr namespace. But now you have to admit that src and shared reserved form emr components*
```diff
@@ -18,7 +18,15 @@
     "strict": true,
     "noUnusedLocals": true,
     "noUnusedParameters": true,
-    "noFallthroughCasesInSwitch": true
+    "noFallthroughCasesInSwitch": true,
+    "allowSyntheticDefaultImports": true,
+
+    /* Modules */
+    "baseUrl": ".",
+    "paths": {
+      "src/*": ["./contrib/fhir-emr/src/*"],
+      "shared/*": ["./contrib/fhir-emr/shared/*"]
+    },
   },
   "include": ["src"],
   "references": [{ "path": "./tsconfig.node.json" }]
```
Also, update tsconfig.json 
```diff
@@ -4,8 +4,7 @@
     "skipLibCheck": true,
     "module": "ESNext",
     "moduleResolution": "bundler",
-    "allowSyntheticDefaultImports": true,
-    "strict": true
+    "allowSyntheticDefaultImports": true
   },
   "include": ["vite.config.ts"]
 }
```
and vite.config.ts
```typescript
import { createRequire } from 'module'
import * as path from 'path'

import react from '@vitejs/plugin-react'
import { defineConfig } from 'vite'

const require = createRequire(import.meta.url)

// https://vitejs.dev/config/
export default defineConfig(({ command }) => ({
    server: {
        port: command === 'build' ? 5000 : 3000,
    },
    plugins: [
        react({
            babel: {
                plugins: [
                    'macros',
                    [
                        'babel-plugin-styled-components',
                        {
                            displayName: true,
                            fileName: false,
                        },
                    ],
                ],
            },
        }),
    ],
    resolve: {
        alias: [
            { find: 'src', replacement: path.resolve(__dirname, './contrib/fhir-emr/src/') },
            { find: 'shared', replacement: path.resolve(__dirname, './contrib/fhir-emr/shared/') },
        ],
    },
    define: {
        'process.env': {},
    },
    build: {
        outDir: path.resolve(__dirname, 'build'),
        commonjsOptions: {
            defaultIsModuleExports(id) {
                try {
                    const module = require(id)
                    if (module?.default) {
                        return false
                    }
                    return 'auto'
                } catch (error) {
                    return 'auto'
                }
            },
            transformMixedEsModules: true,
        },
    },
}))
```
3. Copy dependencies and devDependencies from [fhir-emr](https://github.com/beda-software/fhir-emr/blob/master/package.json) into your package.json file
4. Add i18n configuration:
Add lingui.config.ts
```typescript
import type { LinguiConfig } from '@lingui/conf';

const config: LinguiConfig = {
    locales: ['en'],
    format: 'po',
    catalogs: [
        {
            path: '<rootDir>/shared/src/locale/{locale}/messages',
            include: ['<rootDir>'],
            exclude: ['**/node_modules/**'],
        },
    ],
};

export default config;
```
Add command to package.json
```diff
@@ -7,7 +7,9 @@
        "dev": "vite",
        "build": "tsc && vite build",
        "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
-       "preview": "vite preview"
+       "preview": "vite preview",
+       "extract": "lingui extract",
+       "compile": "lingui compile --typescript"
    },
    "dependencies": {
        "@ant-design/colors": "^7.0.0",
```
6. Inside the src directory create a prefixed directory for your EMR customization. For example, let's call it '`yourAwesomeEmr`.
7. Creare entrypoint into EMR `src/yourAwesomeEmr/containers/App/index.tsx`. In this file, you should define all contexts and specify routes.
```typescript
import { i18n } from '@lingui/core';
import { I18nProvider } from '@lingui/react';
import { useEffect } from 'react';
import { dynamicActivate, getCurrentLocale } from 'shared/src/services/i18n';

import { dashboard } from 'src/dashboard.config';
// You can copy dashboard into your project workspace and adjust appearance and widgets.
// Use https://github.com/beda-software/fhir-emr/blob/master/src/dashboard.config.ts as example;

import { ThemeProvider } from 'src/theme/ThemeProvider';
// You can specify your own theme to ajdust color,
// Use you https://github.com/beda-software/fhir-emr/blob/master/src/theme/ThemeProvider.tsx as example

import { App } from 'src/containers/App';
import { PatientDashboardProvider } from 'src/components/Dashboard/contexts';

export const AppWithContext = () => {
    useEffect(() => {
        dynamicActivate(getCurrentLocale())
    }, [])

    return (
        <I18nProvider i18n={i18n}>
            <PatientDashboardProvider dashboard={dashboard}>
                <ThemeProvider>
                    <App />
                </ThemeProvider>
            </PatientDashboardProvider>
        </I18nProvider>
    )
}

```
6. Update `main.tsx` to use a new entrypoint.
```typescript
import React from 'react'
import ReactDOM from 'react-dom/client'
import {AppWithContext} from './rsdue/containers/App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <AppWithContext />
  </React.StrictMode>,
)
```
7. Prepare to run
```
yarn
```
compile translations for your app and fhir-emr
```
yarn compile
cd contrib/fhir-emr/
yarn
yarn  compile
```
Use configuration file for local development
```
contrib/fhir-emr/shared/src/config.local.ts contrib/fhir-emr/shared/src/config.ts
```
8. Run
```
yarn dev
```

Now you have fhir-emr under your full control. You can replace the patient dashboard and theme as a first step of customization.
Then you can copy the whole https://github.com/beda-software/fhir-emr/blob/master/src/containers/App/index.tsx into your workspace to adjust routes and adjust page components. 
