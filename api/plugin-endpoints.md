# Plugin Endpoints

These are the APIs that **your plugin must implement**. Datahub calls these to manage tasks.

## 1. Frontend Configuration (JS)
Datahub dynamically loads your plugin's JS to render the configuration UI.

### Interface
Your JS must return a module with `init` and `validateConfig` methods.

```javascript
return {
  /**
   * Initialize UI
   * @param {string} containerId - DOM ID to render into
   * @param {function} updateConfig - Callback to save config { key: value }
   */
  init: function(containerId, updateConfig) {
      // 1. Render HTML into document.getElementById(containerId)
      // 2. Bind events to inputs
      // 3. Call updateConfig(newConfig) when inputs change
  },

  /**
   * Validate Configuration
   * @param {object} config 
   * @returns {object} { valid: boolean, message: string }
   */
  validateConfig: function(config) {
      if (!config.keyword) return { valid: false, message: "Keyword required" };
      return { valid: true, message: "" };
  }
};
```

---

## 2. Server-side Endpoints

### 2.1 Create Task
**POST** `/api/plugin/create_task`

Initiates a task. The `config` object matches what your Frontend JS generated.

#### Request
```json
{
    "subtask_id": "task_abc_001_ingest",
    "config": {
        "keyword": "cat",
        "maxImages": 20
    }
}
```

#### Response
```json
{ "code": 0, "msg": "success", "data": null }
```

### 2.2 Manage Task
**POST** `/api/plugin/manage_task`

Control task lifecycle (Pause/Resume/Cancel).

#### Request
```json
{
    "subtask_id": "task_abc_001_ingest",
    "status": 1
}
```
*   `status`: `1` (Resume/Running), `2` (Pause), `4` (Cancel).

### 2.3 Health Check
**GET** `/api/plugin/health`

#### Request (Query)
`timestamp=1763382017`

#### Response
```json
{
    "code": 0,
    "msg": "success",
    "status": "healthy",
    "plugin": {
        "uptime": 3600,
        "version": "1.0.0",
        "plugin_id": "plugin-6cb79301"
    }
}
```

### 2.4 Payment Validation (Optional)
**POST** `/api/plugin/payment`

#### Request
```json
{
    "tx_hash": "0x123...",
    "subtask_id": "task_abc_001_ingest"
}
```
