# JobPilot AI
An AI-powered job application assistant that automates job search, resume coaching, interview preparation, and scheduling — built with a serverless automation backend and a modern React frontend.

**Live demo:** [https://jobwiz-assistant-the-second-brain.lovable.app](https://jobwiz-assistant-the-second-brain.lovable.app)

---

## Overview

JobPilot AI helps job seekers move from "searching for roles" to "walking into an interview prepared" in one connected flow. Instead of juggling job boards, resume edits, and prep notes across different tools, the entire workflow lives in a single application:

1. **Search** for jobs matched to your skills and location
2. **Analyze** how well your resume matches a specific job description, with AI-generated, ATS-focused suggestions
3. **Generate** a personalized, day-by-day interview preparation plan
4. **Save and schedule** that plan directly to Google Docs and Google Calendar

The backend is built entirely on workflow automation (n8n) rather than custom servers, integrating live job search data, an LLM, and Google Workspace APIs into a single pipeline.

---

## How it works

```
React frontend (Lovable)
        │
        ▼
Cloudflare Worker (CORS proxy)
        │
        ▼
n8n (hosted on Render) ── 5 independent webhook-triggered workflows
        │
        ├── SerpAPI            → live job search results
        ├── Google Gemini      → resume analysis & interview plan generation
        ├── Google Docs API    → saves the prep plan
        └── Google Calendar API → schedules daily prep sessions
```

Each feature (job search, resume coaching, prep plan generation, doc saving, calendar scheduling) is its own independently triggered n8n workflow, communicating with the frontend over a webhook API and routed through a lightweight Cloudflare Worker to handle cross-origin requests cleanly.

---

## Features

**Job Search**
Searches live job postings by title, skills, and location (multi-location supported), scores each result against the user's skill set, and filters out excluded keywords.

**AI Resume Coach**
Rather than auto-rewriting a resume (which often produces generic, hard-to-trust output), this analyzes the uploaded resume against the full job description and returns specific, actionable feedback: a match score, missing skills, bullet point rewrites, ATS keywords to add, and copy-ready snippets the user can apply themselves.

**Interview Prep Plan Generator**
Given a target role, company, interview date, and available prep hours, generates a structured day-by-day study plan — including company-specific research pulled from live search results — rendered as clean, readable formatted content (not raw JSON or markdown text).

**Save & Schedule**
One click saves the generated prep plan to Google Docs and creates daily calendar events leading up to the interview date, so prep time is blocked out automatically.

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React (built and deployed via Lovable) |
| Automation / backend | [n8n](https://n8n.io) (self-hosted on Render) |
| LLM | Google Gemini |
| Job search data | SerpAPI (Google Jobs engine) |
| Google integrations | Google Calendar API, Google Docs API, Google Drive API |
| CORS handling | Cloudflare Workers |
| Hosting | Render (n8n), Lovable (frontend) |

---

## Architecture decisions

**Why n8n instead of a custom backend?**
Each feature in this app is fundamentally a pipeline — receive input, call an external API or LLM, transform the output, return a response. n8n made it possible to build, test, and iterate on five independent workflows visually, swap models/providers without redeploying code, and get built-in execution logs for debugging — while keeping the system entirely serverless.

**Why a Cloudflare Worker in front of n8n?**
n8n's webhook responses don't natively expose CORS headers in a way that satisfied browser preflight requests on the hosting platform used here. Rather than fighting platform-level CORS configuration, a thin Cloudflare Worker sits between the frontend and n8n, forwarding requests and attaching the correct CORS headers — a pattern that's also useful for centralizing rate limiting or auth in the future.

**Why an "AI coach" instead of an auto-rewritten resume?**
Early versions of this project had Gemini fully rewrite the user's resume. In practice, fully automated rewrites are harder to trust and edit. The current approach treats the AI as a reviewer: it diffs the resume against the job description and hands back specific, explainable suggestions the user applies themselves — keeping a human in the loop on their own resume.

---

## n8n workflows

The automation logic is split into five independently triggered workflows (exported as JSON in [`/n8n-workflows`](./n8n-workflows)):

| Workflow | Trigger | Purpose |
|---|---|---|
| `job-search` | Webhook (POST) | Queries SerpAPI, scores results against user skills |
| `resume-coach` | Webhook (POST) | Sends resume + JD to Gemini, parses structured feedback |
| `prep-plan` | Webhook (POST) | Researches company context, generates day-by-day plan via Gemini |
| `save-docs` | Webhook (POST) | Creates and writes the prep plan to a Google Doc |
| `schedule-calendar` | Webhook (POST) | Generates and creates daily prep events in Google Calendar |

---

## Project status

This is an actively developed personal project. Current focus areas:
- Improving job description completeness from source listings
- Expanding multi-location search coverage
- Refining the resume coach output format

---

## Setup (high level)

This project relies on several external services. To run your own instance you'll need:

1. A Google Cloud project with Calendar, Docs, Drive, and Gmail APIs enabled, and OAuth credentials configured
2. A [SerpAPI](https://serpapi.com) key
3. A [Google AI Studio](https://aistudio.google.com) API key for Gemini
4. An n8n instance (self-hosted, e.g. on Render) with the workflows in `/n8n-workflows` imported and credentials configured
5. A Cloudflare Worker forwarding requests to your n8n instance with CORS headers attached
6. The frontend configured with your worker URL as the webhook base

---

## Author

Built by Sri Krishna Charan Sai.
