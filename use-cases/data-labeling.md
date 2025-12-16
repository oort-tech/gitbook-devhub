# Use Case: Data Labeling

## Overview
**Category**: Data Processing

Raw data often needs human context before it is useful for AI. Data Labeling plugins bridge the gap between raw files and training-ready datasets.

## The Goal
Create a workflow where raw data (e.g., unlabelled images) is distributed to real users (via OORT Launchpad or your own workforce) for annotation.

## How it Works

1.  **Data Ingestion**: A user (or another plugin) uploads a dataset to OORT Datahub.
2.  **Task Creation**: Datahub calls your plugin with the location of the dataset.
3.  **Distribution**: Your plugin effectively "shards" the dataset into small micro-tasks.
    *   *Integration Note*: You can integrate with OORT Launchpad to show these micro-tasks to the OORT community.
4.  **Labeling Interface**: You provide a UI (embedded or external) where users draw bounding boxes, classify text, or transcribe audio.
5.  **Aggregation**: Your plugin collects the answers, possibly implementing a "consensus" mechanism (e.g., 3 users must agree).
6.  **Final Output**: The labeled dataset (e.g., COCO format JSON) is uploaded back to Datahub.

## Impact
Your labeling plugin becomes a critical node in the supply chain, turning "data" into "knowledge". High-quality labeling tools are in high demand for computer vision and LLM fine-tuning.
