# Datahub Endpoints

These are APIs provided by **Datahub** for plugins to call.
**Base URL**: `https://datahub.oortech.com`

## 1. File Upload
**Principle**: Datahub only provides Presigned URLs. You upload file data directly to S3/OSS.

### 1.1 Initialize Upload
**POST** `/api/task/upload/init`

Supports single file or multipart (chunked) upload.

#### Request
```json
{
	"subtask_id": "task_abc_001",
	"file_name": "data.csv",
	"file_size": 10485760,
	"file_md5": "md5hash",
	"chunk_size": 5242880  // Optional. If set, triggers multipart.
}
```

#### Response
```json
{
    "code": 0,
    "data": {
        "upload_id": "upload_123",
        "part_upload_urls": [
            { "index": 1, "url": "https://s3...part1" },
            { "index": 2, "url": "https://s3...part2" }
        ]
    }
}
```

### 1.2 Perform Upload (Direct to Storage)
For each part, sending a `PUT` request to the `url`.
*   **Response Header**: Capture the `ETag`. You need this for completion.

### 1.3 Complete Upload
**POST** `/api/task/upload/complete`

#### Request
```json
{
    "upload_id": "upload_123",
    "subtask_id": "task_abc_001",
    "file_name": "data.csv",
    "parts": [
        { "index": 1, "etag": "\"etag1\"" },
        { "index": 2, "etag": "\"etag2\"" }
    ]
}
```

---

## 2. File Download
**GET** `/api/task/download/presign`

Get a presigned URL to download input files. Supports HTTP Range requests for resumable downloads.

#### Request (Query)
`subtask_id=task_abc_001`

#### Response
```json
{
  "code": 0,
  "data": {
    "download_url": "https://s3...",
    "file_name": "input.zip"
  }
}
```

---

## 3. Status & Result

### 3.1 Update Status
**POST** `/api/task/update_status`

Report state changes (e.g., Started, Paused, Failed).

#### Request
```json
{
    "subtask_id": "task_abc_001",
    "status": 1  // 1: Running, 2: Paused, 4: Cancelled, 5: Failed
}
```

### 3.2 Heartbeat
**POST** `/api/task/heartbeat`

Send every ~30 seconds to keep task alive.

#### Request
```json
{
    "subtask_id": "task_abc_001",
    "timestamp": 1704067200,
    "process": 50
}
```

### 3.3 Submit Result
**POST** `/api/task/result`

Final submission when task is complete.

#### Request
```json
{
    "subtask_id": "task_abc_001",
    "result": {
         "total_counts": 100,
         "success_counts": 100
    }
}
```
