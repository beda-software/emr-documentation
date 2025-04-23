---
sidebar_position: 1
---

# Getting started with Beda EMR 

[![beda-emr-logo](https://user-images.githubusercontent.com/6428960/222070888-a97e2d97-7eb0-4cb3-8310-5fdb7b56aa10.svg)](https://beda.software/emr)

### Design
EMR design is open source and publicly available ->
[Figma](https://www.figma.com/file/2bxMDfG3lRPEZpRwDC4gTB/SaaS-EMR-System)

### Demo
The easiest way to check the system is our publicly availabe demo imstalation: [emr.beda.software](https://emr.beda.software/)
[This YouTube playlist](https://www.youtube.com/watch?v=k1qDO8qBPWw&list=PLrnm9AbXp-mhg1_26EGBxBjtTd6j-M7rt) contains all basic scenarios, you can try all of them in the demo environment.



## License
The EMR source code is licensed by [MIT License](https://github.com/beda-software/fhir-sdc/blob/master/LICENSE).  

## FHIR Backend
Beda EMR is a frontend. It is a user interface that requires a FHIR server to store medical data.  
For both development and production environments, we recommend using Aidbox FHIR Server.  
It is a primary backend platform for Beda EMR.
You can get a free Aidbox trial license to run the application locally.  
You need to purchase Aidbox license for any production installation or installation that manages PHI data.  
[Here](https://docs.aidbox.app/getting-started/editions-and-pricing) you can find more information about Aidbox licensing.  
Obviously, you can try any other FHIR server. All core features just need FHIR API.  
However, you have to adjust some parts of the application that are not covered in the FHIR specification and where we use Aidbox API.  

## Installation

### Setup env variables

```
cp .env.tpl .env
# Get Aidbox license at https://aidbox.app/ and place license JWT to .env
```

### Local setup

#### Video calls local setup
Before you start you need to set up your own Jitsi Meet video instance. See this [guide from Jitsi](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker/)

Also, you need to configure JWT authentication on the video server side. See this [guide](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker/#authentication-using-jwt-tokens)

**Important note**: We use react component to represent the video call frame from [jitsi-meet-react-sdk](https://github.com/jitsi/jitsi-meet-react-sdk/tree/main). This component requires HTTPS schema for the Jitsi server, so you need to publish your server with this requirement

In the EMR folder add these variables to `.env` file with values you generated on the video server side:

```
# Application identifier
AUTH_JWT_SECRET=
AUTH_JWT_ACCEPTED_ISSUERS=
AUTH_JWT_ACCEPTED_AUDIENCES=
```

To run EMR with our Jitsi authentication backend service run:
`make up-video`

#### Prepare frontend configuration

```
cp contrib/emr-config/config.local.js contrib/emr-config/config.js
```

This file (`contrib/emr-config/config.js`) is ignored by git. So, feel free to change it.

```sh
yarn
yarn compile
```

#### Docker
You need to set up docker to run Aidbox, SDC, and other microservices. https://docker.com/  
Once you get docker installed in your local machine, you can run all project dependencies
```sh
make build-seeds
make up
```
The first start may take a few minutes since it synchronizes the terminology.

### Start

```sh
yarn start           # start watch all workspaces
```

### Test

```sh
yarn test            # launch tests for all workspaces
```

## Update seeds

To see the changes that were added to `resources/seeds` follow the next steps

```sh
make seeds
```

## Project History

Started as part of [https://github.com/HealthSamurai/xmas-hackathon-2021](https://github.com/HealthSamurai/xmas-hackathon-2021/issues/13) FHIR EMR evolved into something bigger.
