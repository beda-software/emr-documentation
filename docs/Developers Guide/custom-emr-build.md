# Custom EMR build
Beda EMR is designe to be a framework for building EHR and EMR solutions on top of it. This article descrines how you can build your own custom version of Beda EMR sutable for your needs.

1. Add https://github.com/beda-software/fhir-emr as a submodule to your repository. In tsconfig.json specify `src/*` and `shared/*` to be seached in beda-emr submodule. 
*Now fhir-emr uses relative paths from src/ it will be fixed in the futurte to use @fhir-emr namespace. But niw you have to admit that src and shared reserved form emr components*
```json
{
  "compilerOptions": {
  ...
  "paths": {
      "src/*": ["./contrib/fhir-emr/src/*"],
      "shared/*": ["./contrib/fhir-emr/shared/*"]
    },
  }
}
```
3. Add fhir emr dependencies into your package.json file
4. Inside src directory create a prefixed directory for your emr customization. For example let's call it '`yourEmr`.
5. Creare entrypoint into EMR `src/yourEmr/containers/App/index.tsx`. In this file you should define all contexts and specify routes.
```typescript
import { i18n } from '@lingui/core';
import { I18nProvider } from '@lingui/react';
import React, { useEffect } from 'react';
import { dynamicActivate, getCurrentLocale } from 'shared/src/services/i18n';

import { dashboard } from 'src/dashboard.config';
// You can copy dashboard into your project workspace and adjust appearance and widgets.
// Use https://github.com/beda-software/fhir-emr/blob/master/src/dashboard.config.ts as example;

import { ThemeProvider } from 'src/theme/ThemeProvider.tsx';
// You can specify your own theme to ajdust color,
// Use you https://github.com/beda-software/fhir-emr/blob/master/src/theme/ThemeProvider.tsx as example

import { App } from 'src/containers/App';

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
6. Use this file in `main.tsx` as entrypoint.
7. Now you have fhir emr under your full control. You can replace patient dhasboard and theme as a firs step of customization.
Then you can copy the whole https://github.com/beda-software/fhir-emr/blob/master/src/containers/App/index.tsx ino your workspace to adjust rotes and adjust page components. 
