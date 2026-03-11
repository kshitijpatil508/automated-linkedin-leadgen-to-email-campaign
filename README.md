# AI-Powered LinkedIn Lead Generation & Cold Email Campaign System 🚀

**A two-part n8n automation pipeline that scrapes LinkedIn for high-potential leads, enriches and scores them using AI, then runs a fully automated, multi-account, personalized cold email campaign with reply tracking.**

[![n8n](https://img.shields.io/badge/Built_With-n8n-orange?style=for-the-badge&logo=n8n)](https://n8n.io/)
[![Stack](https://img.shields.io/badge/Stack-Apify%20%7C%20Gemini%20%7C%20Groq%20%7C%20Gmail-blue?style=for-the-badge)](https://github.com/kshitijpatil508)
[![LinkedIn](https://img.shields.io/badge/Connect-LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/kshitij-patil-750914213/)

---

## 📖 Overview

Manual outreach at scale is broken. This system solves it end-to-end.

This system operates as a two-phase pipeline — **Part 1** intelligently builds a scored, enriched, and validated lead list by scraping LinkedIn and applying an AI scoring engine. **Part 2** takes those scored leads and runs a fully automated, time-aware, multi-account cold email campaign — complete with personalized templates, round-robin account assignment, and automatic reply tracking.

The result: a self-sustaining outbound system that continuously sources, qualifies, and engages B2B leads — with zero manual intervention after setup.

---

## 🗺️ System Architecture — The Full Pipeline

```
[ PART 1: Lead Intelligence ]
  01. Scrape LinkedIn Profiles via Apify (keyword + filter)
  02. Lead Enrichment via Hunter.io (build + verify email)
  03. Email Validation API (confirm deliverability)
  04. AI Lead Scoring via Gemini / Groq (0–100 score + reason)
        ↓
  [ Google Sheets: "Scored Leads Sheet" ]
        ↓
[ PART 2: Email Campaign Engine ]
  05. Campaign Initialization (migrate scored leads → campaign sheet, assign accounts)
  06. Personalized Multi-Account Email Sending (Schedule Trigger, 3 Gmail accounts)
  07. Reply Tracking & Status Update (Gmail Triggers → update sheet)
```

---

## ✨ Part 1 — Lead Intelligence: Features & Stack

| Stage | Feature | Description | Technology |
| --- | --- | --- | --- |
| **01** | **LinkedIn Scraping** | Scrapes profiles by keyword (Sales Leader, Coach), filters by role-based keywords (Outbound, Cold Calling), and fetches recent post URLs + comment data (Name, Company, Comment). | `Apify` (LinkedIn Scraper, Posts URLs & Insights, Comments Scraper) |
| **02** | **Lead Enrichment** | Fetches company domain from Hunter.io. If a sending pattern is returned (e.g. `{first}.{last}@company.com`), builds the lead's email address automatically. | `Hunter.io API`, `Code Node` |
| **03** | **Email Validation** | Validates all built emails via an Email Validator API. Invalid emails are flagged in the sheet; valid ones proceed to scoring. Batched with Wait nodes to respect API rate limits. | `HTTP Request`, `If Node`, `splitInBatches`, `Wait` |
| **04** | **AI Lead Scoring** | An AI Agent (Gemini + Groq) scores each lead from 0–100 based on authority, role relevance, and comment intent. A Structured Output Parser ensures consistent JSON output every time. Score, priority tier, and reason are written back to the sheet. | `lmChatGoogleGemini`, `lmChatGroq`, `langchain.agent`, `Structured Output Parser` |

---

## ✨ Part 2 — Email Campaign Engine: Features & Stack

| Stage | Feature | Description | Technology |
| --- | --- | --- | --- |
| **05** | **Campaign Initialization** | Reads all scored leads, filters out already-started ones, assigns each lead to a Gmail account using a round-robin formula (`rowNumber % 3 + 1`), sets status to `Initial Outreach`, and migrates data to the Email Campaign Sheet. | `googleSheets`, `Filter`, `Code Node` |
| **06** | **Safety & Timing Filter** | Acts as the "Traffic Controller" — blocks `Replied`, `Bounced`, and `Completed` leads. Enforces send gaps: Email 2 requires >24h after Email 1; Email 3 requires >48h after Email 2. | `Code Node`, `Schedule Trigger` |
| **06** | **Personalization Engine** | Selects the correct email template based on `Current Step`, injects `{{First Name}}` and `{{Company Name}}` placeholders with real data, and pre-calculates the next step + timestamp for the sheet update. | `Code Node` (Draft Personalized Email) |
| **06** | **Multi-Account Routing** | Routes each lead to its assigned Gmail account (Account 1, 2, or 3) via a Switch node, then sends the email and updates `Action Time` and `Current Step` in the sheet. | `Switch`, `gmail` (×3), `googleSheets` |
| **07** | **Reply Tracking** | Three Gmail Triggers (one per account) fire when a reply arrives. Each checks whether the sender's email exists in the campaign sheet. If matched, it updates `Replied Status` and logs the reply content. | `gmailTrigger` (×3), `Code Node`, `googleSheets` |

---



## 📸 Visuals

### Part 1 — LinkedIn Lead Intelligence Workflow

<img width="2170" height="1240" alt="image" src="https://github.com/user-attachments/assets/8564fabc-b7b7-4fb6-b4b7-440745b45864" />
<img width="2290" height="1230" alt="image" src="https://github.com/user-attachments/assets/9fdb6b07-d226-4ca9-bd04-4caca8fd6df7" />


---

### Part 2 — Email Campaign Engine Workflow

<img width="2252" height="878" alt="image" src="https://github.com/user-attachments/assets/6f0cc215-ecd7-4e41-b65d-530193808d9e" />
<img width="2716" height="930" alt="image" src="https://github.com/user-attachments/assets/b39a7786-d1b2-43d0-b4e3-de90f17f5049" />
<img width="2382" height="854" alt="image" src="https://github.com/user-attachments/assets/1ab1fcb9-c57f-4e83-b7a1-f2f3ae2b97ab" />

---

### Google Sheet — Scraped Raw Leads Sheet
<img width="1456" height="695" alt="Screenshot 2026-03-11 at 5 18 07 PM" src="https://github.com/user-attachments/assets/d1ad8849-4181-4235-905a-1d582f73ac63" />

---

### Google Sheet — Filtered Email & Scored Leads Sheet

<img width="1465" height="689" alt="Screenshot 2026-03-11 at 5 18 19 PM" src="https://github.com/user-attachments/assets/5f5a8e1a-af21-4919-8dc8-5dc8101c215a" />

---

### Google Sheet — Email Campaign Sheet

<img width="1466" height="685" alt="Screenshot 2026-03-11 at 5 18 31 PM" src="https://github.com/user-attachments/assets/d5e99fa6-a928-4ad5-a989-e17a9022f385" />

---
## 🛠️ Setup and Installation Guide

### Files

* **Part 1 Workflow:** `LinkedIn_Lead_Intelligence.json` — Import into your n8n instance.
* **Part 2 Workflow:** `Email_Campaign_Engine.json` — Import into your n8n instance.

---

### 1) Import the Workflows

1. Open your n8n editor.
2. Import `LinkedIn_Lead_Intelligence.json` first.
3. Import `Email_Campaign_Engine.json` second.

---

### 2) Google Sheets Setup

You need **two** Google Sheets with the following structure:

**Scraped Leads Sheet** (used by Part 1)

| Column | Purpose |
| --- | --- |
| `Name`, `Company`, `Role`, `LinkedIn URL` | Scraped from LinkedIn |
| `Email` | Built by Hunter.io pattern |
| `Email Status` | Set by Email Validator API (`Valid` / `Invalid`) |
| `Tier` | Set by Code Node based on score |
| `Score`, `Priority`, `Score Reason` | Written by AI Lead Scoring Agent |
| `Status` | Tracks pipeline state (e.g., `Campaign Started`) |

**Email Campaign Sheet** (used by Part 2)

| Column | Purpose |
| --- | --- |
| `First Name`, `Company Name`, `Email` | Migrated from Scraped Leads Sheet |
| `Assigned Account` | Round-robin assigned (1, 2, or 3) |
| `Campaign Status` | `Ready`, `In Progress`, `Completed` |
| `Current Step` | `Initial Outreach`, `First Follow up`, `Second Follow up` |
| `Last Action Date` | Timestamp of last email sent |
| `Replied` | `Yes` / `No` |
| `Reply Content` | Logged reply text |

---

### 3) Configure Credentials

Ensure all credentials below are added in your n8n instance:

| Credential | Purpose |
| --- | --- |
| **Google Sheets account** | Read/write both pipeline sheets |
| **Apify API Token** | LinkedIn profile scraping, post URLs, comment data |
| **Hunter.io API Key** | Company domain lookup and email pattern retrieval |
| **Email Validator API Key** | Validate deliverability of built emails |
| **Google Gemini API Key** | Primary LLM for lead scoring agent |
| **Groq API Key** | Fallback LLM for lead scoring agent |
| **Gmail Account 1** | Outreach sending account 1 + reply trigger |
| **Gmail Account 2** | Outreach sending account 2 + reply trigger |
| **Gmail Account 3** | Outreach sending account 3 + reply trigger |

---

### 4) Node Configuration

* **Apify Nodes:** Set your target LinkedIn search keywords (e.g., `Sales Leader`, `Sales Coach`) and the role-based filter keywords (e.g., `Outbound`, `Cold Calling`) in the `Filter Potential Influencers` Code node.
* **Hunter.io Node:** Confirm your API key is active and your rate limit tier matches the batch sizes set in the `splitInBatches` nodes.
* **AI Scoring Agent:** The system prompt is pre-configured (Authority → 40pts, Engagement → 30pts, Intent → 30pts). Modify it in the `Sticky Note4` reference node if you need to adjust scoring rules.
* **Gmail Nodes:** Open each `Account 1`, `Account 2`, `Account 3` node and ensure the correct credential is selected. Do the same for the three `gmailTrigger` nodes.
* **Google Sheets Nodes:** Update the `Spreadsheet ID` and `Sheet Name` in every Google Sheets node to point to your own files.

---

### 5) Test Run

1. **Part 1:** Execute `SWADES_AI_LINKEDIN_ASSIGNMENT.json` manually. Verify scraped leads appear in your Scraped Leads Sheet, with emails built, validated, and AI scores populated.
2. **Part 2 — Init:** Run the `Initialize Lead Campaign` flow manually to migrate scored leads to the Email Campaign Sheet and confirm round-robin account assignment.
3. **Part 2 — Campaign:** Activate the Schedule Trigger. Verify the first email batch goes out and the sheet updates `Current Step` and `Last Action Date` correctly.
4. **Reply Tracking:** Reply to a test email from one of the outreach accounts and confirm the Gmail Trigger fires and updates `Replied` status in the sheet.

---

## 🎯 How it Works — Step by Step

### Part 1: Lead Intelligence
1. **Scrape:** Apify scrapes LinkedIn profiles matching your keywords, fetches their recent post URLs, and collects comments data (Name, Company, Comment Text).
2. **Enrich:** Hunter.io identifies the company domain and email pattern. A Code node constructs the lead's email address.
3. **Validate:** The Email Validator API confirms deliverability. Invalid emails are flagged; valid leads advance.
4. **Score:** The AI Agent evaluates each lead on authority (role seniority), relevance (role + industry), and intent (comment engagement). Gemini and Groq return a structured score, priority tier, and reason, all written to the sheet.

### Part 2: Email Campaign Engine
1. **Initialize:** The init flow reads all leads with `Campaign Started` status, assigns each to a Gmail account via round-robin, sets `Current Step` to `Initial Outreach`, and writes them to the Email Campaign Sheet.
2. **Schedule:** A Schedule Trigger fires the main campaign flow at your configured interval.
3. **Filter:** The Safety & Timing Filter blocks ineligible leads and enforces the >24h / >48h wait gaps between follow-ups.
4. **Personalize:** The Personalization Engine selects the correct template for each lead's current step and injects real data into placeholders.
5. **Route & Send:** A Switch node routes each lead to its assigned Gmail account. The email is sent and the sheet is updated with the new step and timestamp.
6. **Reply Tracking:** Each Gmail account has a dedicated trigger listening for replies. On match, `Replied` and `Reply Content` are updated in the campaign sheet, preventing further follow-ups.

---

## 🤝 Contribution

Feel free to fork and adapt this for:
- Different lead sources (e.g., Twitter/X, Apollo, Sales Navigator)
- Additional CRM integrations (Zoho, HubSpot)
- More Gmail accounts for higher sending volume

---

## 📬 Connect

[![LinkedIn](https://img.shields.io/badge/Recruiters%20%7C%20LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/kshitij-patil-750914213/)
[![Email](https://img.shields.io/badge/Clients%20%7C%20Email_Me-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:kshitijpatil508@gmail.com)
