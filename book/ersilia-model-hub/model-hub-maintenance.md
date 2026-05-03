---
description: >-
  The Ersilia Model Hub is maintained through a dedicated maintenance
  repository. It describes the automated workflows, reports and processes used
  to track model health, testing and maintenance actions.
---

# Model Hub maintenance

## 1. Overview

The Ersilia Model Hub is maintained through a dedicated [maintenance repository](https://github.com/ersilia-os/ersilia-maintenance/tree/main) that provides automated infrastructure for monitoring, testing, and tracking models across the hub. This repository acts as a central coordination layer, ensuring that model metadata stays synchronized, model health is continuously assessed, and maintenance signals are made visible in a consistent and transparent way.

Rather than focusing on individual model development, the maintenance repository operates at the hub level, enabling systematic quality control, early detection of issues, and data-driven maintenance decisions through automated workflows and regularly updated reports.

## 2.Automated workflows and reports

The maintenance repository is driven by a set of scheduled automated workflows. Each workflow targets a specific aspect of model maintenance, such as metadata synchronization, continuous testing, or hub-level monitoring. Together, they ensure that the state of the Model Hub is regularly updated and that maintenance signals are generated in a consistent and reproducible way.

| Workflow                | Purpose                                                         | Schedule         | Output                                         |
| ----------------------- | --------------------------------------------------------------- | ---------------- | ---------------------------------------------- |
| Repo data sync          | Synchronize model metadata and repository status across the hub | Every 10 days    | Updated metadata and full model catalog report |
| Weekly model testing    | Periodically test a subset of models using the Ersilia CLI      | Weekly (Mondays) | Weekly model testing report                    |
| Monthly health analysis | Compute hub-level health metrics and trends                     | Monthly          | Monthly health report                          |

The workflows produce a set of public reports that provide structured and up-to-date visibility into the Model Hub. These reports are intended to support monitoring, issue tracking, and maintenance prioritization by summarizing model-level and hub-level information in a format that is easy to inspect and reference

| Report                                                                                                                     | Description                                                                    | Update frequency | Intended use                                        |
| -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ---------------- | --------------------------------------------------- |
| [Model report](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/model_report.md)                        | Full snapshot of all models in the hub, including metadata, status, and issues | Every 10 days    | Global overview and issue tracking                  |
| [Weekly model testing report](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/weekly_model_testing.md) | Detailed results of the latest model testing cycle                             | Weekly           | Detect failing or unstable models                   |
| [Monthly health report](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/monthly_health_report.md)      | Aggregated ecosystem-level analytics and historical trends                     | Monthly          | Monitor long-term health and maintenance priorities |

## 3. Maintenance guidelines

This section describes how the Ersilia team uses the maintenance repository to monitor model health, coordinate maintenance work, and resolve model-related issues in a timely and transparent manner.

#### 👤 Roles and responsibilities

The Lead Software Engineer is responsible for overseeing maintenance activities and ensuring that required actions are carried out. This includes:

* Coordinating maintenance work
* Assigning action items to team members during the weekly technology meeting
* Following up on their completion

When a team member is assigned to a maintenance task, they are accountable for the status of the assigned model or issue, including monitoring progress, coordinating fixes, and seeking support from other team members when needed.

If the designated person is unable to complete the assigned task, or identifies blockers that prevent timely resolution, this should be communicated to the Lead Software Engineer as early as possible. This allows tasks to be reassigned, priorities to be adjusted, or appropriate decisions to be made to ensure continuity of maintenance activities.

#### 🔁 Weekly maintenance activities

On a weekly basis, the team performs the following actions

*   🧪 **Review of weekly model testing results**

    The results of the weekly testing workflow are reviewed. If one or more models fail, a responsible team member is assigned to investigate the issue. This includes:

    1. Diagnosing the cause of the failure
    2. Applying fixes where possible
    3. Triggering the relevant testing workflows
    4. Reporting back to the team once the model is confirmed to be working again.

    These actions are expected to be completed before the next technology meeting.
*   🐛 **Review of models with open issues**

    Each week, a team member is assigned to review a subset of models (typically five) with open issues. The goal is to update issue status, add new information if available, and assess whether issues can be resolved or require further action.

#### ⏱️ Issue resolution and communication

The team aims to resolve maintenance issues within one week whenever possible. If an issue cannot be closed within that timeframe, its status should be updated with:

* The current state of the issue
* Any blocking factors
* An estimated resolution timeline

Regular updates are preferred over fast closure, ensuring that issues remain visible and that maintenance decisions are documented transparently. Issues may be closed once the problem has been resolved, mitigated, or explicitly deferred with justification.

#### 📊 Monthly review

Once per month, the team discusses the overall state of the Model Hub in the technology meeting. This includes:

* Discussing insides form the monthly health report
* Identifying recurrent problems or long-term maintenance priorities
* Reviewing the list of open issues
