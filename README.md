# 🏥 VCBHN IAM Security Lab

**ValleyCrest Behavioral Health Network — Identity & Access Management Security Operations**

> A portfolio project demonstrating healthcare IAM security operations in a HIPAA-regulated environment. Built under the SentinelDrift brand to showcase real-world skills in access lifecycle management, IAM anomaly detection, and compliance-aligned security tooling.

---

## About VCBHN

ValleyCrest Behavioral Health Network (VCBHN) is a fictional multi-site behavioral health organization used as the consistent backdrop for this portfolio. It operates under:

- **HIPAA Privacy & Security Rules** (45 CFR Parts 160 and 164)
- **42 CFR Part 2** — Substance Use Disorder record protections
- **NIST SP 800-66 Rev 2** — Implementing the HIPAA Security Rule
- **NIST CSF** — Cybersecurity Framework

This lab focuses on the **Identity & Access Management (IAM)** security operations function — one of the highest-risk domains in healthcare security due to the sensitivity of PHI and the complexity of clinical access requirements.

---

## What This Project Demonstrates

| Skill | How It's Shown |
|-------|---------------|
| Access lifecycle management | Python access review script with HIPAA-mapped risk rules |
| IAM anomaly detection | 4 production-ready KQL queries for Microsoft Sentinel |
| Security monitoring & dashboards | Deployable Sentinel Workbook with scorecards and trend charts |
| Security policy and procedures | Two full SOPs covering provisioning and exception handling |
| Healthcare compliance (HIPAA) | Every artifact maps to specific HIPAA and NIST controls |
| Least-privilege enforcement | Risk rules flag over-privileged accounts and policy violations |
| Incident response documentation | Exception SOP and runbook language throughout |

---

## Repository Structure

```
vcbhn-iam-security-lab/
│
├── access-review/
│   ├── access_review.py               # Quarterly access certification automation
│   ├── mock_users.csv                 # Simulated VCBHN user roster
│   └── vcbhn_access_review_report.html  # Generated HTML report (run script to produce)
│
├── kql-detections/
│   ├── 01_stale_account_phi_access.kql          # CRITICAL: Inactive accounts accessing PHI
│   ├── 02_after_hours_privilege_escalation.kql  # HIGH: Priv changes outside change window
│   ├── 03_bulk_permission_changes.kql           # HIGH: Mass IAM changes in short window
│   └── 04_new_admin_outside_change_window.kql   # CRITICAL: Backdoor admin account creation
│
├── sentinel-workbook/
│   └── vcbhn_iam_workbook.json        # Deployable Azure Sentinel Workbook (IAM Dashboard)
│
└── sops/
    ├── SOP-IAM-001-Provisioning-Deprovisioning.md   # Full provisioning/offboarding runbook
    └── SOP-IAM-002-Security-Exception-Handling.md  # Exception request and approval process
```

---

## Component 1 — Access Review Automation

**File:** `access-review/access_review.py`

Automates the quarterly HIPAA access certification review. Ingests a user roster CSV, applies risk rules, and generates a prioritized HTML report for the access review committee.

### Risk Rules

| Severity | Rule |
|----------|------|
| 🔴 CRITICAL | Stale account (90+ days inactive) with active PHI access |
| 🔴 CRITICAL | Orphaned account — no manager, PHI access retained |
| 🔴 CRITICAL | Access review overdue 365+ days |
| 🟠 HIGH | Non-IT user assigned Admin role (least-privilege violation) |
| 🟠 HIGH | Service or temp account with PHI access |
| 🟡 MEDIUM | Access review overdue 180+ days |
| 🟡 MEDIUM | Non-clinical staff (HR, Facilities, Finance) with PHI access |
| 🔵 LOW | Access review approaching due date (90+ days) |

### Running the Script

```bash
cd access-review
python access_review.py
```

Output: Console summary + `vcbhn_access_review_report.html`

### Sample Output

