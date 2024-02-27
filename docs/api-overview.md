# API Overview

This page will define how to call the REST APIs provided in `syncserver.server`.

The base URL for all API endpoints is:

```
http://localhost:$SYNCSERVER_PORT
```

All the endpoints only accept JSON (except for `/upload` and `/modify`).

## Authentication

All API requests require authentication. You can use these headers for username-password authentication:

- **syncServer-Username**: YOUR-USERNAME
- **syncServer-Token**: YOUR-PASSWORD

Or an `Authorization` header for API key authentication:

- **Authorization**: YOUR-API-KEY

Failing to authenticate or providing invalid credentials will result in these responses:

- **INVALID_APIKEY:**
  You passed an invalid API key to the server.

- **APIKEY_NOT_AUTHORIZED:**
  The API key you passed doesn't have the permission required to access this endpoint.

- **INVALID_CREDENTIALS:**
  Your username or password is incorrect.

## Endpoints

### `/upload`

Send a file to the server.

**Parameters:**

- Form data including a file, and the field name. (`remote_name=@local_filename`)

**Error Codes and Meaning:**

- **MISSING_FILES:** No files were provided in the request for upload.
    - The server did not receive any files, please check if the file data is being properly passed.

- **NO_REMOTE_PATH:** A remote path was not specified for the file.
    - The remote path was not passed when uploading. The server gets the parameter name as the remote path of the file: `/remote-path=@local-file.txt`.

- **INVALID_CONTENT:** The parameter name is not a proper data type.
    -  The server does not accept any parameter name type other than binary or string.

- **EMPTY_STREAM:** An empty file stream was passed to the server.
    - The server attempted to read from an empty file stream, so it rejects the file.

- **NO_DIR_EXISTS:** The directory path does not exist.
    - Within the parameter name, you can put the directory name in the syntax of a Unix path. If the directory path does not exist, then it will not continue writing: `/no-such-dir/remote-path`.

- **FILE_EXISTS:** The file already exists on the server.
    - The server has detected a file with the same name exists on the server, so it will not write to prevent data being overwritten. The `/modify` endpoint writes to existing files on the server.

### `/modify`

Modify an existing file on the server.

**Parameters:**

- Form data including a file, and the field name. (`remote_name=@local_filename`)

**Error Codes and Meaning:**

- **NO_REMOTE_PATH:** A remote path was not specified for the file.
  - **Meaning:** The remote path was not passed when uploading. The server gets the parameter name as the remote path of the file: `/remote-path=@local-file.txt`.

- **INVALID_CONTENT:** The parameter name is not a proper data type.
  - **Meaning:** The server does not accept any parameter name type other than binary or string.

- **EMPTY_STREAM:** An empty file stream was passed to the server.
  - **Meaning:** The server attempted to read from an empty file stream, so it rejects the file.

- **NO_DIR_EXISTS:** The directory path does not exist.
  - **Meaning:** Within the parameter name, you can put the directory name in the syntax of a Unix path. If the directory path does not exist, then it will not continue writing: `/no-such-dir/remote-path`.

- **NO_FILE_EXISTS:** The file does not exist on the server.
  - **Meaning:** The server could not find a file with the same name exists on the server, so it will not write. The `/upload` endpoint writes to new files to the server.
