# Academic Plagiarism Detection Automation Bot

An end-to-end RPA pipeline built in **UiPath Studio** that automates academic plagiarism checking — from reading student submissions to scoring, recommendations, and dual email alerts — eliminating manual review.

## Overview

Faculty manually checking each submission for plagiarism is slow, inconsistent, and easy to miss. This bot monitors a submissions folder, extracts text from PDF/TXT files, checks similarity via the **Copyleaks API**, scores and classifies each submission, logs results to Excel, and emails both the professor and the flagged student automatically.

## How It Works

1. **File ingestion** — scans the input folder for student submissions (PDF/TXT), naming convention: `StudentName - AssignmentName.pdf`
2. **Text extraction** — `Read PDF Text` for normal PDFs, with automatic fallback to `Read PDF With OCR` (Tesseract) for scanned documents; `Read Text File` for `.txt`
3. **Student lookup** — case-insensitive match against a student database Excel file to retrieve each student's email
4. **Plagiarism check (Copyleaks API)**:
   - Authenticate and retrieve a Bearer access token
   - Submit extracted text for scanning via HTTP PUT, with a webhook URL for async results
   - Poll the webhook (via webhook.site) to retrieve the similarity report
5. **Scoring & recommendation** — parses the aggregated score (0–100) and assigns a recommendation:
   - `> 70%` → Reject submission immediately
   - `40–70%` → Manual review recommended
   - `< 40%` → Minor revision required
   - `≤ 30%` → No action / no email triggered
6. **Email alerts** — for flagged submissions, sends Gmail SMTP notifications to both the professor and the student with the score and recommendation
7. **Excel logging** — writes `StudentName`, `FileName`, `PlagiarismScore`, and `Date` to a results spreadsheet

## Tech Stack

- **UiPath Studio** (VB.NET expressions) — workflow orchestration
- **Copyleaks API** — plagiarism similarity scoring
- **webhook.site** — asynchronous webhook result delivery
- **Gmail SMTP** — automated email notifications
- **Microsoft Excel** — student database & results logging
- **Newtonsoft.Json** — JSON parsing of API responses
- **Tesseract OCR** — fallback text extraction for scanned PDFs

## Error Handling

- All API calls and the result-parsing loop are wrapped in Try/Catch — a failure on one file logs an error and the bot continues to the next
- Email sending is independently wrapped in Try/Catch to handle SMTP failures gracefully
- Missing student emails are logged as warnings rather than crashing the workflow

## Results

Tested end-to-end with a deliberately plagiarized sample document — the bot correctly detected a **96.7% similarity score**, generated the "Reject submission immediately" recommendation, sent both email alerts, and logged the result to Excel.

## Setup

1. Open the project in **UiPath Studio** (Legacy framework, dependencies listed in `project.json`)
2. Configure folder paths for input files, student database, and output Excel
3. Add your own credentials:
   - Copyleaks account email + API key
   - A fresh webhook.site UUID
   - Gmail address + App Password for SMTP
4. Run `Main.xaml`

> **Note:** All credentials in this repo are placeholders. You'll need your own Copyleaks API key, webhook.site endpoint, and Gmail App Password to run the bot.

## Challenges Solved

- Webhook URL expiry → generate a fresh UUID per run
- Premature polling returning score `0` → increased delay from 15s to 45s to allow Copyleaks processing time
- Score formatting bug (`10000%`) → removed redundant `*100` multiplication
- Case-sensitive student name matching → applied `.ToLower()` on both sides of the lookup
- Gmail SMTP auth failures → resolved via App Password regeneration

---

*Built as part of the Robotic Process Automation coursework (24AM6PERPA), B.M.S College of Engineering.*
