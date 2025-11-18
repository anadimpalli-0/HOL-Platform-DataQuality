# College of Platform - Snowflake Data Quality HOL

**Trusted data is vital for confident decisions, analytics, and AI. Snowflake’s Data Quality Framework ensures data remains accurate, complete, and reliable end-to-end.**

---

### 🛠️ Hands-On Lab Overview

This lab enables Solutions Engineers to learn, demonstrate, and articulate Snowflake Data Quality capabilities end-to-end. You will stand up a small but realistic data pipeline, define quality rules, validate data with tasks, observe results in Snowsight, and operationalize alerts and remediation patterns. The focus is on practical workflows that mirror customer scenarios, with checkpoints to confirm value at each step.

**Objectives**
- Understand how to design quality controls using constraints, policies, and programmatic checks
- Implement automated validation using tasks, streams, and procedures
- Monitor and triage quality issues in Snowsight with query history, dashboards, and alerts
- Ability to set up a governed landing-to-consumption flow with quality gates

**Target Audience**
- SEs preparing for customer demos, POVs, and enablement sessions
- Field engineers needing repeatable patterns to show value quickly

### 📋 What You’ll Do:
- Task 1: Provision a lab schema, roles, and warehouses
- Task 2: Ingest sample data (landing → raw) and create a curated table
- Task 3: Define quality rules using constraints, conditional logic, and reference checks
- Task 4: Build automated validation with tasks and procedures
- Task 5: Track metrics and surface outcomes in Snowsight (dashboards or worksheets)
- Task 6: Trigger alerts on failed checks; quarantine or fix bad data

Detailed instruction on executing the Lab can be found [HERE](/lab_instructions/README.md)

DQ Lab WorkFlow 


<img src="/images/DQ_Workflow.png" width="70%">

### ⏲️ Estimated Lab Timeline

- Setup and prerequisites: ~20 minutes
- Data ingestion and modeling: ~20 minutes
- Rules and validation automation: ~30 minutes
- Monitoring, alerting,remediation & Cleanup: ~20 minutes
- DORA Grading: ~5 minutes
  
---

## 📖 Table of Contents

- [Why this Matters](#-why-this-matters)
- [Suggested Discovery Questions](#-suggested-discovery-questions)
- [Repository Structure](#-repository-structure)
- [Prerequisites & Setup Details](#-prerequisites--setup-details)
- [Lab Instructions](#️-lab-instructions)
- [Grading](#️-grading)
- [Cleanup & Cost-Stewardship Procedures](#-cleanup--cost-stewardship-procedures)
- [Pro Tips, Talking Points, and Resources](#pro-tips-talking-points-and-resources)
- [Author & Support](#-author--support)

---

## 📌 Why this Matters
Customers expect data products to be reliable, observable, and governed. This lab demonstrates how Snowflake’s native capabilities enforce quality at ingestion and transformation without moving data to separate tools. You’ll show how to:
- **Reduce Risk:** prevent bad data from reaching downstream analytics and AI
- **Lower TCO:** avoid additional pipelines, storage, and syncs to external quality tools
- **Improve Trust:** quantify quality with metrics that business stakeholders recognize
- **AI Readiness:** High-quality, dependable data is essential for AI. AI models fueled by poor data will generate untrustworthy and potentially harmful outcomes.

---

## ❓ Suggested Discovery Questions

Provide **5 to 6 open-ended questions** for customer conversations related to this HOL.

**General**
- What downstream decisions or SLAs depend on this data being correct and timely?
- How do you detect bad data today, and who triages issues?
- What remediation patterns (quarantine, defaulting, rejection) are acceptable for your business?
- Which stakeholders need visibility (Ops, Data Stewards, Execs), and how?

**Persona Prompts**
- Data Engineer: Which checks catch the majority of incidents? How do you automate validation in CI/CD and jobs? What’s your rollback strategy?
- Platform Architect: Where do you prefer enforcement—at ingestion, transformation, or consumption? How do quality, lineage, and governance align in your architecture?
- Data Product Owner: What business rules define “fit for use”? Which metrics matter for stakeholder trust?
- CIO/CTO: What’s the cost of bad data to the business? How would a platform-native approach reduce TCO and complexity?

---

## 📂 Repository Structure

```bash
├── README.md           # Main entry point
├── Lab_instructions/   # Step-by-step detailed instructions
├── images/             # Images for the Lab Instructions
├── config/             # Configuration for DORA and Grading
└── troubleshooting/    # Common issues and resolutions
```
---

## ✅ Prerequisites & Setup Details

Internally helpful setup requirements:
- Snowflake account (ACCOUNTADMIN or equivalent to bootstrap; will create least-privileged roles)
- Ability to create warehouses, databases, schemas, roles, and tasks
- Snowsight access for Worksheets and Dashboards

---
### 🛠️ Lab Instructions

Detailed instruction on executing the scripts can be found [HERE](/lab_instructions/README.md)

---
## ⚠️ Grading 
Detailed instruction on Grading instructions can be found [HERE](config/readme.md)

---
## 🧹 Cleanup & Cost-Stewardship Procedures

- Run these commands
```sql```

 USE ROLE ACCOUNTADMIN;

 DROP DATABASE dq_tutorial_db;

 DROP WAREHOUSE dq_tutorial_wh;

 DROP ROLE dq_tutorial_role;

---

## Pro Tips, Talking Points, and Resources

**Pro Tips for SEs**
- Lead with outcome: show failed rows quarantined and dashboards updating
- Keep SQL small and readable; emphasize native, in-platform ops
- Map each rule to a business risk (revenue, compliance, SLA) during the demo

**Key Talking Points**
- Platform-native: no data movement; leverage tasks, policies, governance, and observability
- Extensible: start with SQL rules; grow into Snowpark, procedures, and alerts
- Cost-efficient: use auto-suspend warehouses and only-run-on-change patterns

**Resources**
- [DQ UI Documentation](https://docs.snowflake.com/user-guide/data-quality-ui-monitor)
- [Anomaly Detection](https://docs.snowflake.com/en/LIMITEDACCESS/data-quality/anomaly-detection)
- [DQ Expectations](https://docs.snowflake.com/en/LIMITEDACCESS/data-quality/expectations)
- [Integrated Notifications](https://docs.snowflake.com/en/LIMITEDACCESS/data-quality/notifications)
- [UI Demo](https://www.loom.com/share/49a37c1b02034f44a6ea196e4c3f4d49)

---

End of Lab.

---

## 👤 Author & Support

**Lab created by:** Aparna Nadimpalli – SE Enablement Senior Manager  
**Created on:** October 29, 2025 | **Last updated:** November 17, 2025 

💬 **Need Help or Have Feedback?**  
- Slack Channel: [#College-of-Platform](#)  
- Slack DM: [@aparna.nadimpalli](https://snowflake.enterprise.slack.com/team/U03RQG03MJR)  
- Email: [aparna.nadimpalli@snowflake.com](mailto:aparna.nadimpalli@snowflake.com)

🌟 *We greatly value your feedback to continuously improve our HOL experiences!*
