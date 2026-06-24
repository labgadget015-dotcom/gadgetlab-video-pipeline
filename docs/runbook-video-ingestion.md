# Runbook: Video Ingestion Pipeline

> **Owner:** GadgetLab Automation  
> **Updated:** 2025-01-01  
> **N8N Workflow:** GadgetLab — TikTok & YouTube Auto-Ingestion Pipeline  
> **Workflow ID:** Tt44DxPbsBFUXXjl

## Overview

This runbook covers the end-to-end autonomous video ingestion pipeline. When a TikTok or YouTube URL is posted to the `#all-gadget-ai` Slack channel, this pipeline:

1. Extracts the URL(s) from the message
2. Fetches video metadata (title, description, tags, duration)
3. Generates a GPT-4o AI summary with 5 key actionable insights
4. Saves a markdown note to Google Drive (`GadgetLab/video-insights/`)
5. Creates an Obsidian note in the `Video-Insights` vault folder
6. Posts a digest back to `#all-gadget-ai` with title + 3-bullet summary + Drive link
7. Notifies `#security-alerts` if any step fails

## Prerequisites

| Secret | Location | Purpose |
|--------|----------|---------|
| `SLACK_BOT_TOKEN` | N8N Credentials | Slack API trigger + posting |
| `YOUTUBE_API_KEY` | N8N Credentials | YouTube Data API v3 metadata |
| `OPENAI_API_KEY` | N8N Credentials | GPT-4o summarisation |
| `GOOGLE_SERVICE_ACCOUNT` | N8N Credentials | Google Drive file creation |
| `SLACK_WEBHOOK_SECURITY_ALERTS` | GitHub Secrets | CI failure notifications |

## Triggering the Pipeline

Post any YouTube or TikTok URL to `#all-gadget-ai` in Slack. Examples:
```
https://www.youtube.com/watch?v=dQw4w9WgXcQ
https://www.tiktok.com/@user/video/1234567890
```
Multiple URLs in a single message are each processed independently.

## N8N Node Reference

| Node | Type | Notes |
|------|------|-------|
| New Message in #all-gadget-ai | Slack Trigger | Polls every 60s |
| Extract Video URLs | Code | Regex: `(https?://(?:www\.)?(?:youtube\.com/watch\?v=|youtu\.be/|tiktok\.com/@[^/]+/video/)\S+)` |
| Route by Platform | IF | Condition: URL contains `youtube` |
| Fetch YouTube Metadata | HTTP Request | YouTube Data API v3 |
| Fetch TikTok Metadata | HTTP Request | TikTok scraper endpoint |
| Generate AI Summary | OpenAI | GPT-4o, prompt: extract 5 insights |
| Save to Google Drive | Google Drive | Folder: GadgetLab/video-insights/ |
| Save to Obsidian | HTTP Request | POST to m900-homelab:27123 |
| Post Digest to Slack | Slack | Channel: #all-gadget-ai |
| Send Error Alert | Slack | Channel: #security-alerts |

## Error Handling

- **Node failure:** The On Workflow Error trigger catches failures and posts to `#security-alerts`
- **Invalid URL:** Extract URLs node returns empty array; downstream Split node handles gracefully with no-op
- **API rate limits:** YouTube API quota = 10,000 units/day; monitor via Google Cloud Console

## Monitoring

- View execution history: N8N Executions tab for workflow `Tt44DxPbsBFUXXjl`
- Check Google Drive: `GadgetLab/video-insights/` folder for generated markdown files
- Check Obsidian: `Video-Insights/` vault folder on m900-homelab

## Secrets Rotation

1. Rotate YOUTUBE_API_KEY every 90 days in Google Cloud Console
2. Rotate OPENAI_API_KEY via OpenAI dashboard
3. Update N8N credentials immediately after rotation
4. Test with a sample URL post to `#all-gadget-ai`
