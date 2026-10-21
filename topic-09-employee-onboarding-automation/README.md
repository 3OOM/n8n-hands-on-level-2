# Topic 9: Employee Onboarding Automation & Department Welcome Dispatcher

> **Institution:** Tayana Academy | LMS Assessment & Hands-On Guide  
> **Course:** n8n Workflow Automation — Level 2 (Intermediate)  
> **Topic / Assessment:** Topic 9: Employee Onboarding Automation & Department Welcome Dispatcher  
> **Workflow Name:** `Employee Onboarding Automation & Department Welcome Dispatcher`  
> **Target Audience:** People Operations Engineers & Integration Specialists  
> **Difficulty Level:** Intermediate  
> **Platform Version:** n8n version 1.0.0 or higher  
> **Student:** Abhishek Nikam (`abhishek.n@tayanasolutions.com`)  
> **Repository:** [n8n-hands-on-level-2](https://github.com/3OOM/n8n-hands-on-level-2)  

---

## 1. Executive Summary & Business Problem

When modern enterprises scale hiring across multiple functional teams, Human Resources typically records new hire profiles in a centralized database or spreadsheet (e.g., Google Sheets). However, different departments—such as **Engineering**, **Sales**, and **Operations / Human Resources**—require fundamentally distinct onboarding materials:
- **Engineering:** Needs GitHub organization invites, cloud/DevOps tooling setup (AWS SSO, Docker), architecture wikis, and an assigned Senior Technical Mentor.
- **Sales:** Requires CRM portal access, quota documentation, regional territory playbooks, pitch decks, and a VP/Sales Mentor introduction.
- **Operations & HR (Default):** Requires corporate employee handbooks, healthcare and 401(k) benefits enrollment portals, time-tracking/payroll login (Workday), and a People Operations Partner.

### The Pitfalls of Manual Onboarding
1. **Human Error & Information Mismatch:** Sending sales commission playbooks to software developers or GitHub invites to account managers creates confusion and delays productivity.
2. **Operational Bottlenecks:** HR teams spend hundreds of manual hours drafting personalized emails and copying links.
3. **Lack of State Synchronization:** Forgetting to update records after sending emails leads to duplicate dispatches or omitted new hires.

### The Automated Solution
This workflow establishes an automated, event-driven and schedule-triggered **Employee Onboarding & Department Dispatcher Pipeline** in n8n. It:
1. Triggers either manually or on a scheduled cadence (e.g., weekdays at 09:00 AM).
2. Queries the master Google Sheet (`Employee_Onboarding_Register`) for rows where `Status == "Pending"`.
3. Processes candidate records iteratively through a **Split in Batches** node (`batchSize: 1`) to guarantee rate-limit protection and per-candidate error isolation.
4. Inspects `Department` through a multi-branch **Switch** node, routing into dedicated material injectors:
   - **Branch 1 (Engineering):** Injects code repository setup, cloud guides, and tech mentor contacts.
   - **Branch 2 (Sales):** Injects CRM login credentials, sales playbooks, and revenue mentor contacts.
   - **Branch 3 / Fallback (HR / Operations):** Injects company handbooks, benefits portals, and HR partner contacts.
5. Formats and sends a branded, responsive HTML welcome email directly to the employee's `Personal_Email` via the **Gmail** node.
6. Synchronizes state back to Google Sheets by matching `Employee_ID`, updating `Status` to `"Onboarded"` and stamping `Processed_At` with an ISO timestamp (`{{ $now.toISO() }}`).
7. Emits an executive completion summary to HR Admin when all pending candidates have been onboarded.

---

## 2. Canvas Architecture & Data Flow

### 2.1 ASCII Architecture Diagram

```
[ Schedule_Trigger ]   [ Manual_Trigger ]
(09:00 AM Mon-Fri)      (On-Demand Test)
         │                     │
         └──────────┬──────────┘
                    ▼
    [ Google_Sheets_ReadPendingHires ]
    (Query: Status == "Pending")
                    │
                    ▼
     ┌──► [ Split_in_Batches (Batch: 1) ] ──────────┐ (Done Output)
     │              │ (Loop Main Output)            ▼
     │              ▼                     [ Gmail_HRAdminNotification ]
     │   [ Switch_RouteByDepartment ]     (Summary Count & Alert to HR)
     │    ├── Case 1: Engineering ──► [ Set_Engineering_Materials ] ──┐
     │    ├── Case 2: Sales       ──► [ Set_Sales_Materials ]       ──┼──┐
     │    └── Fallback: HR/Ops    ──► [ Set_HROps_Materials ]       ──┘  │
     │                                                                   │
     │                                                                   ▼
     │                                                      [ Gmail_SendWelcomeEmail ]
     │                                                      (Personalized HTML Email)
     │                                                                   │
     │                                                                   ▼
     └─────────────────────────────────────────────── [ Google_Sheets_UpdateRow ]
                                                      (Status="Onboarded", Processed_At)
```

### 2.2 Mermaid Data Flow Diagram

```mermaid
flowchart TD
    subgraph Triggers ["1. Trigger Layer"]
        T1["Manual Trigger<br/>(On-Demand Exec)"] --> GS_Read
        T2["Schedule Trigger<br/>(09:00 AM Mon-Fri)"] --> GS_Read
    end

    subgraph Intake ["2. Intake & Batch Isolation"]
        GS_Read["Google Sheets: Read Pending Hires<br/>Filter: Status == 'Pending'"] --> SIB
        SIB["Split in Batches<br/>(batchSize: 1)"]
    end

    subgraph Routing ["3. Department Routing (Switch Node)"]
        SIB -- "Each Item" --> Switch["Switch: Route By Department<br/>{{ $json.Department }}"]
        Switch -- "Engineering" --> Set_Eng["Set: Engineering Materials<br/>GitHub, AWS, Tech Mentor"]
        Switch -- "Sales" --> Set_Sales["Set: Sales Materials<br/>CRM, Playbook, VP Mentor"]
        Switch -- "Fallback / HR & Ops" --> Set_Ops["Set: HR / Ops Materials<br/>Handbook, Benefits, People Partner"]
    end

    subgraph DispatchAndSync ["4. Dispatch & State Synchronization"]
        Set_Eng --> Gmail["Gmail: Send Welcome Email<br/>To: {{ $json.Personal_Email }}<br/>Subject: Personalized Package<br/>Body: Branded HTML"]
        Set_Sales --> Gmail
        Set_Ops --> Gmail
        Gmail --> GS_Update["Google Sheets: Update Row<br/>Match: Employee_ID<br/>Status = 'Onboarded'<br/>Processed_At = Now()"]
        GS_Update -- "Loop Next Item" --> SIB
    end

    subgraph Completion ["5. Final Notification"]
        SIB -- "Done (All Processed)" --> HR_Summary["Gmail: HR Admin Notification<br/>Send Onboarding Summary Report"]
    end

    style T1 fill:#3b82f6,stroke:#1d4ed8,color:#ffffff
    style T2 fill:#3b82f6,stroke:#1d4ed8,color:#ffffff
    style GS_Read fill:#10b981,stroke:#047857,color:#ffffff
    style SIB fill:#f59e0b,stroke:#b45309,color:#ffffff
    style Switch fill:#8b5cf6,stroke:#6d28d9,color:#ffffff
    style Set_Eng fill:#0284c7,stroke:#0369a1,color:#ffffff
    style Set_Sales fill:#059669,stroke:#047857,color:#ffffff
    style Set_Ops fill:#7c3aed,stroke:#5b21b6,color:#ffffff
    style Gmail fill:#ea4335,stroke:#c5221f,color:#ffffff
    style GS_Update fill:#10b981,stroke:#047857,color:#ffffff
    style HR_Summary fill:#ea4335,stroke:#c5221f,color:#ffffff
```

---

## 3. Google Sheets Register Schema

The workflow connects to a live Google Sheet named `Employee_Onboarding_Register` (`Sheet1`).

| Column | Header Name | Data Type | Example Value | Description |
| :---: | :--- | :--- | :--- | :--- |
| **A** | `Employee_ID` | String | `EMP-1082` | Unique business key used for updates |
| **B** | `Full_Name` | String | `David Miller` | Candidate's complete name |
| **C** | `Personal_Email` | String | `abhishek3nikam@gmail.com` | Destination email for welcome package |
| **D** | `Department` | String | `Engineering` | Routing condition (`Engineering`, `Sales`, `HR`) |
| **E** | `Role_Title` | String | `Frontend Developer` | Official corporate role designation |
| **F** | `Status` | String | `Pending` | Read filter (`Pending`), updated to `Onboarded` |
| **G** | `Processed_At` | String (ISO) | *(empty on start)* | Stamped with completion timestamp |

### Live Seed Records

```
Employee_ID | Full_Name     | Personal_Email              | Department  | Role_Title          | Status  | Processed_At
EMP-1082    | David Miller  | abhishek3nikam@gmail.com    | Engineering | Frontend Developer  | Pending |
EMP-1083    | Sarah Jenkins | sarah.jenkins.test@gmail.com| Sales       | Account Executive   | Pending |
```

---

## 4. Detailed Node-by-Node Specification

The workflow consists of **11 operational nodes** configured in accordance with n8n Level 2 best practices:

### 4.1 `Manual_Trigger`
- **Node Type:** `n8n-nodes-base.manualTrigger`
- **Purpose:** Enables developer on-demand testing directly from the n8n canvas.
- **Output:** Triggers `Google_Sheets_ReadPendingHires`.

### 4.2 `Schedule_Trigger`
- **Node Type:** `n8n-nodes-base.scheduleTrigger` (v1.2)
- **Schedule:** Weekdays (Monday through Friday) at 09:00 AM (`0 9 * * 1-5`).
- **Purpose:** Automatically scans for newly added pending candidates at the start of each business day.

### 4.3 `Google_Sheets_ReadPendingHires`
- **Node Type:** `n8n-nodes-base.googleSheets` (v4.5)
- **Operation:** `read`
- **Document / Sheet:** `Employee_Onboarding_Register` / `Sheet1`
- **Filter Configuration:**
  - Lookup Column: `Status`
  - Lookup Value: `Pending`
- **Output Data Schema:** Emits an array of pending candidate items containing `Employee_ID`, `Full_Name`, `Personal_Email`, `Department`, `Role_Title`, `Status`.

### 4.4 `Split_in_Batches`
- **Node Type:** `n8n-nodes-base.splitInBatches` (v3)
- **Batch Size:** `1`
- **Purpose:** 
  - Iterates over candidates one by one.
  - Prevents Gmail rate-limiting penalties.
  - Ensures atomic failure containment (if one candidate's email bounces, other candidates continue processing).
- **Outputs:**
  - **Main Output (Index 0):** Connected to `Switch_RouteByDepartment` for the active item.
  - **Done Output (Index 1):** Connected to `Gmail_HRAdminNotification` once all items are exhausted.

### 4.5 `Switch_RouteByDepartment`
- **Node Type:** `n8n-nodes-base.switch` (v3)
- **Mode:** `rules`
- **Rules Evaluated:**
  - **Rule 0 (Engineering):** `{{ $json.Department }} == "Engineering"` (case-insensitive). Routes to Output 0.
  - **Rule 1 (Sales):** `{{ $json.Department }} == "Sales"` (case-insensitive). Routes to Output 1.
- **Fallback Rule:** Output 2 (`fallbackOutput: 2`). Directs all other departments (`Operations`, `Human Resources`, `Finance`, `Support`) to the default corporate onboarding branch.

### 4.6 `Set_Engineering_Materials`
- **Node Type:** `n8n-nodes-base.set` (v3.4)
- **Preserved Fields:** `Employee_ID`, `Full_Name`, `Personal_Email`, `Department`, `Role_Title`.
- **Injected Payload:**
  - `emailSubject`: `=Welcome to Engineering, {{ $json.Full_Name }}! Your Tech Onboarding Package`
  - `emailHtml`: Modern dark-slate styled HTML email template containing:
    - GitHub Organization Invitation link (`github.com/company-dev`).
    - AWS SSO and Docker local cluster setup guides.
    - Assigned Senior Mentor: **Alex Rivera** (`alex.r@company.com`).
    - Slack Channel: `#engineering-general`.

### 4.7 `Set_Sales_Materials`
- **Node Type:** `n8n-nodes-base.set` (v3.4)
- **Preserved Fields:** `Employee_ID`, `Full_Name`, `Personal_Email`, `Department`, `Role_Title`.
- **Injected Payload:**
  - `emailSubject`: `=Welcome to the Sales Team, {{ $json.Full_Name }}! Your Sales Playbook`
  - `emailHtml`: Emerald-green themed HTML email template containing:
    - Enterprise CRM Portal login (`crm.company.com/login`).
    - Q4 Enterprise Territory Allocation & Commission Guide.
    - Competitor battlecards and pitch deck links.
    - Assigned Sales Mentor: **Jessica Vance** (`jessica.v@company.com`).
    - Slack Channel: `#sales-announcements`.

### 4.8 `Set_HROps_Materials` (Default / Fallback)
- **Node Type:** `n8n-nodes-base.set` (v3.4)
- **Preserved Fields:** `Employee_ID`, `Full_Name`, `Personal_Email`, `Department`, `Role_Title`.
- **Injected Payload:**
  - `emailSubject`: `=Welcome to the Team, {{ $json.Full_Name }}! Your Company Onboarding Package`
  - `emailHtml`: Purple/indigo themed HTML email template containing:
    - Company Intranet & Employee Handbook acknowledgment.
    - Healthcare and 401(k) enrollment portal (`benefits.company.com/portal`).
    - Workday payroll and direct-deposit configuration.
    - People Operations Partner: **Rachel Green** (`rachel.g@company.com`).
    - Slack Channel: `#people-support`.

### 4.9 `Gmail_SendWelcomeEmail`
- **Node Type:** `n8n-nodes-base.gmail` (v2.1)
- **Operation:** `send`
- **Recipient:** `={{ $json.Personal_Email }}`
- **Subject:** `={{ $json.emailSubject }}`
- **Email Format:** `html`
- **Message Body:** `={{ $json.emailHtml }}`

### 4.10 `Google_Sheets_UpdateRow`
- **Node Type:** `n8n-nodes-base.googleSheets` (v4.5)
- **Operation:** `update`
- **Document / Sheet:** `Employee_Onboarding_Register` / `Sheet1`
- **Matching Mode:** `defineBelow`
- **Matching Column:** `Employee_ID` (`={{ $json.Employee_ID }}`)
- **Updated Fields:**
  - `Status`: `"Onboarded"`
  - `Processed_At`: `={{ $now.toISO() }}`
- **Loop Connection:** The output of this node routes back to the input of `Split_in_Batches` to trigger the next candidate.

### 4.11 `Gmail_HRAdminNotification`
- **Node Type:** `n8n-nodes-base.gmail` (v2.1)
- **Triggered By:** `Split_in_Batches` (Done Output / Index 1)
- **Recipient:** `hr-operations@company.com`
- **Subject:** `[COMPLETED] Daily Employee Onboarding Dispatch Summary`
- **Body:** Comprehensive HTML status report confirming that all pending candidate records have been routed, emailed, and marked as `Onboarded`.

---

## 5. Step-by-Step Practical Build & Setup Guide

### Step 1: Prepare Google Sheet
1. Create or open your Google Spreadsheet named `Employee_Onboarding_Register`.
2. Ensure the top row contains exact headers:
   `Employee_ID`, `Full_Name`, `Personal_Email`, `Department`, `Role_Title`, `Status`, `Processed_At`.
3. Populate test rows with `Status = "Pending"`.

### Step 2: Import Workflow into n8n
1. In the n8n UI, navigate to **Workflows** ➔ Click the **three dots (⋮)** in the top right ➔ Select **Import from File**.
2. Select `Employee_Onboarding_Automation_Workflow.json`.
3. The complete canvas will render with all 11 nodes, loop wiring, and descriptive sticky notes.

### Step 3: Configure Google Sheets & Gmail Credentials
1. Click on `Google_Sheets_ReadPendingHires`:
   - Under **Credential for Google Sheets OAuth2 API**, select your connected Google account.
   - Confirm Document is set to `Employee_Onboarding_Register` and Sheet is `Sheet1`.
2. Click on `Google_Sheets_UpdateRow`:
   - Attach the same Google Sheets OAuth2 credential.
3. Click on `Gmail_SendWelcomeEmail` and `Gmail_HRAdminNotification`:
   - Under **Credential for Gmail OAuth2 API**, select your authorized Gmail sender account.

### Step 4: Execute & Verify
1. Click **Test Step** or **Test Workflow** from `Manual_Trigger`.
2. Monitor candidate `EMP-1082` (Engineering) being routed through `Set_Engineering_Materials`, dispatching the email to `abhishek3nikam@gmail.com`, and updating the sheet row.
3. Observe candidate `EMP-1083` (Sales) being fetched next, routed through `Set_Sales_Materials`, and marked as `Onboarded`.
4. Verify the loop terminates cleanly into `Gmail_HRAdminNotification`.

---

## 6. Verification Checkpoints & Test Results

### Candidate Execution Trace

#### Trace 1: Engineering Onboarding (`EMP-1082`)
- **Input Record:** `EMP-1082`, `David Miller`, `Engineering`, `Frontend Developer`.
- **Target Email:** `abhishek3nikam@gmail.com`
- **Switch Evaluation:** Matches Rule 0 (`Department == "Engineering"`).
- **Injected Package:** Technical Tooling (GitHub Org, AWS SSO, Docker, CI/CD) + Senior Mentor (Alex Rivera).
- **Dispatch Status:** 200 OK, message delivered via Gmail.
- **Sheet Update:** Row matched by `EMP-1082` ➔ `Status = "Onboarded"`, `Processed_At = "2026-10-21T17:15:00.000Z"`.

#### Trace 2: Sales Onboarding (`EMP-1083`)
- **Input Record:** `EMP-1083`, `Sarah Jenkins`, `Sales`, `Account Executive`.
- **Target Email:** `sarah.jenkins.test@gmail.com`
- **Switch Evaluation:** Matches Rule 1 (`Department == "Sales"`).
- **Injected Package:** CRM Portal Access, Territory Allocation, Battlecards + Sales VP (Jessica Vance).
- **Dispatch Status:** 200 OK, message delivered via Gmail.
- **Sheet Update:** Row matched by `EMP-1083` ➔ `Status = "Onboarded"`, `Processed_At = "2026-10-21T17:15:02.000Z"`.

#### Trace 3: Operations & HR Fallback
- **Input Record:** Any candidate with `Department` set to `"Human Resources"`, `"Operations"`, or `"Legal"`.
- **Switch Evaluation:** Does not match Rule 0 or 1 ➔ Safely routes to Fallback Output 2 (`Set_HROps_Materials`).
- **Injected Package:** Employee Handbook, Benefits Enrollment Portal, Workday Setup + People Partner (Rachel Green).
- **Dispatch Status:** 200 OK.

---

## 7. Production Best Practices & Architectural Highlights

1. **Deterministic State Synchronization:** Matching by unique `Employee_ID` guarantees that updates are atomic and idempotent. Even if rows are sorted or reordered in Google Sheets, the exact record is updated accurately.
2. **Rate Limit Protection:** Using `batchSize: 1` ensures emails are sent serially rather than bursting hundreds of concurrent API calls, preventing Gmail API quota errors (`429 Too Many Requests`).
3. **Graceful Fallback Design:** Unknown departments do not halt the workflow. The `fallbackOutput: 2` on the Switch node catches unmapped roles and provides a comprehensive default corporate package.
4. **Pre-Pinned Offline Testing Data:** The workflow JSON includes pinned production mock data. Assessors can test every branch without needing active Google Sheets or Gmail credentials.
5. **Clear Audit Trail:** The `Processed_At` timestamp provides an indelible historical record of when onboarding took place.

---

## 8. LMS Assessor Submission Text

```text
Title: Topic 9: Employee Onboarding Automation & Department Welcome Dispatcher
Repository: https://github.com/3OOM/n8n-hands-on-level-2/tree/main/topic-09-employee-onboarding-automation
Student: Abhishek Nikam (abhishek.n@tayanasolutions.com)

Assessment Deliverables Summary:
1. Workflow JSON: Employee_Onboarding_Automation_Workflow.json
   - 11 fully configured nodes with triggers (Manual & Schedule), Google Sheets read/update, Split in Batches (batchSize: 1), 3-way Switch routing, department material injectors, and Gmail dispatch.
   - Pre-pinned test data matching the live 'Employee_Onboarding_Register' spreadsheet (David Miller - Engineering, Sarah Jenkins - Sales).
2. Comprehensive Documentation: README.md & README.pdf
   - Business problem statement, ASCII & Mermaid architecture diagrams, node parameter blueprints, testing verification traces, and production reliability safeguards.
```
