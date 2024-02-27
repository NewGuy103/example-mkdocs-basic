# API Overview

This page will define how to call the REST APIs provided in `syncserver.server`.

## Base URL

The base URL for all API endpoints is:

```
http://localhost:$SYNCSERVER_PORT
```

## Authentication

All API requests require authentication. Include the following headers in your request:

- **syncServer-Username**: YOUR-USERNAME
- **syncServer-Token**: YOUR-PASSWORD

Replace `YOUR-USERNAME` and `YOUR-PASSWORD` with a valid username and password.

## Endpoints

### 1. Upload Endpoint

**Description:**

This endpoint is where you upload new files to the server. This can handle more than one file, and will return depending if only one file or more than one file is passed to the server.

**Endpoint:**

```
POST /upload
```

**Parameters:**

- `files`: List of files passed to upload to the server.
 
**Example Request:**

```python
import requests

url = "http://localhost:$SYNCSERVER_PORT/upload"
headers = {
    "syncServer-Username": "YOUR-USERNAME",
    "syncServer-Token": "YOUR-PASSWORD"
}

files = {
    "/remote-path": ("/remote-path", open("path/to/filename.txt", "rb"))
}

response = requests.post(url, headers=headers, files=files)
```

**Error Codes and Meaning:**

- **MISSING_FILES:** No files were provided in the request for upload.
  - **Meaning:** The server did not receive any files, please check if the file data is being properly passed.

- **NO_REMOTE_PATH:** A remote path was not specified for the file.
  - **Meaning:** The remote path was not passed when uploading. The server gets the parameter name as the remote path of the file: `/remote-path=@local-file.txt`.

- **INVALID_CONTENT:** The parameter name is not a proper data type.
  - **Meaning:** The server does not accept any parameter name type other than binary or string.

- **EMPTY_STREAM:** An empty file stream was passed to the server.
  - **Meaning:** The server attempted to read from an empty file stream, so it rejects the file.

- **NO_DIR_EXISTS:** The directory path does not exist.
  - **Meaning:** Within the parameter name, you can put the directory name in the syntax of a Unix path. If the directory path does not exist, then it will not continue writing: `/no-such-dir/remote-path`.

- **FILE_EXISTS:** The file already exists on the server.
  - **Meaning:** The server has detected a file with the same name exists on the server, so it will not write to prevent data being overwritten. The `/modify` endpoint writes to existing files on the server.

**Example Responses:**

- **Non-Batch Successful Upload:**

```json
{
    "batch": false,
    "success": true
}
```

- **Non-Batch Failed Upload:**
```json
{
    "/remote-path": {
        "ecode": "FILE_EXISTS",
        "error": "Target file path already exists. Use /modify to edit a file or check the filename."
    }
}
```

- **Batch Uploads:**
```json
{
    "batch": true,
    "fail": {
        "/remote-path-1": {
            "ecode": "FILE_EXISTS",
            "error": "Target file path already exists. Use /modify to edit a file or check the filename."
        }
    },
    "ok": [
        "/remote-path-2"
    ]
}
```

### 2. Modify Endpoint

**Description:**

This endpoint is where you modify existing files by uploading them to the server. This can handle more than one file, and will return depending if only one file or more than one file is passed to the server.

**Endpoint:**

```
POST /modify
```

**Parameters:**

- `files`: List of files passed to upload to the server.
 
**Example Request:**

```python
import requests

url = "http://localhost:$SYNCSERVER_PORT/modify"
headers = {
    "syncServer-Username": "YOUR-USERNAME",
    "syncServer-Token": "YOUR-PASSWORD"
}

files = {
    "/remote-path": ("/remote-path", open("path/to/filename.txt", "rb"))
}

response = requests.post(url, headers=headers, files=files)
```

**Error Codes and Meaning:**

- **MISSING_FILES:** No files were provided in the request for upload.
  - **Meaning:** The server did not receive any files, please check if the file data is being properly passed.

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

**Example Responses:**

- **Non-Batch Successful Upload:**

```json
{
    "batch": false,
    "success": true
}
```

- **Non-Batch Failed Upload:**
```json
{
    "/remote-path": {
        "ecode": "NO_FILE_EXISTS",
        "error": "Target file path does not exist. Use /upload to create a file or check the filename."
    }
}
```

- **Batch Uploads:**
```json
{
    "batch": true,
    "fail": {
        "/remote-path-1": {
            "ecode": "NO_FILE_EXISTS",
            "error": "Target file path does not exist. Use /upload to create a file or check the filename."
        }
    },
    "ok": [
        "/remote-path-2"
    ]
}
```

