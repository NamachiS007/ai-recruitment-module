# 🏢 HRM — Recruitment Module

![Status](https://img.shields.io/badge/status-active-brightgreen) ![Stack](https://img.shields.io/badge/stack-React%20%7C%20Python%20%7C%20PostgreSQL-blue) ![AI](https://img.shields.io/badge/AI-NLP%20%7C%20Resume%20Parsing%20%7C%20RAG-orange) ![Tools](https://img.shields.io/badge/tools-FastAPI%20%7C%20spaCy%20%7C%20Transformers-yellow) ![License](https://img.shields.io/badge/license-MIT-lightgrey)

![Status](https://img.shields.io/badge/status-active-brightgreen) ![Stack](https://img.shields.io/badge/stack-React%20%7C%20Python%20%7C%20PostgreSQL-blue) ![AI](https://img.shields.io/badge/AI-NLP%20%7C%20Resume%20Parsing%20%7C%20RAG-orange) ![Tools](https://img.shields.io/badge/tools-FastAPI%20%7C%20spaCy%20%7C%20Transformers-yellow) ![License](https://img.shields.io/badge/license-MIT-lightgrey)

> A full-cycle **Recruitment Management System** — handling everything from AI-assisted job creation and LinkedIn publishing to resume screening, interview scheduling, and offer delivery.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Module Architecture](#module-architecture)
- [Features](#features)
  - [1. Add New Job](#1-add-new-job)
  - [2. Manage Jobs](#2-manage-jobs)
    - [LinkedIn Job Publishing](#linkedin-job-publishing)
  - [3. Candidates List](#3-candidates-list)
  - [4. Candidate Detail](#4-candidate-detail)
- [Tech Stack](#tech-stack)
- [Database Schema](#database-schema)
- [API Reference](#api-reference)
- [Event-Driven Architecture](#event-driven-architecture)
- [Workflow Summary](#workflow-summary)

---

## Overview

**HRM** is a personal full-stack project simulating an enterprise-grade HR platform. This module focuses on **Recruiting & Talent Acquisition**, managing the complete lifecycle of a job vacancy — from posting to hiring.

```
Job Created → Published → Applications Received → AI Screening → Interview → Offer Sent
```

---

## Module Architecture

```
Recruiting
├── Add New Job          ← Create & configure vacancies
├── Manage Jobs          ← Monitor listings & applicant counts
│   └── Publish          ← LinkedIn external job publishing
│   └── Candidate Detail
│     ├── Overview         ← Profile & contact info
│     ├── AI Analysis      ← Resume parsing & match score
│     ├── Interview Schedule
│     └── Evaluation Notes
├── Candidates           ← Global talent pool view
```

---

## Features

### 1. Add New Job

**Location:** `Recruiting > Add New Job`

Allows HR administrators to create job vacancies with rich metadata and AI-assisted job descriptions.

#### Form Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Job Title | Text | ✅ | Official position name |
| Department | Dropdown | ✅ | Linked to `org_departments` |
| Position Type | Dropdown | ✅ | Full-time / Part-time / Contract |
| Experience | Text | ✅ | Format: `"X-Y years"` |
| Work Type | Dropdown | ✅ | On-site / Remote / Hybrid |
| Location | Text | ❌ | Office or city |
| Salary Range | Dual Input | ❌ | Min and Max (numeric) |
| No. of Positions | Number | ✅ | Headcount quota |
| Job End Date | Date Picker | ✅ | Must be ≥ current date |

#### AI-Assisted Job Description

- Users provide a **custom prompt** (e.g., *"Focus on React experience and soft skills"*)
- `[Generate Job Description]` triggers a call to the LLM backend
- Output is rendered in a **WYSIWYG Rich Text Editor** (Quill / CKEditor)

#### Resume Parsing Threshold

A slider (0–100%) that sets the minimum AI match score for a candidate to pass initial screening. Candidates scoring below this threshold are automatically flagged.

> **Default:** `50%` — stored as an integer in the database.

#### Validation Rules

- `Job End Date >= Current Date`
- `Max Salary >= Min Salary`
- No negative values for numeric fields
- `[Preview & Save]` triggers a summary modal before committing to DB

---

### 2. Manage Jobs

**Location:** `Recruiting > Manage Jobs`

Central dashboard for monitoring all active and archived job postings.

#### Search & Filter

| Filter | Description |
|---|---|
| Job Title | Partial/full text search |
| Department | Dropdown by org unit |
| From / To Date | Date range for creation/posting date |
| Job Status | `PUBLISHED` / `CLOSED` / `DRAFT` |

#### Data Grid Columns

| Column | Description |
|---|---|
| Job ID | System-generated unique identifier |
| Job Title | Vacancy name |
| Department | Assigned department |
| Work Location | Office or Remote |
| Work Type | Employment type |
| Applicants | Count + `[View]` link to candidate list |
| Job Status | 🟢 Published / 🔴 Closed |
| Job End Date | Application deadline |

> Clicking **"View"** on the applicants column auto-filters the Candidates page to that specific `job_id`.

#### LinkedIn Job Publishing

**Location:** `Recruiting > Manage Jobs > Publish`

The **Publish** tab inside the Job Configuration modal allows recruiters to push finalized job postings directly to LinkedIn via the **LinkedIn Marketing Developer Platform**, expanding vacancy reach to a wider professional audience instantly.

##### OAuth 2.0 Integration Requirements

| Credential | Description |
|---|---|
| Client ID | Unique identifier for the HRM LinkedIn app |
| Client Secret | Confidential key for server-side authentication |
| Permissions | `w_member_social` (member feed) or `w_organization_social` (company page) |

##### Publishing Workflow

```
1. System verifies Platform Status = CONFIGURED
       ↓
2. Recruiter clicks [Publish Job]
       ↓
3. User redirected to LinkedIn OAuth login (new tab)
       ↓
4. LinkedIn returns Authorization Code on success
       ↓
5. HRM backend exchanges code for Access Token
       ↓
6. Post is automatically executed on LinkedIn
       ↓
7. User redirected to Manage Jobs with "Published" confirmation
```

##### Publishing Sub-Grid Fields

| Field | Description |
|---|---|
| Job ID | Internal tracking number (e.g., `198`) |
| Job Status | Internal state (e.g., `PUBLISHED`) |
| Platform Status | LinkedIn API readiness (`CONFIGURED`) |
| Publish Action | Primary button to initiate the external API call |

##### Error Handling

| Scenario | System Response |
|---|---|
| Expired Token | UI prompts user to re-authenticate |
| Invalid Client ID / Secret | Displays `"Authentication Failed"` alert |

---

### 3. Candidates List

**Location:** `Recruiting > Candidates`

Global talent pool dashboard — search across all candidates regardless of job.

#### Filters

| Field | Type |
|---|---|
| Candidate Name | Text (partial match) |
| Job Title | Text — filters by applied role |
| Candidate Status | Dropdown: Active / Interviewing / Onboarded |

#### Data Grid

- Name, Email, Phone
- Total Experience (parsed from resume)
- Number of positions applied
- Pipeline status (color-coded)
- `[View]` → redirects to Candidate Detail

#### Split-View Layout

Selecting a candidate row loads an **"Applied Jobs" panel** on the right, showing the candidate's full history across departments — without navigating away.

> Filters are persisted via **URL parameters** to support bookmarking.

---

### 4. Candidate Detail

**Location:** `Recruiting > Manage Jobs > [View Applicants] > Candidate`

Deep-dive profile with four structured tabs:

#### Tab 1 — Overview
- Contact info: Email, Phone
- Application status history
- Resume download (`[Download Resume]`)

#### Tab 2 — AI Analysis
- Displays parsed skill match percentage vs. job requirements
- Visual indicator: **Strong Match** / **Partial Match** / **Below Threshold**
- Shows `"Resume Not Processed"` alert if parsing failed or resume is missing

#### Tab 3 — Interview Schedule
- Calendar integration (Teams / Zoom links)
- Stage tracking: `Technical Round 1` → `HR Round` → etc.

#### Tab 4 — Evaluation Notes
- Collaborative space for interviewers
- Star ratings (1–5) + qualitative comments
- Used to justify the final hiring decision

#### Persistent Action Bar (Header)

| Action | Description |
|---|---|
| `[Offer Letter]` | Generates and sends offer PDF to candidate's email |
| `[Update Status]` | Moves candidate through pipeline stages |
| `[Change Status Reason]` | Audit trail entry (e.g., *"Offer Sent on 2026-04-18"*) |
| `[Download Resume]` | Quick download for final review |

**Pipeline Stages:**
```
Sourcing → Screening → Interviewing → Offered → Hired / Rejected
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React.js + Material UI |
| **Backend** | Java / Spring Boot |
| **Database** | PostgreSQL / MySQL |
| **Messaging** | Apache Kafka |
| **Rich Text** | Quill / CKEditor |
| **Email** | SMTP Service |
| **AI/LLM** | Internal LLM Backend |
| **External Integration** | LinkedIn Marketing Developer Platform (OAuth 2.0) |

---

## Database Schema

### `job_postings` Table

```sql
CREATE TABLE job_postings (
    job_id            SERIAL PRIMARY KEY,
    job_title         VARCHAR(255) NOT NULL,
    dept_id           INT NOT NULL,
    pos_type_id       INT,
    experience_req    VARCHAR(50),
    work_type         VARCHAR(50),
    job_location      VARCHAR(100),
    salary_min        DECIMAL(12, 2),
    salary_max        DECIMAL(12, 2),
    num_positions     INT DEFAULT 1,
    job_end_date      DATE,
    skills_tech       TEXT,
    special_requirements TEXT,
    ai_prompt         TEXT,
    parsing_threshold INT DEFAULT 50,
    job_description_html TEXT,
    linkedin_published_at TIMESTAMP,          -- Set when job is pushed to LinkedIn
    platform_status   VARCHAR(20) DEFAULT 'CONFIGURED', -- CONFIGURED / LIVE
    created_at        TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### LinkedIn Publish Tracking Query

```sql
UPDATE job_postings 
SET linkedin_published_at = CURRENT_TIMESTAMP, 
    platform_status = 'LIVE' 
WHERE job_id = :job_id;
```

### Key Query — Manage Jobs Grid

```sql
SELECT 
    j.job_id, j.job_title, j.department, j.work_location,
    j.work_type, j.job_status, j.end_date,
    (SELECT COUNT(*) FROM candidate_applications c WHERE c.job_id = j.job_id) AS applicant_count
FROM jobs j
WHERE j.org_id = :current_org_id
ORDER BY j.created_at DESC;
```

### Key Query — Candidate AI Analysis

```sql
SELECT 
    c.full_name, c.email,
    a.parsing_score, a.ai_summary,
    j.parsing_threshold
FROM candidate_profiles c
JOIN job_applications a ON c.id = a.candidate_id
JOIN job_postings j ON a.job_id = j.job_id
WHERE c.id = :selected_candidate_id;
```

---

## API Reference

### AI Job Description Generation

```
POST /api/v1/ai/generate-job-desc
```

**Request:**
```json
{
  "job_title": "React Developer",
  "skills": "React, TypeScript, Material UI",
  "prompt": "Highlight the need for 3 years of experience in enterprise apps."
}
```

**Response:**
```json
{
  "generatedHtml": "<h3>Role Overview</h3><p>We are seeking...</p>"
}
```

---

### Save Job Listing

```
POST /api/v1/jobs/create
```
Returns `201 Created` on success.

---

### Fetch Manage Jobs Grid

```
GET /api/v1/jobs/manage
```

**Query Params:** `jobTitle`, `deptId`, `status`, `startDate`, `endDate`

**Response:** JSON array of job objects including `applicantCount`.

---

### Send Offer Letter

```
POST /api/v1/recruiting/send-offer
```

**Request:**
```json
{
  "candidate_id": "DK",
  "template_id": "standard_offer",
  "sender_id": "current_user"
}
```

Generates a PDF offer letter and dispatches it via SMTP to the candidate's registered email.

---

### Publish Job to LinkedIn

```
POST https://api.linkedin.com/v2/ugcPosts
```

**Request Payload:**
```json
{
  "author": "urn:li:organization:12345",
  "lifecycleState": "PUBLISHED",
  "specificContent": {
    "com.linkedin.ugc.ShareContent": {
      "shareCommentary": {
        "text": "We are hiring! Check out our new opening for: AI Developer"
      },
      "shareMediaCategory": "NONE"
    }
  },
  "visibility": { "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC" }
}
```

> Authentication via **OAuth 2.0** — HRM backend exchanges the Authorization Code for an Access Token before executing this call.

---

## Event-Driven Architecture

All major state transitions publish events to **Apache Kafka**:

| Kafka Topic | Trigger | Consumers |
|---|---|---|
| `hrm.job.created` | Job saved with `Active` status | Notification Service, Portal Sync Service |
| `hrm.job.published` | Job successfully posted to LinkedIn | Dashboard stats update, Audit log |
| `hrm.candidate.status_change` | Candidate status updated | Manage Jobs dashboard (real-time count update) |

### `hrm.job.created` Payload

```json
{
  "job_id": "uuid",
  "job_title": "Senior Java Developer",
  "created_at": "2026-05-01T10:00:00Z"
}
```

---

## Workflow Summary

```
1. HR creates a job vacancy          →  Add New Job
       ↓
2. AI generates job description      →  LLM Backend
       ↓
3. Job goes live, candidates apply   →  Manage Jobs (monitoring)
       ↓
4. Job pushed to LinkedIn            →  OAuth 2.0 → LinkedIn API → platform_status = LIVE
       ↓
5. Resumes auto-screened by AI       →  Parsing Threshold filter
       ↓
6. Shortlisted candidates reviewed   →  Candidate Detail > AI Analysis
       ↓
7. Interviews scheduled & evaluated  →  Interview Schedule + Evaluation Notes
       ↓
8. Offer letter generated & sent     →  Offer Letter Action → SMTP
       ↓
9. Candidate marked Hired/Rejected   →  Status updated → Kafka event fired
```

---

> 🚀 **HRM** — Personal Project | Recruiting & Talent Acquisition Module v1.0  
> Complete Flow: Job Creation → Management → LinkedIn Publishing → Candidate Assessment → Offer Delivery
