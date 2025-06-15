
---

# n8n Email Automation Workflow

## Overview
This project is an n8n workflow designed to automate the processing of Gmail emails. It collects emails daily, analyzes their content using the Grok AI model, categorizes them, checks for attachments, and stores the results in a Notion database.

## Features
- **Daily Email Collection**: Automatically fetches emails using a scheduled trigger.
- **Content Analysis**: Uses Grok (xAI) to classify emails into categories (e.g., High Priority, Meeting Related, etc.).
- **Attachment Detection**: Checks for email attachments and records their presence.
- **Notion Integration**: Stores email details (subject, date, category, attachment status) in a Notion database.

## Prerequisites
- **n8n**: Version 1.98.1 or later (self-hosted).
- **Gmail API**: Configured with OAuth 2.0 credentials.
- **xAI API**: Access to Grok with a valid API key.
- **Notion API**: Set up with an Internal Integration Secret.
- **Node.js**: For self-hosted n8n environment.

## Installation

### 1. Set Up n8n
- Install n8n if not already done:
  ```bash
  npm install n8n -g
  ```
- Start n8n:
  ```bash
  n8n
  ```

### 2. Configure APIs
- **Gmail**:
  - Create a project in [Google Cloud Console](https://console.developers.google.com/).
  - Enable Gmail API and set up OAuth 2.0 credentials.
  - Note the Client ID and Client Secret.
- **xAI (Grok)**:
  - Sign up at [x.ai/api](https://x.ai/api) to obtain an API key.
- **Notion**:
  - Create a new integration at [Notion My Integrations](https://www.notion.so/my-integrations).
  - Copy the Internal Integration Secret.

### 3. Install Additional Nodes
- If the Grok node is not available, install the community node:
  ```bash
  git clone https://github.com/jvkassi/n8n-nodes-grok
  ```
- Place it in the `nodes` directory of your n8n installation and restart.

## Workflow Setup

### 1. Nodes and Configuration
- **Schedule Trigger**:
  - Set to run daily at 9:00 AM HKT.
- **Gmail Trigger**:
  - Configure with OAuth credentials to fetch new emails.
- **Basic LLM Chain (Grok)**:
  - Set prompt to analyze email content and return a category:
    ```
    請根據以下郵件內容，生成一個分類標籤（選項：高優先級、會議相關、工作、其他、廣告、個人、財務相關、項目更新、社交媒體、垃圾郵件）。郵件主旨：{{ $json.subject }}，郵件內容：{{ $json.text }}。請根據關鍵詞匹配分類，例如“緊急”對應“高優先級”，“會議”對應“會議相關”，其他情況選擇“其他”。只返回分類標籤，例如"高優先級"。
    ```
  - Use xAI API key as credential.
- **Function**:
  - Check for attachments:
    ```
    try {
      const payload = $json.payload || {};
      const parts = payload.parts || [];
      const hasAttachment = parts.some(part => part && part.filename);
      return {
        hasAttachment: hasAttachment
      };
    } catch (e) {
      return {
        hasAttachment: false
      };
    }
    ```
- **Date & Time**:
  - Convert `$json.date` to ISO 8601 format (e.g., `2025-06-16T00:29:00+08:00`).
- **Notion**:
  - Map fields:
    - `Title`: `$json.subject`
    - `日期.date.start`: `$node["Date & Time"].json["formatted"]`
    - `分類`: `$node["Basic LLM Chain"].json["category"]`
    - `附件`: `$json.hasAttachment`

### 2. Test the Workflow
- Send test emails with varied content (e.g., "Urgent Meeting", "20% Discount").
- Execute the workflow and verify Notion updates.

## Usage
- The workflow runs daily at 9:00 AM HKT, collecting and processing emails.
- Categories and attachment status are stored in the Notion database for review.

## Troubleshooting
- **Invalid Date Error**: Ensure the Date & Time node correctly converts `$json.date` to ISO 8601 format.
- **Grok Returns "Other"**: Check if `$json.subject` and `$json.text` contain data; optimize the prompt with keyword rules.
- **Attachment Detection Fails**: Verify `payload.parts` or use `$json.attachments` if available.

## Contributing
Feel free to fork this repository, submit issues, or pull requests to improve the workflow.


