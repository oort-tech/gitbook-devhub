# Authentication & Security



Datahub uses a double-direction API Key validation mechanism:
* **Datahub → Plugin:** Validates orders, creates tasks, manages tasks.
* **Plugin → Datahub:** Uploads/Downloads files, reports status.

## Signature Verification
Plugin developers obtain an `API KEY` and `API SECRET KEY` from the plugin management page.

### 1. Algorithm
* **Algorithm:** `HMAC-SHA256(secret, body + timestamp)`.
* **Content to Sign:**
    * **With Query Params:** `query_string (JSON sorted) + body_bytes + timestamp`.
    * **Without Query Params:** `body_bytes + timestamp`.
* **Query Param Handling:** Sort by key name and convert to a compact JSON string (no spaces).

### 2. Request Headers
All requests must include the following headers:
* `D-API-KEY`: Plugin API Key.
* `D-TIMESTAMP`: Unix timestamp (string).
* `D-SIGNATURE`: HMAC-SHA256 signature (Hex string).

### 3. Timestamp Validation
The request timestamp must be within **300 seconds (5 minutes)** of the server time, otherwise, it will be rejected.

## Python Implementation Example
```python
import hmac
import hashlib
import time
import json
from typing import Optional, Dict, Any

def generate_signature(api_secret: str, body_bytes: bytes, timestamp: str,
                       query_params: Optional[Dict[str, Any]] = None) -> str:
    """Generate HMAC-SHA256 Signature"""
    
    if query_params:
        # Sort params and convert to compact JSON
        sorted_params = dict(sorted(query_params.items()))
        query_string = json.dumps(sorted_params, separators=(',', ':'))
        message = query_string.encode('utf-8') + body_bytes + timestamp.encode('utf-8')
    else:
        message = body_bytes + timestamp.encode('utf-8')

    signature = hmac.new(
        api_secret.encode('utf-8'),
        message,
        hashlib.sha256
    ).hexdigest()
    
    return signature

# Usage Example
api_secret = "your_api_secret"
body = {"subtask_id": "subtask_001", "config": {"keyword": "test"}}
body_bytes = json.dumps(body).encode('utf-8')
timestamp = str(int(time.time()))

signature = generate_signature(api_secret, body_bytes, timestamp)
```