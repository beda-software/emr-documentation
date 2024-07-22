# Smile CDR integration

This doc will help you to launch Beda EMR with Smile HAPI FHIR Server

## Authentication

### OAuth2

Scopes:
- openid
- fhirUser
- profile
- launch/practitioner
- patient/*.read
- patient/*.write
- offline_access