### 3. Delete Endpoint

**Description:**

This endpoint is where you delete existing files from the server. This will remove the file from the directory and from the database.

**Endpoint:**

```
POST /delete
```

**Parameters:**

- `file-paths`: JSON List of remote files to delete from the server.

**Example Request:**

```python
import requests
import json

url = "http://localhost:$SYNCSERVER_PORT/delete"
headers = {
    "syncServer-Username": "YOUR-USERNAME",
    "syncServer-Token": "YOUR-PASSWORD"
}

data = {
    "file-paths": json.dumps(['/remote-path'])
}

response = requests.post(url, headers=headers, data=data)
```

**Error Codes and Meaning:**

- **MISSING_FILEPATHS:** No file paths were provided in the request for upload.
  - **Meaning:** The server did not receive the JSON list, please check if the parameter is `file-paths`.

- **INVALID_JSON:** The JSON data is malformed or invalid.
  - **Meaning:** The server could not properly parse the JSON list.

- **NO_DIR_EXISTS:** The directory path does not exist.
  - **Meaning:** Within the parameter name, you can put the directory name in the syntax of a Unix path. If the directory path does not exist, then it will not continue writing: `/no-such-dir/remote-path`.

- **NO_FILE_EXISTS:** The file does not exist on the server.
  - **Meaning:** The server could not find a file with the same name exists on the server, so it will not write. The `/upload` endpoint writes to new files to the server.

- **EMPTY_PATHLIST:** The file path list is empty.
  - **Meaning:** This means that the provided list has no items. This can happen when passing the wrong data type into the list, and the server ends up filtering it, making the list have zero items.

**Example Responses:**

- **Non-Batch Successful Upload:**

```json
{
    "batch": false,
    "success": true
}
```

- **Non-Batch Failed Upload:**
```json
{
    "/remote-path": {
        "ecode": "NO_FILE_EXISTS",
        "error": "Target file path does not exist. Use /upload to create a file or check the filename."
    }
}
```

or:

```json
{
    "error": "filenames path is empty",
    "ecode": "EMPTY_PATHLIST"
}
```

- **Batch Uploads:**
```json
{
    "batch": true,
    "fail": {
        "/remote-path-1": {
            "ecode": "NO_FILE_EXISTS",
            "error": "Target file path does not exist. Use /upload to create a file or check the filename."
        }
    },
    "ok": [
        "/remote-path-2"
    ]
}
```

### 4. Read Endpoint

**Description:**

This endpoint is where you read existing files on the server. This returns a plain content stream, and can only take one file path at a time.

**Endpoint:**

```
POST /read
```

**Parameters:**

- `file-path`: Remote file path to read.
 
**Example Request:**

```python
import requests

url = "http://localhost:$SYNCSERVER_PORT/read"
headers = {
    "syncServer-Username": "YOUR-USERNAME",
    "syncServer-Token": "YOUR-PASSWORD"
}
data = {
    "file-path": "/remote-path"
}

response = requests.post(url, headers=headers, data=data)
content = response.content
```

**Error Codes and Meaning:**

- **MISSING_FILEPATH:** No file path provided to read from.
  - **Meaning:** The server did not get the file path parameter. Please check if the parameter is `file-path`.

- **INVALID_CONTENT:** The parameter name is not a proper data type.
  - **Meaning:** The server does not accept any parameter name type other than binary or string.

- **NO_DIR_EXISTS:** The directory path does not exist.
  - **Meaning:** Within the parameter name, you can put the directory name in the syntax of a Unix path. If the directory path does not exist, then it will not continue writing: `/no-such-dir/remote-path`.

- **NO_FILE_EXISTS:** The file does not exist on the server.
  - **Meaning:** The server could not find a file with the same name exists on the server, so it will not write. The `/upload` endpoint writes to new files to the server.

**Example Responses:**

- **Non-Batch Failed Upload:**
```json
{
    "ecode": "FILE_EXISTS",
    "error": "Target file path already exists. Use /modify to edit a file or check the filename."
}
```

### 5. Create Directory Endpoint

**Description:**

This endpoint is where you create a directory on the server to store files. This directory is not a traditional file system, but an emulated one using databases.

**Endpoint:**

```
POST /create-dir
```

**Parameters:**

- `dir-path`: Directory path to create.
 
**Example Request:**

```python
import requests

url = "http://localhost:$SYNCSERVER_PORT/create-dir"
headers = {
    "syncServer-Username": "YOUR-USERNAME",
    "syncServer-Token": "YOUR-PASSWORD"
}
data = {
    "dir-path": "/remote-dir"
}

response = requests.post(url, headers=headers, data=data)
```