```
============================================================
  VCBHN Quarterly Access Review
  ValleyCrest Behavioral Health Network
============================================================

  [CRITICAL] ghost.account        → Stale Account with PHI | Orphaned | Review Overdue 500d
  [CRITICAL] service.account.lab  → Orphaned Account with PHI Access | Review Overdue
  [CRITICAL] temp.nurse.2024      → Stale Account with PHI | Orphaned | Review Overdue
  [HIGH    ] Patricia Dunn        → Non-IT User Assigned Admin Role
  [INFO    ] Amy Chen             ✓ Passed

  SUMMARY: 20 accounts reviewed
  CRITICAL   6  |  HIGH  2  |  MEDIUM  2  |  LOW  5  |  PASSED  5
```

---

## Component 2 — KQL IAM Detection Pack

**Directory:** `kql-detections/`

Four production-ready KQL queries for Microsoft Sentinel, each targeting a distinct IAM threat pattern in a healthcare environment.

| File | Severity | Threat Scenario | MITRE |
|------|----------|----------------|-------|
| `01_stale_account_phi_access.kql` | 🔴 CRITICAL | Inactive account accesses PHI system | T1078 |
| `02_after_hours_privilege_escalation.kql` | 🟠 HIGH | Role/group changes outside change window | T1078.004, T1098 |
| `03_bulk_permission_changes.kql` | 🟠 HIGH | 5+ IAM changes in 10 minutes by one admin | T1098 |
| `04_new_admin_outside_change_window.kql` | 🔴 CRITICAL | New account + admin role assigned after hours | T1136.003 |

Each query includes:
- Inline comments explaining the detection logic
- Configurable thresholds
- HIPAA control and MITRE ATT&CK mapping
- Recommended response action

**Deploying to Sentinel:**
1. Open Microsoft Sentinel → Logs
2. Paste query content into the query editor
3. Validate against your log schema
4. Save as Analytics Rule with appropriate severity and alert grouping

---

## Component 3 — Sentinel IAM Workbook

**File:** `sentinel-workbook/vcbhn_iam_workbook.json`

A deployable Azure Sentinel Workbook providing real-time IAM security monitoring.

**Dashboard Includes:**
- 📊 Scorecard tiles: Stale PHI accounts, after-hours changes, orphaned accounts, admin assignments
- 📈 Privilege change trend (business hours vs. after-hours time series)
- 🚨 Active IAM alert table with severity color coding
- 👥 Top 10 accounts by IAM activity volume
- 🥧 IAM operation breakdown pie chart

**Deploying the Workbook:**
1. Navigate to Microsoft Sentinel → Workbooks → Add Workbook
2. Switch to the JSON editor (Advanced Editor)
3. Paste the contents of `vcbhn_iam_workbook.json`
4. Click **Apply** and **Save**

---

## Component 4 — IAM Standard Operating Procedures

**Directory:** `sops/`

Two operational runbooks built to HIPAA-regulated healthcare standards.

### SOP-IAM-001: User Provisioning & Deprovisioning
Covers the full user lifecycle:
- New hire provisioning checklist (including PHI access approval gates)
- Admin account provisioning with PIM requirements
- Service account governance
- Deprovisioning timelines by termination type (voluntary, involuntary, role change)
- Involuntary termination — immediate action protocol

### SOP-IAM-002: Security Exception Handling
Covers the full exception lifecycle:
- Exception categories and approval tiers
- Risk assessment criteria
- Approval, denial, and escalation paths
- Exception monitoring and 30-day review cadence
- Closure and lessons learned process
- Exception Register field definitions

---

## Compliance Mapping

| HIPAA Control | What It Requires | How This Lab Addresses It |
|---------------|-----------------|--------------------------|
| §164.308(a)(1) | Security Management Process | Exception SOP, risk assessment criteria |
| §164.308(a)(3) | Workforce Access Management | Provisioning SOP, access review automation |
| §164.308(a)(4) | Access Authorization | Exception approval workflow, PHI access gates |
| §164.312(a)(1) | Access Control | KQL detections, workbook monitoring |
| §164.312(b) | Audit Controls | KQL queries targeting audit log events |

---

## Author

**Leon Kiyonga** | mataleon
SOC Analyst · Healthcare IT Security · Detection Engineering

- 🔗 [GitHub: SentinelDrift](https://github.com/mataleon)
- 🔗 [LinkedIn](https://linkedin.com/in/leonkiyonga)

---

*This is a portfolio project. VCBHN is a fictional organization. All user data in mock_users.csv is fabricated.*
