# Appendix

## Error Codes
The following standard error codes are used within the system to indicate the status of API requests.

| Code | Description |
| :--- | :--- |
| **1** | **Business Error:** Indicates a logical error within the business processing flow. |
| **400** | **Request Error:** The request parameters or format are invalid. |
| **401** | **Authentication Failed:** The API Key signature verification failed or the key is invalid. |
| **500** | **Internal Server Error:** An unexpected error occurred on the server side. |

## Security Specifications
To ensure the security and integrity of the Datahub ecosystem, all plugins must adhere to the following specifications:

* **HTTPS:** All API communication must be encrypted using HTTPS.
* **Secret Management:** API Keys and Secrets must be kept confidential and never exposed in client-side code.
* **Privacy:** Plugins must not leak or misuse user privacy data.
* **Hosting:** Plugin JS files should be hosted on a CDN or OORT Storage for performance and security.

## FAQ

**Q: How do I get existing config values when editing a task?**
A: Currently, the plugin JS does not receive the previously saved configuration during initialization. The plugin is initialized fresh each time. Future versions of the protocol may support passing existing values.

**Q: Can I use third-party UI frameworks (like React, Vue, or Bootstrap)?**
A: Yes, you can use other frameworks, but with strict caveats:
1.  **CSS:** Do not rely on external CSS files unless you explicitly load them within your JS or ensure they are available.
2.  **Style:** The UI should aesthetically match Datahub's design (Tailwind CSS is recommended).
3.  **Responsiveness:** The UI must remain responsive.

**Q: Is there a size limit for the configuration data?**
A: Yes, since the configuration is stored in the database, it is recommended to keep the JSON data size under 10KB.

**Q: Can I load resources asynchronously?**
A: Yes, but you must ensure that all necessary resources are loaded and the user has input data before `updateConfig` is called to avoid submitting incomplete configuration data.

```