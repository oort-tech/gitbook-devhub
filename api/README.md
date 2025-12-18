# Introduction & Overview

Datahub supports third-party development plugins, enabling functions such as data collection, storage, and verification. All plugins follow a unified operational standard.

## Core Components
Developing a plugin requires two main components:
1.  **Frontend JS File:** Used to display the plugin parameter configuration page.
2.  **Plugin Backend API:** Used to be called by Datahub to execute tasks and upload results.

## Plugin Types
Datahub currently supports the following plugin types, with support for future extensions:

| Type | Description | Examples |
| :--- | :--- | :--- |
| **Ingest** | Collect data from external systems | Crawlers, API collection, IoT data |
| **Storage** | Upload data to external storage | AWS S3, OSS |
| **Verification** | Validate data quality and integrity | Duplicate detection, AI image recognition |
| **Notification** | Task completion notifications | Email, SMS, Slack messages |
| **Security** | Encryption, masking, auditing | Key management, sensitive data masking |
| **Labeling** | Data annotation | Image labeling, text classification |
| **Transformation** | Format conversion, structure processing | JSON to CSV, image compression |