**Error Codes and Meaning:**

- **MISSING_DIRPATH:** No directory path provided to create.
  - **Meaning:** The server did not get the dir path parameter. Please check if the parameter is `dir-path`.

- **INVALID_CONTENT:** The directory name is not a proper data type.
  - **Meaning:** The server does not accept any directory name type other than binary or string.

- **DIR_EXISTS:** The directory path already exists.
  - **Meaning:** The directory was found and already exists, so the server does not continue writing. `/remove-dir` can be used to remove directories and all the files within it.

- **MISSING_PATH:** The target path is missing.
  - **Meaning:** This means that no directory path was actually provided. (i.e an empty string.)

- **INVALID_DIR_PATH:** The directory path is malformed or invalid.
  - **Meaning:** The directory path could have characters before the first forward slash (like `chars/dir1`) and is treated as malformed.


**Example Responses:**

- **Successful Request:**
```json
{"success": true}
```

- **Failed Request:**
```json
{
    "error": "Directory path is malformed or invalid",
    "ecode": "INVALID_DIR_PATH"
}
```

### 6. Remove Directory Endpoint

**Description:**

This endpoint is where you delete a directory and the files within it.

**Endpoint:**

```
POST /remove-dir
```

**Parameters:**

- `dir-path`: Directory path to delete.
 
**Example Request:**

```python
import requests

url = "http://localhost:$SYNCSERVER_PORT/remove-dir"
headers = {
    "syncServer-Username": "YOUR-USERNAME",
    "syncServer-Token": "YOUR-PASSWORD"
}
data = {
    "dir-path": "/remote-dir"
}

response = requests.post(url, headers=headers, data=data)
```

**Error Codes and Meaning:**

- **MISSING_DIRPATH:** No directory path provided to create.
  - **Meaning:** The server did not get the dir path parameter. Please check if the parameter is `dir-path`.

- **INVALID_CONTENT:** The directory name is not a proper data type.
  - **Meaning:** The server does not accept any directory name type other than binary or string.

- **NO_DIR_EXISTS:** The directory path does not exist.
  - **Meaning:** The directory wasn't found, so the server does not continue writing. `/create-dir` can be used to create directories.

- **ROOT_DIR:** Attempted to delete the root directory.
  - **Meaning:** This means that you attempted to set `dir-path` to `/`, which is a safeguard to prevent recursively deleting everything that your user account owns.

- **INVALID_DIR_PATH:** The directory path is malformed or invalid.
  - **Meaning:** The directory path could have characters before the first forward slash (like `chars/dir1`) and is treated as malformed.


**Example Responses:**

- **Successful Request:**
```json
{"success": true}
```

- **Failed Request:**
```json
{
    "error": "Cannot delete root directory",
    "ecode": "ROOT_DIR"
}
```

### 7. List Directory Endpoint

**Description:**

This endpoint is where you list an existing directory on the server. This will show all the filenames that are in that directory.

**Endpoint:**

```
POST /list-dir
```

**Parameters:**

- `dir-path`: Directory path to list.
 
**Example Request:**

```python
import requests

url = "http://localhost:$SYNCSERVER_PORT/list-dir"
headers = {
    "syncServer-Username": "YOUR-USERNAME",
    "syncServer-Token": "YOUR-PASSWORD"
}
data = {
    "dir-path": "/remote-dir"
}

response = requests.post(url, headers=headers, data=data)
json_path = response.content
```

**Error Codes and Meaning:**

- **MISSING_DIRPATH:** No directory path provided to create.
  - **Meaning:** The server did not get the dir path parameter. Please check if the parameter is `dir-path`.

- **INVALID_CONTENT:** The directory name is not a proper data type.
  - **Meaning:** The server does not accept any directory name type other than binary or string.

- **DIR_EXISTS:** The directory path already exists.
  - **Meaning:** The directory was found and already exists, so the server does not continue writing. `/remove-dir` can be used to remove directories and all the files within it.

- **INVALID_DIR_PATH:** The directory path is malformed or invalid.
  - **Meaning:** The directory path could have characters before the first forward slash (like `chars/dir1`) and is treated as malformed.


**Example Responses:**

- **Successful Request:**
```json
{
    "success": true
    "dir-listing": "['file1', 'file2']"
}
```

- **Failed Request:**
```json
{
    "error": "Directory path is malformed or invalid",
    "ecode": "INVALID_DIR_PATH"
}
```
