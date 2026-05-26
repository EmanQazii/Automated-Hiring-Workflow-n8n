# Multi-Role AI Hiring Automation System

A fully automated hiring pipeline built in n8n that handles end-to-end recruitment for two roles: Software Engineer (SWE) and Business Development Manager (BDM). From the moment a candidate applies to the moment they receive a response, everything runs automatically with zero manual work.

---

## What It Does

- Ingests applications from Google Forms into Google Sheets
- Downloads and extracts text from candidate resumes (PDF) stored in Google Drive
- Evaluates each resume using Mistral AI with role-specific criteria
- Scores and classifies candidates as strong, average, or weak
- Sends personalized HTML emails based on classification
- Creates Google Calendar interview events for strong candidates
- Notifies the hiring manager for average candidates
- Sends warm rejection emails to weak candidates
- Tracks every candidate in a central Master Sheet

---

## System Architecture

The system runs as two separate n8n workflows:

### Workflow 1: Main Processing Pipeline
Runs every 2 hours. Handles all data processing.

```
Schedule Trigger (every 2 hours)
        |
Read SWE Sheet + Read BDM Sheet (parallel)
        |
Merge both streams
        |
Filter unprocessed candidates
        |
IF no candidates: exit gracefully
        |
Loop over candidates one at a time
        |
Clean and standardize data
        |
IF resume exists:
  - Download PDF from Google Drive
  - Extract resume text
  - Build role-specific AI prompt
  - Send to Mistral AI
  - Parse AI response (score, classification, summary)
  - Write to Master Sheet
  - Mark as Processed in source sheet
ELSE:
  - Log as edge case and skip
```

### Workflow 2: Email Dispatcher
Runs every 30 minutes. Handles all candidate communication.

```
Schedule Trigger (every 30 minutes)
        |
Read Master Sheet
        |
Filter candidates where Email Sent = No
        |
IF no pending candidates: exit gracefully
        |
Loop over candidates one at a time
        |
Switch by classification:
  - Strong: Interview invitation email + Google Calendar event
  - Average: Hiring manager notification email
  - Weak: Professional rejection email
        |
Update Master Sheet: Email Sent = Yes, Status updated
```

---

## AI Evaluation

Each role is evaluated on completely different criteria.

### Software Engineer Criteria
- Programming languages and frameworks
- Project complexity and real-world impact
- GitHub activity and open source contributions
- System design and architecture experience
- Overall technical depth

### Business Development Manager Criteria
- Sales and revenue track record
- Client relationship management
- Market expansion experience
- Communication and leadership skills
- Industry network and business acumen

### Scoring Thresholds
| Classification | Score Range |
|---|---|
| Strong | 60 and above |
| Average | 35 to 59 |
| Weak | Below 35 |

---

## Tech Stack

| Component | Tool |
|---|---|
| Automation | n8n |
| Application Forms | Google Forms |
| Data Storage | Google Sheets |
| Resume Storage | Google Drive |
| AI Model | Mistral AI (mistral-small-latest) |
| Email | Gmail API |
| Calendar | Google Calendar API |
| PDF Extraction | n8n Extract from File node |

---

## Google Sheets Structure

### SWE Applications Sheet
```
Timestamp | Full Name | Email | Phone Number | Years of Experience
Tech Stack | GitHub Profile Link | Upload Your Resume | Processed
```

### BDM Applications Sheet
```
Timestamp | Full Name | Email | Phone Number | Years of Experience
Industry Background | LinkedIn Profile URL | Upload Your Resume | Processed
```

### Master Candidates Sheet
```
Candidate ID | Name | Email | Role | Resume Link | Score
Classification | Summary | Strengths | Concerns | Interview Recommended
Status | Email Sent | Timestamp | row_number
```

---

## Setup Instructions

### Prerequisites
- n8n installed locally or n8n Cloud account
- Google account with access to Forms, Sheets, Drive, Gmail, Calendar
- Mistral AI API key (free at console.mistral.ai)
- Google Cloud project with OAuth2 credentials

### Step 1: Google Cloud Setup
1. Go to console.cloud.google.com
2. Create a new project
3. Enable these APIs:
   - Google Sheets API
   - Google Drive API
   - Gmail API
   - Google Calendar API
4. Create OAuth2 credentials (Web Application type)
5. Add redirect URI: `http://localhost:5678/rest/oauth2-credential/callback`

### Step 2: Google Forms Setup
1. Create SWE form with fields: Full Name, Email, Phone, Years of Experience, Tech Stack, GitHub Profile, Resume Upload
2. Create BDM form with fields: Full Name, Email, Phone, Years of Experience, Industry Background, LinkedIn Profile, Resume Upload
3. Link each form to its own Google Sheet via the Responses tab
4. Add a Processed column to each linked sheet

### Step 3: Master Sheet Setup
Create a new Google Sheet named Master Candidates with headers:
```
Candidate ID | Name | Email | Role | Resume Link | Score | Classification
Summary | Strengths | Concerns | Interview Recommended | Status | Email Sent | Timestamp | row_number
```

### Step 4: n8n Setup
1. Install n8n: `npm install -g n8n`
2. Start n8n: `n8n`
3. Open http://localhost:5678
4. Go to Credentials and add:
   - Google Sheets OAuth2
   - Google Drive OAuth2
   - Gmail OAuth2
   - Google Calendar OAuth2
5. Import the workflow JSON file

### Step 5: Configure Workflow
After importing update these values in the nodes:
- SWE Sheet ID
- BDM Sheet ID
- Master Sheet ID
- Mistral API key in the HTTP Request URL
- Hiring manager email in the Average branch Gmail node

---

## Workflow JSON

The file `HiringAutomation.json` contains both workflows packaged together. Import it into n8n via:

```
Workflows > Import > Select file
```

Both workflows will appear in your n8n instance ready to configure.

---

## Edge Cases Handled

- No new applications: workflow exits cleanly without errors
- Missing resume: candidate is logged and skipped, loop continues
- AI response parsing failure: candidate defaults to average classification
- SWE vs BDM routing: handled by IF node checking role field
- Duplicate prevention: Processed and Email Sent columns prevent reprocessing

---

## Known Limitations

- Mistral free tier has rate limits, large batches may get throttled
- Only PDF resumes are supported
- Interview scheduling does not account for weekends or holidays
- No retry mechanism if AI API call fails
- Duplicate applications from same email are not deduplicated

---


## Author

Eman Qazi
- GitHub: github.com/EmanQazii
- LinkedIn: linkedin.com/in/eman-qazi
- Email: emanqazi786@gmail.com
