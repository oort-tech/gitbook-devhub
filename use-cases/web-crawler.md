# Use Case: Web Crawler Plugin

## Overview
**Category**: Data Collection

In the AI Data Supply Chain, raw data collection is the first step. A Web Crawler Plugin is a perfect example of a tool that developers can build on OORT Datahub to source high-quality data from the internet.

## The Goal
Build a plugin that accepts a list of URLs and specific extraction rules (e.g., "extract all images" or "get article text"), and returns the structured data to OORT Datahub.

## How it Works

1.  **Task Assignment**: OORT Datahub calls your plugin's `POST /api/plugin/create_task` endpoint with a payload containing the target URLs.
2.  **Execution**: Your plugin spins up its crawling workers to fetch the pages.
3.  **Result Upload**: Once crawled, your plugin packages the data (e.g., JSON or ZIP of images) and uses the Datahub API `POST /api/task/upload/init` to upload the results.
4.  **Completion**: You notify Datahub that the task is finished via `POST /api/task/result`.

## Example Payload (`create_task`)

```json
{
  "task_id": "crawl_Job_123",
  "task_config": {
    "target_url": "https://example.com/news",
    "depth": 2,
    "extract": "article_body"
  }
}
```

## Developer Value
By building a robust crawler, you can monetize your tool by charging a fee per MB of data collected or per URL processed, directly through the OORT platform.
