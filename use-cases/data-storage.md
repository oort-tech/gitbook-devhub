# Use Case: Data Storage

## Overview
**Category**: Data Storage & Infrastructure

In the decentralized Data Supply Chain, data ownership is key. Users and developers often need to store their collected or processed data in specific locations for compliance, cost, or performance reasons.

## The Goal
Enable developers to build **Storage Plugins** or configure existing plugins to route data to custom storage backends (BYOS - Bring Your Own Storage).

## How it Works

OORT Datahub is designed to be storage-agnostic. While it provides default storage, specific **Storage Plugins** can be developed to bridge Datahub with external providers.

### Scenario: Custom S3 Backend
A developer creates an "AWS S3 Connector" plugin.
1.  **Configuration**: The user configures the plugin with their `Bucket Name`, `Region`, and `IAM Credentials` via the plugin's frontend.
2.  **Integration**: When a Data Collection task runs, instead of uploading to OORT's default bucket, the system uses the Storage Plugin to generate presigned URLs for the user's private bucket.
3.  **Direct Upload**: The collection worker uploads data directly to the user's AWS S3, ensuring Datahub never holds the raw data, only the metadata.

### Scenario: Decentralized Storage (Web3)
A developer builds a "Filecoin Archival" plugin.
1.  **Trigger**: After a dataset is labeled and finalized.
2.  **Action**: The plugin initiates a deal to store the immutable dataset on the Filecoin network.
3.  **Proof**: The `CID` and storage deal ID are returned to Datahub as the "Result", providing a permanent, verifiable record of the data.

## Developer Value
*   **Privacy**: Build solutions for enterprises that require on-premise or private cloud storage.
*   **Resilience**: Leverage decentralized networks (IPFS, Arweave) for censorship-resistant data hosting.
