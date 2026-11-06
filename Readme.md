# 🚀 AI Job Discovery & Application Assistant (n8n + Ollama Workflow)

An automated end-to-end AI agent built in **n8n** that accepts job search queries via a form, fetches live remote job postings, deduplicates against an existing Google Sheet database, evaluates candidates using a local **Ollama LLM (Llama 3)** instance, sends email notifications, and logs matched jobs into a Google Sheet tracker.

---

## 📌 Features

- **Form Trigger:** User-friendly input form for specifying job target titles, preferred location, and match threshold.
- **Data Normalization & Sanitization:** Filters and strips HTML tags from raw job feeds.
- **Smart Deduplication:** Cross-checks new findings against Google Sheets records to prevent processing duplicate jobs.
- **Privacy-First AI Evaluation:** Runs local LLMs via **Ollama** (e.g., `llama3`) for resume-to-job matching without relying on paid third-party APIs.
- **Automated Alerts & Digest:** Sends real-time Gmail notifications summarizing short-listed jobs and alerts when no new listings are found.
- **Google Sheets Tracker Integration:** Saves structured match data (`Role`, `Company`, `Location`, `MatchScore`, `Status`, `Reasoning`) directly into Google Sheets.

---

## 🏗️ Workflow Architecture Breakdown

```
[Job Search Form]
       │
       ▼
[Parse & Validate Input] ──(Invalid)──► [Stop - Invalid Input]
       │
    (Valid)
       ▼
[Build Search Request] ──► [Fetch Jobs (API)] ──► [Normalize Jobs]
                                                      │
                                                      ▼
                                            [Dedupe & Hard Filters]
                                                      │
                                                      ▼
                                              [Read Existing Jobs]
                                                      │
                                                      ▼
                                           [Remove Already Tracked]
                                                      │
                                      ┌───────────────┴───────────────┐
                                (Has New Jobs)                  (No New Jobs)
                                      │                               │
                                      ▼                               ▼
                              [Loop Over Jobs]               [Notify No Results]
                                      │
            ┌─────────────────────────┴────────────────────────┐
            ▼                                                  ▼
   [Build Review Digest]                               [Build AI Request]
            │                                                  │
            ▼                                                  ▼
   [Send Review Digest]                              [AI Match & Score (Ollama)]
                                                               │
                                                               ▼
                                                     [Validate AI Output]
                                                               │
                              ┌────────────────────────────────┴────────────────────────────────┐
                          (Valid)                                                           (Invalid)
                              │                                                                 │
                              ▼                                                                 ▼
                   [Meets Minimum Score?]                                               [Mark AI Error]
                      │               │                                                         │
             (Score ≥ 70)           (Score < 70)                                                 │
                  │                   │                                                         │
                  ▼                   ▼                                                         │
          [Mark Shortlisted]  [Mark Below Threshold]                                            │
                  │                   │                                                         │
                  └───────────────────┼─────────────────────────────────────────────────────────┘
                                      │
                                      ▼
                            [Prepare Tracker Row]
                                      │
                                      ▼
                            [Save to Tracker (Google Sheets)]
```

---

## 📋 Prerequisites

1. **n8n Instance:** Self-hosted (Docker/npm) or n8n Cloud.
2. **Ollama Installed Locally:** 
   - Installed on local machine with the `llama3` model downloaded (`ollama run llama3`).
   - Default host endpoint: `http://localhost:11434` (or `http://host.docker.internal:11434` if n8n is running inside Docker).
3. **Google Workspace Account:** For Google Sheets integration and Gmail notifications.

---

## ⚙️ Setup & Configuration Instructions

### 1. Configure Local Ollama Endpoint
Ensure Ollama is running and accessible on port `11434`.

```bash
# Pull and start the llama3 model locally
ollama run llama3
```

### 2. Configure Google Sheets
Create a Google Sheet titled **Job Application Tracker** with a tab named `Tracked_Jobs` containing the following header columns:

| ID | RoleTitle | Company | Location | URL | MatchScore | Status | Reasoning |
|---|---|---|---|---|---|---|---|

### 3. Import Workflow into n8n
1. Open your n8n canvas.
2. Click **Menu (Top-Right)** $\rightarrow$ **Import from File**.
3. Select the provided `full_ai_job_discovery_assistant_n8n.json` file.

### 4. Credentials Setup
- **Google Sheets Node:** Connect your Google OAuth2 / Service Account credentials. Replace `YOUR_GOOGLE_SHEET_ID` inside the `Read Existing Jobs` and `Save to Tracker` nodes with your actual Document ID.
- **Gmail Node:** Authorize your Gmail OAuth2 credentials in `Send Review Digest` and `Notify No Results`.

---

## 🧪 Testing the Workflow

1. Click on the **Job Search Form** trigger node in n8n and copy the Form URL.
2. Open the URL, enter a target role (e.g., `Python Developer`), select a minimum score (`70`), and click **Submit**.
3. Observe the step-by-step canvas execution:
   - Fetches and filters jobs.
   - Evaluates match scores via local Ollama.
   - Saves results into Google Sheets and sends email alerts.

---

## 🛠️ Troubleshooting Common Errors

- **`Normalize Jobs` returns 0 items:**
  Ensure the API response handles both direct arrays and object wrappers. Use the provided safe parser snippet in the JavaScript code block.
- **Docker Connection Refused for Ollama:**
  If n8n is running inside Docker, change `localhost:11434` to `host.docker.internal:11434` in the HTTP Request node configuration.