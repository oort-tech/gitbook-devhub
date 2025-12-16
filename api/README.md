# API Reference

Welcome to the OORT Datahub API documentation. This documentation covers the communication protocol between OORT Datahub and third-party plugins.

## Authentication

Security is paramount. All requests between Datahub and plugins must be authenticated using HMAC-SHA256 signatures.

### API Keys
Plugins are assigned an **API Key** and **API Secret** upon registration in the Datahub management portal.
- **API Key**: Public identifier for the plugin.
- **API Secret**: Kept private, used to sign requests.

### Headers
Every request must include the following headers:

| Header | Description |
| :--- | :--- |
| `D-API-KEY` | The Plugin API Key. |
| `D-TIMESTAMP` | Current Unix timestamp (seconds). Valid for 5 minutes (300s). |
| `D-SIGNATURE` | The HMAC-SHA256 signature of the request. |
| `Content-Type` | `application/json` |

### Signature Algorithm
The signature is generated using HMAC-SHA256.

**Formula:**
`HMAC-SHA256(secret, message)`

**Message Construction:**
1.  **With Query Params**: `sorted_json_string(query_params) + body_bytes + timestamp`
2.  **Without Query Params**: `body_bytes + timestamp`

> **Note**: JSON stringification must use separators `(',', ':')` to remove whitespace.

### Python Example

```python
import hmac
import hashlib
import time
import json
import requests

def generate_signature(api_secret: str, body_bytes: bytes, timestamp: str, query_params: dict = None) -> str:
    if query_params:
        # Sort keys and compact JSON
        sorted_params = dict(sorted(query_params.items()))
        query_string = json.dumps(sorted_params, separators=(',', ':'))
        message = query_string.encode('utf-8') + body_bytes + timestamp.encode('utf-8')
    else:
        message = body_bytes + timestamp.encode('utf-8')
    
    return hmac.new(
        api_secret.encode('utf-8'),
        message,
        hashlib.sha256
    ).hexdigest()

# Example Usage
api_key = "your_api_key"
api_secret = "your_api_secret"
timestamp = str(int(time.time()))
url = "http://datahub.example.com/api/plugin/create_task"
body = {"subtask_id": "subtask_001", "config": {"keyword": "test"}}
body_bytes = json.dumps(body).encode('utf-8')

signature = generate_signature(api_secret, body_bytes, timestamp)

headers = {
    "Content-Type": "application/json",
    "D-API-KEY": api_key,
    "D-TIMESTAMP": timestamp,
    "D-SIGNATURE": signature
}

response = requests.post(url, json=body, headers=headers)
```

## Error Codes

| Error Code | Description |
| :--- | :--- |
| `0` | Success |
| `1` | Business Error |
| `400` | Bad Request |
| `401` | Authentication Failed |
| `500` | Internal Server Error |

## Security Guidelines

1.  **HTTPS Only**: All API communication must be encrypted via HTTPS.
2.  **Secret Secrecy**: Never expose your API Secret in client-side code.
3.  **Privacy**: Do not leak user privacy data.
