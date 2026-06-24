# GadgetLab Video Pipeline

> **Autonomous TikTok & YouTube video ingestion** — zero human intervention required.

[![CI](https://github.com/labgadget015-dotcom/gadgetlab-video-pipeline/actions/workflows/ci.yml/badge.svg)](https://github.com/labgadget015-dotcom/gadgetlab-video-pipeline/actions/workflows/ci.yml)

## What It Does

Post any TikTok or YouTube URL to `#all-gadget-ai` in Slack. The pipeline automatically:

```
Slack URL post
    ↓
Extract & split URLs (regex)
    ↓
Route: YouTube ↔ TikTok
    ↓           ↓
YT API v3    TikTok Scraper
    ↓           ↓
    Normalize & Merge
         ↓
    GPT-4o Summary
    5 actionable insights
         ↓
  ┌────────────────────┴───────────────────┐
  Google Drive            Obsidian
  GadgetLab/             Video-Insights/
  video-insights/        (m900-homelab)
  └────────────────────┬───────────────────┘
         ↓
  Slack Digest → #all-gadget-ai
         ↓
  Error? → #security-alerts
```

## Repository Structure

```
gadgetlab-video-pipeline/
├── .github/
│   └── workflows/
│       └── ci.yml          # Validates JSON, checks required files, Slack alerts
├── docs/
│   └── runbook-video-ingestion.md  # Full runbook with secrets, monitoring, rotation
├── workflows/
│   └── video-pipeline.json         # N8N workflow scaffold (16 nodes + connections)
└── README.md
```

## N8N Workflow

**Live workflow:** [gadgetlab.app.n8n.cloud/workflow/Tt44DxPbsBFUXXjl](https://gadgetlab.app.n8n.cloud/workflow/Tt44DxPbsBFUXXjl)

| Node | Purpose |
|------|---------|
| New Message in #all-gadget-ai | Slack trigger, polls every 60s |
| Extract Video URLs | Regex extraction from message text |
| Split URLs | Process multiple URLs independently |
| Route by Platform | IF node: youtube vs tiktok |
| Fetch YouTube Metadata | YouTube Data API v3 |
| Fetch TikTok Metadata | TikTok scraper HTTP request |
| Normalize YouTube/TikTok Data | Unified metadata schema |
| Combine Metadata | Merge both platform branches |
| Generate AI Summary | GPT-4o: title + 5 insights |
| Save to Google Drive | Markdown to GadgetLab/video-insights/ |
| Save to Obsidian | POST to m900-homelab:27123 |
| Prepare + Post Slack Digest | 3-bullet summary to #all-gadget-ai |
| On Workflow Error + Send Error Alert | Auto-notifies #security-alerts |

## Required Credentials (N8N)

| Credential | API Key Source |
|------------|---------------|
| `SLACK_BOT_TOKEN` | Slack App → Bot Token |
| `YOUTUBE_API_KEY` | Google Cloud Console → YouTube Data API v3 |
| `OPENAI_API_KEY` | platform.openai.com |
| `GOOGLE_SERVICE_ACCOUNT` | Google Cloud Console → Service Accounts |

## Required GitHub Secrets

| Secret | Purpose |
|--------|---------|
| `SLACK_WEBHOOK_SECURITY_ALERTS` | CI failure notifications to #security-alerts |

## Quick Start

1. Set up N8N credentials (see table above)
2. Open the [live workflow](https://gadgetlab.app.n8n.cloud/workflow/Tt44DxPbsBFUXXjl) and connect credentials to each node
3. Set the Slack channel to `#all-gadget-ai`
4. Activate the workflow
5. Post a YouTube or TikTok URL in `#all-gadget-ai` — the pipeline fires automatically

## Runbook

See [docs/runbook-video-ingestion.md](docs/runbook-video-ingestion.md) for full operational details including error handling, monitoring, and secrets rotation.
