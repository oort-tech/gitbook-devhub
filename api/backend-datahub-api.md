# Backend Development: Datahub APIs

Plugins communicate with Datahub to manage file transfers and report progress.

> **Core Principle:** All file I/O connects directly to third-party object storage (S3/OSS/COS). Datahub only provides presigned URLs and does not proxy the data stream.

## 1. File Upload (Multipart)

### Step 1: Initialize Upload
This interface initializes the upload process, supporting both single file and multipart uploads.

* **URL:** `/api/task/upload/init`
* **Method:** `POST`

**Request:**
```json
{
  "subtask_id": "task_abc_001_ingest",
  "file_name": "data.csv",
  "file_size": 10485760,
  "chunk_size": 5242880
}

```

* `subtask_id`: Subtask ID (Required).
* `filename`: File name (Required).
* `filesize`: File size in bytes (Required).
* `chunk_size`: Optional. If specified, enables multipart upload. If `null`, it defaults to single file upload. Minimum chunk size is 5MB if multipart is used.

**Response:** Returns `upload_id` and a list of `part_upload_urls` (presigned URLs).

### Step 2: Direct Upload

The plugin `PUT`s data directly to the `part_upload_urls`. The response header contains an `ETag`, which must be saved by the plugin.

### Step 3: Complete Upload

This interface notifies Datahub that the file upload is complete or merges the parts.

* **URL:** `/api/task/upload/complete`
* **Method:** `POST`

**Request:**

```json
{
  "upload_id": "upload_123456",
  "subtask_id": "task_abc_001_ingest",
  "parts": [
    { "index": 1, "etag": "\"etag_value_1\"" },
    { "index": 2, "etag": "\"etag_value_2\"" }
  ]
}

```

* `upload_id`: Upload ID obtained from initialization (Required).
* `parts`: Array of part information, including `index` and `etag` (Required).

## 2. File Download

All downloads utilize Datahub presigned URLs.

* **URL:** `/api/task/download/presign`
* **Method:** `GET`
* **Query Param:** `subtask_id` (Required).
* **Response:** Returns `download_url`.
* **Note:** Supports Range requests for resuming downloads.

## 3. Update Status

When the task status changes, the plugin must actively call this interface to report it.

* **URL:** `/api/task/update_status`
* **Method:** `POST`

**Request:**

```json
{
  "subtask_id": "task_abc_001_ingest",
  "status": 1
}

```

* `status`: Task status code.
* `1`: Running
* `2`: Paused
* `4`: Cancelled
* `5`: Failed
* *Note:* Status `3` (Completed) must be set via the `/api/task/result` interface.



## 4. Heartbeat

Plugins can actively report task progress to maintain a heartbeat.

* **URL:** `/api/task/heartbeat`
* **Method:** `POST`

**Request:**

```json
{
  "subtask_id": "task_abc_001_ingest",
  "timestamp": 1704067200,
  "process": 50
}

```

* `timestamp`: Unix timestamp in seconds (Required).
* `process`: Task progress percentage (0-100) (Required).

## 5. Submit Result

Plugins use this to report the final task results.

* **URL:** `/api/task/result`
* **Method:** `POST`

**Request:**

```json
{
  "subtask_id": "task_abc_001_ingest",
  "result": {
    "total_counts": 20,
    "success_counts": 20,
    "failed_counts": 0
  }
}

```

* `result`: Task result data JSON object (Required).

## 6. Query Task Status

Plugins can actively query the current status of a task.

* **URL:** `/api/task/notify`
* **Method:** `GET`
* **Query Param:** `task_id`.

**Response:**

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "status": 0
  }
}

```

```

```