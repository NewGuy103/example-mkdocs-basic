# API Overview

This page will define how to call the REST APIs provided in `syncserver.server`.

The base URL for all API endpoints is:

```
http://localhost:$SYNCSERVER_PORT
```

## Authentication

All API requests require authentication. You can use these headers for username-password authentication:

- **syncServer-Username**: YOUR-USERNAME
- **syncServer-Token**: YOUR-PASSWORD

Or an `Authorization` header for API key authentication:

- **Authorization**: YOUR-API-KEY

Replace the placehodlers with the proper credentials.

## Endpoints

### `/upload`

Data

