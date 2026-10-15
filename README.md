# n8n Hands-On — Level 2 (Intermediate) Assessments

> **Student / Trainee:** Abhishek Nikam (`abhishek.n@tayanasolutions.com`)  
> **Institution:** Tayana Academy  
> **Course:** n8n Workflow Automation (Level 2 — Intermediate Series)  
> **Target Platform:** n8n version 1.0.0+  
> **Repository:** [3OOM/n8n-hands-on-level-2](https://github.com/3OOM/n8n-hands-on-level-2)  

---

## 📌 Repository Overview

This repository serves as the official, centralized submission portfolio for the **n8n Workflow Automation — Level 2 (Intermediate)** curriculum at Tayana Academy. It expands upon foundational automation concepts to cover mission-critical enterprise design patterns, high-throughput data processing, advanced API orchestration, resilient error escalation, and production readiness.

Each assessment is isolated within its own dedicated directory containing strictly:
1. **Official Workflow JSON (`.json`)**: Fully exportable, self-contained workflow file with pre-pinned test execution states for instant import and inspection.
2. **Comprehensive Technical Documentation (`README.md`)**: In-depth architectural blueprint, node parameter settings, JavaScript expression logic, test plans, and LMS submission notes.
3. **Publication-Ready PDF Documentation (`README.pdf`)**: Print-ready, professionally styled PDF generated directly from the technical documentation.

---

## 📂 Assessments Index

| Folder | Topic / Assessment | Focus & Design Patterns | Difficulty | Status | Deliverables |
| :--- | :--- | :--- | :---: | :---: | :--- |
| [`topic-01-production-workflow-planning/`](./topic-01-production-workflow-planning/) | **Topic 1: Production Workflow Planning & Design Patterns**<br>Webhook Ingest (202) ➔ Idempotency Check ➔ Schema Validation ➔ Retrying Billing API ➔ Global Incident Alert | Idempotency, RFC 5322 Regex, Exponential Backoff, Error Trigger | Intermediate | ✅ Completed | [Workflow JSON](./topic-01-production-workflow-planning/Topic_1_Production_Workflow_Planning.json) • [Docs (MD)](./topic-01-production-workflow-planning/README.md) • [Docs (PDF)](./topic-01-production-workflow-planning/README.pdf) |
| `topic-02-advanced-data-transformation/` *(upcoming)* | **Topic 2: Advanced Data Transformation & JMESPath Querying** | JMESPath, Complex Arrays, Custom Code | Intermediate | ⏳ Upcoming | — |

---

## 🚀 How to Import Any Workflow into n8n

Follow these simple steps to import and run any workflow from this repository in your n8n workspace:

1. **Clone or Download this repository:**
   ```bash
   git clone https://github.com/3OOM/n8n-hands-on-level-2.git
   ```
2. **Open your n8n workspace:**
   - Cloud, Docker, or local instance (`http://localhost:5678`).
3. **Import the Workflow:**
   - Navigate to **Workflows** in the left sidebar.
   - Click the **`...` (More options)** button in the top-right corner.
   - Select **Import from File**.
   - Browse and choose the desired `.json` workflow file (e.g. `topic-01-production-workflow-planning/Topic_1_Production_Workflow_Planning.json`).
4. **Inspect Pre-Pinned Data & Run:**
   - All nodes contain pre-pinned execution data representing verified test runs.
   - Click **Test step** on any node to trace the data transformation or run the full canvas.

---

## 🔒 Security & Privacy Policy

- **Zero Secrets / Tokens:** All exported workflows in this repository strictly adhere to enterprise security guidelines. No private API keys, client secrets, passwords, or active OAuth access tokens are stored in the workflow source files or documentation.
- **Mock Sandbox Compatibility:** Third-party integrations utilize zero-blocker mock cloud endpoints (DummyJSON, Postman Mock Server, or In-n8n sandboxes) allowing frictionless reproduction without live merchant gateway dependencies.
