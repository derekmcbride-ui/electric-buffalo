# Implementation Manager Dashboard
## Visual Architecture & Technical Specification

---

## Executive Overview

The **Implementation Manager (IM) Dashboard** is a centralized, net-new internal application that serves as the "Implementation Command Console" for ParentSquare. It addresses the critical challenge of fragmented implementation data and inconsistent reporting by establishing a single source of truth for implementation project status, health, and workflows.

### The Problem It Solves

Currently, Implementation Managers face:
- **Fragmented data sources** requiring manual aggregation from HubSpot, internal tools, Jira, Zendesk, and product usage systems
- **Inconsistent HubSpot data** where deal stages and fields don't reflect actual implementation reality
- **Manual synchronization burden** leading to reporting delays and data drift
- **Limited visibility** into contact coverage gaps and implementation health signals
- **No unified workflow** forcing IMs to context-switch between multiple systems daily

### The Solution

The IM Dashboard creates a unified workspace where Implementation Managers can:

1. **Manage their complete portfolio** of implementation projects in one interface
2. **Track milestone completion** and phase progression with automated HubSpot synchronization
3. **Monitor customer health** through aggregated signals from multiple systems (Jira tickets, Zendesk support, product usage, sentiment data)
4. **Ensure contact coverage** by validating that key stakeholder roles are filled
5. **Reduce manual work** by 75% through automated status updates and intelligent workflows
6. **Access real-time insights** from ParentSquare product usage data, CRO statuses, and support metrics

The dashboard implements a **hub-and-spoke architecture** where it serves as the central hub, orchestrating bidirectional data flows with HubSpot (the primary integration), while pulling supplementary signals from Jira, Zendesk, and ParentSquare analytics systems.

---

## System Architecture

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRESENTATION LAYER                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│  │  Portfolio View  │  │  Project Detail  │  │  Health Monitor  │         │
│  │                  │  │                  │  │                  │         │
│  │ • Filter by IM   │  │ • Milestones     │  │ • Risk Alerts    │         │
│  │ • Health status  │  │ • Timeline       │  │ • Score trends   │         │
│  │ • Phase filter   │  │ • Contacts       │  │ • Action items   │         │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘         │
│                                                                              │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│  │  CRO Tracker     │  │ Support Insights │  │  Usage Analytics │         │
│  │                  │  │                  │  │                  │         │
│  │ • Jira tickets   │  │ • Zendesk data   │  │ • Feature usage  │         │
│  │ • Priority view  │  │ • Ticket volume  │  │ • Engagement     │         │
│  │ • Status updates │  │ • Response time  │  │ • Completion %   │         │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        APPLICATION LAYER                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                    Core Business Logic Services                       │  │
│  ├──────────────────────────────────────────────────────────────────────┤  │
│  │                                                                       │  │
│  │  ┌─────────────────────┐  ┌──────────────────┐  ┌─────────────────┐ │  │
│  │  │  Project Service    │  │  Sync Service    │  │  Health Service │ │  │
│  │  │                     │  │                  │  │                 │ │  │
│  │  │ • CRUD operations   │  │ • HubSpot sync   │  │ • Score calc    │ │  │
│  │  │ • Phase management  │  │ • Stage mapping  │  │ • Alert logic   │ │  │
│  │  │ • Milestone tracking│  │ • Field sync     │  │ • Risk flagging │ │  │
│  │  └─────────────────────┘  └──────────────────┘  └─────────────────┘ │  │
│  │                                                                       │  │
│  │  ┌─────────────────────┐  ┌──────────────────┐  ┌─────────────────┐ │  │
│  │  │  Contact Service    │  │  Event Service   │  │  Analytics Svc  │ │  │
│  │  │                     │  │                  │  │                 │ │  │
│  │  │ • Role validation   │  │ • Webhook mgmt   │  │ • Usage queries │ │  │
│  │  │ • Coverage checks   │  │ • Event logging  │  │ • Trend analysis│ │  │
│  │  │ • Gap reporting     │  │ • Audit trail    │  │ • Reporting     │ │  │
│  │  └─────────────────────┘  └──────────────────┘  └─────────────────┘ │  │
│  │                                                                       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        INTEGRATION LAYER                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                      Integration Connectors                           │  │
│  ├──────────────────────────────────────────────────────────────────────┤  │
│  │                                                                       │  │
│  │  ┌─────────────────────┐  ┌──────────────────┐  ┌─────────────────┐ │  │
│  │  │  HubSpot Connector  │  │  Jira Connector  │  │ Zendesk Conn.   │ │  │
│  │  │  [PRIMARY]          │  │  [READ-ONLY]     │  │ [READ-ONLY]     │ │  │
│  │  │                     │  │                  │  │                 │ │  │
│  │  │ • Deals API         │  │ • CRO tracking   │  │ • Ticket API    │ │  │
│  │  │ • Contacts API      │  │ • Issue status   │  │ • Support data  │ │  │
│  │  │ • Properties API    │  │ • Priority data  │  │ • Volume trends │ │  │
│  │  │ • Webhooks (inbound)│  │                  │  │                 │ │  │
│  │  │ • Rate limiting     │  │                  │  │                 │ │  │
│  │  │ • Error handling    │  │                  │  │                 │ │  │
│  │  │ • Retry logic       │  │                  │  │                 │ │  │
│  │  └─────────────────────┘  └──────────────────┘  └─────────────────┘ │  │
│  │                                                                       │  │
│  │  ┌─────────────────────┐  ┌──────────────────┐  ┌─────────────────┐ │  │
│  │  │ParentSquare Conn.   │  │ Notification Hub │  │ Future: AI/ML   │ │  │
│  │  │[READ-ONLY]          │  │                  │  │                 │ │  │
│  │  │                     │  │ • Slack webhooks │  │ • Predictive    │ │  │
│  │  │ • Usage analytics   │  │ • Email service  │  │   health scores │ │  │
│  │  │ • Feature adoption  │  │ • Alert routing  │  │ • Smart alerts  │ │  │
│  │  │ • Completion data   │  │                  │  │ • Insights      │ │  │
│  │  └─────────────────────┘  └──────────────────┘  └─────────────────┘ │  │
│  │                                                                       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          DATA LAYER                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                    Primary Application Database                       │  │
│  │                    [SOURCE OF TRUTH]                                  │  │
│  ├──────────────────────────────────────────────────────────────────────┤  │
│  │                                                                       │  │
│  │  Core Tables:                                                         │  │
│  │  • implementation_projects (master table)                            │  │
│  │  • milestones                                                         │  │
│  │  • health_scores (historical tracking)                               │  │
│  │  • contacts_coverage                                                  │  │
│  │  • sync_log (audit trail)                                             │  │
│  │  • integration_cache (external system data)                          │  │
│  │                                                                       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────┐  ┌───────────────────────┐                      │
│  │   Cache Layer         │  │   Event Queue         │                      │
│  │   (Redis/Memcached)   │  │   (RabbitMQ/SQS)      │                      │
│  │                       │  │                       │                      │
│  │ • API response cache  │  │ • Webhook events      │                      │
│  │ • Session data        │  │ • Sync jobs           │                      │
│  │ • Rate limit tracking │  │ • Alert processing    │                      │
│  └───────────────────────┘  └───────────────────────┘                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    INFRASTRUCTURE LAYER                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│  │  Authentication  │  │    Monitoring    │  │     Logging      │         │
│  │                  │  │                  │  │                  │         │
│  │ • SSO/OAuth      │  │ • Health checks  │  │ • Audit logs     │         │
│  │ • RBAC           │  │ • Performance    │  │ • Error tracking │         │
│  │ • API keys       │  │ • Alerts         │  │ • Debug traces   │         │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Integration Architecture

### HubSpot Integration (Primary Bidirectional Sync)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     IM DASHBOARD ↔ HUBSPOT INTEGRATION                      │
└─────────────────────────────────────────────────────────────────────────────┘

HUBSPOT → IM DASHBOARD (Initial Import & Selective Updates)
═══════════════════════════════════════════════════════════
┌──────────────┐                                    ┌─────────────────┐
│   HubSpot    │────── Webhook Triggers ───────────▶│  IM Dashboard   │
│              │                                    │                 │
│  • Deals     │◀──── Poll for New Deals ──────────│  • Listens for: │
│  • Contacts  │         (scheduled job)            │    - New deals  │
│  • Properties│                                    │    - Owner chg  │
└──────────────┘                                    │    - Close date │
                                                    └─────────────────┘

Data Imported from HubSpot:
───────────────────────────
• Deal ID (primary key for sync)
• Deal Name / Account Name
• Deal Amount
• Close Date
• Deal Stage (read-only reference)
• Deal Owner → mapped to IM Owner
• Associated Contacts → for coverage validation
• Product Type (from custom properties)


IM DASHBOARD → HUBSPOT (Continuous Real-Time Sync)
═══════════════════════════════════════════════════
┌─────────────────┐         Events          ┌──────────────────┐
│  IM Dashboard   │                         │   Sync Service   │
│                 │────────────────────────▶│                  │
│ IM Updates:     │                         │ Transforms &     │
│ • Phase change  │                         │ Validates data   │
│ • Health update │                         │                  │
│ • Milestone     │                         │ Handles:         │
│   completion    │                         │ • Rate limiting  │
│ • Go-live date  │                         │ • Retry logic    │
│ • Contact gaps  │                         │ • Error recovery │
└─────────────────┘                         └──────────────────┘
                                                      │
                                                      ▼
                                            ┌──────────────────┐
                                            │   HubSpot API    │
                                            │                  │
                                            │ Updates:         │
                                            │ • Deal Properties│
                                            │ • Deal Stage     │
                                            │   (conditional)  │
                                            │ • Activity notes │
                                            └──────────────────┘


FIELD MAPPING: IM DASHBOARD → HUBSPOT PROPERTIES
═════════════════════════════════════════════════

┌─────────────────────────────┬─────────────────────────────┬───────────────┐
│   IM Dashboard Field        │   HubSpot Custom Property   │   Data Type   │
├─────────────────────────────┼─────────────────────────────┼───────────────┤
│ implementation_phase        │ im_phase                    │ Enumeration   │
│                             │   • Kickoff                 │               │
│                             │   • In Progress             │               │
│                             │   • Launched                │               │
│                             │   • Stabilizing             │               │
│                             │   • Completed               │               │
├─────────────────────────────┼─────────────────────────────┼───────────────┤
│ health_status               │ im_health_status            │ Enumeration   │
│                             │   • Green                   │               │
│                             │   • Yellow                  │               │
│                             │   • Red                     │               │
├─────────────────────────────┼─────────────────────────────┼───────────────┤
│ target_go_live_date         │ im_target_go_live_date      │ Date          │
├─────────────────────────────┼─────────────────────────────┼───────────────┤
│ last_im_update_timestamp    │ last_im_update_date         │ DateTime      │
├─────────────────────────────┼─────────────────────────────┼───────────────┤
│ health_override_reason      │ im_health_reason            │ Text          │
├─────────────────────────────┼─────────────────────────────┼───────────────┤
│ days_since_last_update      │ im_days_since_update        │ Number        │
├─────────────────────────────┼─────────────────────────────┼───────────────┤
│ contact_coverage_status     │ im_contact_coverage         │ Enumeration   │
│                             │   • Complete                │               │
│                             │   • Partial                 │               │
│                             │   • Incomplete              │               │
├─────────────────────────────┼─────────────────────────────┼───────────────┤
│ missing_contact_roles       │ im_missing_roles            │ Text (list)   │
└─────────────────────────────┴─────────────────────────────┴───────────────┘


STAGE PROGRESSION AUTOMATION (Conditional v1)
══════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────┐
│  Phase 1: CONSERVATIVE Stage Updates (2-3 key transitions only)        │
└─────────────────────────────────────────────────────────────────────────┘

Trigger Conditions:
───────────────────
IF implementation_phase = "Launched"
   AND kickoff_complete = true
   AND go_live_complete = true
   THEN update HubSpot Deal Stage to "Live" (or agreed equivalent)

IF implementation_phase = "Completed"
   AND stabilizing_complete = true
   THEN update HubSpot Deal Stage to "Implemented" (or agreed equivalent)

Safety Rails:
─────────────
• Manual override available in dashboard
• Sync log records all stage changes
• Rollback capability for incorrect transitions
• Alert to IM/RevOps team on stage change


ACTIVITY LOGGING TO HUBSPOT
════════════════════════════

Automatically create HubSpot notes/activities when:
───────────────────────────────────────────────────
• Health status changes (especially Green → Yellow/Red)
• Implementation phase advances
• Milestones marked complete
• Contact gaps identified
• Manual status update by IM (with notes)

Note Format:
────────────
[IM Dashboard Update - {timestamp}]
Phase: {phase}
Health: {health_status}
IM Notes: {user_notes}
Updated by: {im_owner_name}
```

---

## Data Flow Architecture

### Primary Data Flows

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     DATA FLOW: IMPLEMENTATION LIFECYCLE                      │
└─────────────────────────────────────────────────────────────────────────────┘

1. PROJECT INITIALIZATION
──────────────────────────
    ┌──────────────┐
    │   HubSpot    │  New Deal Created
    │              │  (Won/Closed-Won)
    └───────┬──────┘
            │
            ▼
    ┌──────────────┐
    │   Webhook    │  Trigger: Deal Stage = "Closed Won"
    │   Listener   │
    └───────┬──────┘
            │
            ▼
    ┌──────────────────────┐
    │  IM Dashboard        │  Creates Implementation Project
    │  Project Service     │  • Imports deal data
    └───────┬──────────────┘  • Assigns IM owner
            │                 • Initializes milestones
            │                 • Sets initial phase = "Kickoff"
            ▼
    ┌──────────────────────┐
    │  Database            │  New record in
    │  implementation_     │  implementation_projects table
    │  projects            │
    └──────────────────────┘


2. DAILY IM WORKFLOW
────────────────────
    ┌──────────────────────┐
    │  IM Dashboard UI     │  IM logs in, views portfolio
    │  Portfolio View      │
    └───────┬──────────────┘
            │
            ▼
    ┌──────────────────────┐
    │  Project Detail      │  IM selects project,
    │  Page                │  sees milestone checklist
    └───────┬──────────────┘
            │
            ▼
    ┌──────────────────────┐
    │  IM Actions:         │
    │  • Checks milestone  │──┐
    │  • Updates health    │  │
    │  • Adds notes        │  │
    │  • Changes phase     │  │
    └──────────────────────┘  │
                              │
            ┌─────────────────┘
            ▼
    ┌──────────────────────┐
    │  Business Logic      │  Validates changes
    │  Project Service     │  Calculates health score
    └───────┬──────────────┘  Determines if stage should change
            │
            ├────────────┬────────────┐
            ▼            ▼            ▼
    ┌──────────┐  ┌──────────┐  ┌──────────┐
    │ Database │  │   Sync   │  │  Alert   │
    │  Update  │  │ Service  │  │ Service  │
    └──────────┘  └─────┬────┘  └─────┬────┘
                        │             │
                        ▼             ▼
                  ┌──────────┐  ┌──────────┐
                  │ HubSpot  │  │  Slack/  │
                  │  Update  │  │  Email   │
                  └──────────┘  └──────────┘


3. HEALTH SCORE CALCULATION (Multi-System)
───────────────────────────────────────────
    ┌──────────────────────────────────────────────────────────┐
    │              HEALTH SCORE INPUTS (Weighted)              │
    └──────────────────────────────────────────────────────────┘
            │
            ├─────────────────┬─────────────────┬─────────────────┐
            ▼                 ▼                 ▼                 ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
    │ParentSquare  │  │    Jira      │  │   Zendesk    │  │   Manual     │
    │   Usage      │  │   CROs       │  │   Tickets    │  │   IM Input   │
    │              │  │              │  │              │  │              │
    │ • Login %    │  │ • Open bugs  │  │ • Volume     │  │ • Delays     │
    │ • Feature    │  │ • Priority   │  │ • Severity   │  │ • Blockers   │
    │   adoption   │  │ • Age        │  │ • Response   │  │ • Sentiment  │
    │ • Data load  │  │              │  │   time       │  │              │
    └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
           │                 │                 │                 │
           └─────────────────┴─────────────────┴─────────────────┘
                             │
                             ▼
                   ┌──────────────────┐
                   │  Health Service  │  Algorithm:
                   │                  │  • Weights each signal
                   │  Calculation:    │  • Applies rules
                   │  Score = f(...)  │  • Suggests G/Y/R
                   └─────────┬────────┘
                             │
                             ▼
                   ┌──────────────────┐
                   │  IM Review &     │  IM can override
                   │  Override        │  with reason
                   └─────────┬────────┘
                             │
                             ▼
                   ┌──────────────────┐
                   │  Final Health    │  Synced to HubSpot
                   │  Status Set      │  + alerts triggered
                   └──────────────────┘


4. CONTACT COVERAGE VALIDATION
───────────────────────────────
    ┌──────────────────────────────────────────────────────────┐
    │              REQUIRED ROLES (per implementation)          │
    └──────────────────────────────────────────────────────────┘
    │  1. District Admin / Executive Sponsor
    │  2. SIS Owner / Data Lead
    │  3. Communications / Parent Engagement Lead
    │  4. Technical Admin / IT Contact
    │  5. Training Coordinator
    └──────────────────────────────────────────────────────────┘
            │
            ▼
    ┌──────────────────────┐
    │  HubSpot Contacts    │  Query associated contacts
    │  API Query           │  for implementation deal
    └───────┬──────────────┘
            │
            ▼
    ┌──────────────────────┐
    │  Contact Service     │  Match contacts to required roles
    │  Role Mapping        │  based on HubSpot contact properties
    └───────┬──────────────┘
            │
            ├─────────────────┬─────────────────┐
            ▼                 ▼                 ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
    │  Coverage    │  │   Gaps       │  │  Dashboard   │
    │  Complete    │  │  Identified  │  │   Display    │
    │              │  │              │  │              │
    │ Green flag   │  │ Red flag +   │  │ • Visual     │
    │              │  │ List missing │  │   indicator  │
    │              │  │ roles        │  │ • Action btn │
    └──────────────┘  └──────┬───────┘  └──────────────┘
                             │
                             ▼
                      ┌──────────────┐
                      │  Alert to IM │
                      │  + Task to   │
                      │  CSM/Sales   │
                      └──────────────┘
```

---

## Component Architecture

### Frontend Components

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FRONTEND COMPONENT TREE                              │
└─────────────────────────────────────────────────────────────────────────────┘

App
├── Authentication
│   ├── LoginPage
│   └── AuthProvider (SSO/OAuth integration)
│
├── Layout
│   ├── Header
│   │   ├── UserProfile
│   │   ├── Notifications
│   │   └── QuickSearch
│   ├── Sidebar Navigation
│   │   ├── Portfolio
│   │   ├── Health Monitor
│   │   ├── Reports
│   │   └── Settings
│   └── Footer
│
├── Portfolio View ★ (Main Landing Page)
│   ├── FilterBar
│   │   ├── IMOwnerFilter (multi-select)
│   │   ├── PhaseFilter (multi-select)
│   │   ├── HealthFilter (multi-select)
│   │   ├── GoLiveDateRangeFilter
│   │   └── SearchByAccount
│   │
│   ├── ProjectsTable
│   │   ├── TableHeader (sortable columns)
│   │   └── ProjectRow (foreach project)
│   │       ├── AccountNameCell (+ link to detail)
│   │       ├── ProductTypeCell
│   │       ├── IMOwnerCell
│   │       ├── PhaseCell (with status badge)
│   │       ├── HealthCell (G/Y/R icon)
│   │       ├── GoLiveDateCell
│   │       ├── DaysSinceUpdateCell
│   │       └── QuickActionsCell (context menu)
│   │
│   └── SummaryStats
│       ├── TotalProjectsCount
│       ├── AtRiskCount (Red health)
│       ├── LaunchingSoonCount (30 days)
│       └── MyPortfolioCount (current user)
│
├── Project Detail View ★
│   ├── ProjectHeader
│   │   ├── AccountInfo
│   │   │   ├── AccountName
│   │   │   ├── ProductType
│   │   │   ├── HubSpotDealLink (external)
│   │   │   └── DealAmount
│   │   ├── OwnerInfo
│   │   │   ├── IMOwner
│   │   │   └── CSMOwner (from HubSpot)
│   │   └── KeyDates
│   │       ├── CloseDate
│   │       ├── TargetGoLive (editable)
│   │       └── ActualGoLive
│   │
│   ├── PhaseProgressBar
│   │   └── PhaseSteps (visual stepper)
│   │       ├── Kickoff
│   │       ├── In Progress
│   │       ├── Launched
│   │       ├── Stabilizing
│   │       └── Completed
│   │
│   ├── MilestonesPanel ★
│   │   ├── MilestoneChecklist
│   │   │   ├── KickoffMilestone
│   │   │   │   ├── CheckboxInput
│   │   │   │   ├── CompletionDate
│   │   │   │   └── Notes
│   │   │   ├── DataImportMilestone
│   │   │   ├── SISIntegrationMilestone
│   │   │   ├── StaffTrainingMilestone
│   │   │   ├── SoftLaunchMilestone
│   │   │   └── GoLiveMilestone
│   │   └── AutoProgressButton (advances phase when criteria met)
│   │
│   ├── HealthPanel ★
│   │   ├── CurrentHealthDisplay (large visual indicator)
│   │   ├── SystemSuggestedHealth
│   │   │   ├── Score breakdown
│   │   │   └── Contributing factors
│   │   ├── ManualOverride
│   │   │   ├── OverrideToggle
│   │   │   ├── ReasonTextArea (required)
│   │   │   └── SaveButton
│   │   ├── HealthHistory (trend chart)
│   │   └── AlertConfiguration
│   │
│   ├── ContactCoveragePanel
│   │   ├── CoverageStatus (overall indicator)
│   │   ├── RequiredRolesList
│   │   │   └── RoleRow (foreach role)
│   │   │       ├── RoleName
│   │   │       ├── AssignedContact (from HubSpot)
│   │   │       ├── StatusIcon (filled/missing)
│   │   │       └── ActionButton (add contact)
│   │   └── MissingRolesAlert
│   │
│   ├── IntegrationsTabset
│   │   ├── HubSpotTab
│   │   │   ├── DealStageDisplay (read-only)
│   │   │   ├── KeyFieldsSnapshot
│   │   │   └── SyncStatus
│   │   ├── ParentSquareTab
│   │   │   ├── UsageMetrics
│   │   │   │   ├── LoginRate
│   │   │   │   ├── FeatureAdoption
│   │   │   │   └── DataLoadProgress
│   │   │   └── CompletionCriteria (for milestones)
│   │   ├── JiraTab (CROs)
│   │   │   ├── OpenCROsList
│   │   │   ├── PriorityDistribution
│   │   │   └── AgingBugs
│   │   └── ZendeskTab
│   │       ├── TicketVolume (last 30 days)
│   │       ├── HighPriorityTickets
│   │       └── ResponseTimeMetrics
│   │
│   ├── TimelinePanel
│   │   └── ActivityFeed (chronological)
│   │       ├── PhaseChanges
│   │       ├── MilestoneCompletions
│   │       ├── HealthChanges
│   │       ├── SystemAlerts
│   │       └── ManualNotes
│   │
│   └── ActionBar
│       ├── SaveButton
│       ├── SyncNowButton (force sync to HubSpot)
│       └── MoreActionsMenu
│
├── Health Monitor Dashboard
│   ├── AtRiskProjects (Red health)
│   ├── WatchlistProjects (Yellow health)
│   ├── HealthTrendsChart (all projects over time)
│   └── RecommendedActions (AI-driven suggestions - future)
│
├── Reports & Analytics
│   ├── PhaseDistribution
│   ├── TimeToLaunchMetrics
│   ├── HealthScoreDistribution
│   ├── ContactCoverageReport
│   └── ExportButton (CSV/PDF)
│
└── Settings & Admin
    ├── UserManagement
    ├── IntegrationSettings
    │   ├── HubSpotConfig
    │   ├── JiraConfig
    │   ├── ZendeskConfig
    │   └── ParentSquareConfig
    ├── MilestoneTemplates
    ├── AlertRules
    └── AuditLog
```

### Backend Services Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      BACKEND SERVICES (Microservices Pattern)                │
└─────────────────────────────────────────────────────────────────────────────┘

API Gateway / Router
├── Authentication Middleware
├── Authorization Middleware (RBAC)
├── Rate Limiting
├── Request Validation
└── Error Handling

Core Services
│
├── Project Service
│   ├── Responsibilities:
│   │   ├── CRUD operations for implementation projects
│   │   ├── Phase management and transitions
│   │   ├── Milestone tracking and validation
│   │   └── Project portfolio queries
│   ├── Endpoints:
│   │   ├── GET    /api/projects (list with filters)
│   │   ├── GET    /api/projects/{id} (detail)
│   │   ├── POST   /api/projects (create)
│   │   ├── PATCH  /api/projects/{id} (update)
│   │   ├── POST   /api/projects/{id}/milestones (mark complete)
│   │   └── POST   /api/projects/{id}/phase (advance phase)
│   └── Dependencies:
│       ├── Database (implementation_projects table)
│       ├── Sync Service (trigger HubSpot updates)
│       └── Event Service (log changes)
│
├── Health Service
│   ├── Responsibilities:
│   │   ├── Calculate health scores from multiple inputs
│   │   ├── Apply weighting rules and algorithms
│   │   ├── Track health history over time
│   │   ├── Generate health alerts
│   │   └── Support manual overrides with audit trail
│   ├── Endpoints:
│   │   ├── GET    /api/health/{project_id} (current status)
│   │   ├── GET    /api/health/{project_id}/history (trend)
│   │   ├── POST   /api/health/{project_id}/calculate (refresh)
│   │   ├── POST   /api/health/{project_id}/override (manual set)
│   │   └── GET    /api/health/at-risk (query projects)
│   ├── Health Score Algorithm (v1 simple):
│   │   ├── Input: Days behind schedule (if target date passed)
│   │   ├── Input: Open Jira CROs (weighted by priority)
│   │   ├── Input: Zendesk ticket volume (last 30 days)
│   │   ├── Input: ParentSquare usage metrics (below threshold)
│   │   ├── Input: Manual IM flags
│   │   └── Output: Green (no issues) / Yellow (1-2 flags) / Red (3+ flags)
│   └── Dependencies:
│       ├── Jira Connector (CRO data)
│       ├── Zendesk Connector (ticket data)
│       ├── ParentSquare Connector (usage data)
│       └── Database (health_scores table)
│
├── Sync Service ★ (Critical Path)
│   ├── Responsibilities:
│   │   ├── Bidirectional sync with HubSpot
│   │   ├── Field mapping and transformation
│   │   ├── Conditional stage updates
│   │   ├── Activity/note logging to HubSpot
│   │   ├── Error handling and retry logic
│   │   └── Sync audit trail
│   ├── Endpoints:
│   │   ├── POST   /api/sync/hubspot/trigger (manual sync)
│   │   ├── POST   /api/sync/hubspot/import-deal (pull from HubSpot)
│   │   ├── GET    /api/sync/status/{project_id} (sync history)
│   │   └── POST   /api/sync/hubspot/webhook (receive HubSpot events)
│   ├── Sync Rules Engine:
│   │   ├── Field change detection (diff)
│   │   ├── Mapping rules (IM field → HubSpot property)
│   │   ├── Stage progression rules (conditional)
│   │   ├── Conflict resolution (last-write-wins with timestamp)
│   │   └── Rollback capability
│   ├── Background Jobs:
│   │   ├── Poll new deals (every 15 min)
│   │   ├── Batch sync stale records (nightly)
│   │   └── Retry failed syncs (exponential backoff)
│   └── Dependencies:
│       ├── HubSpot Connector
│       ├── Event Queue (async processing)
│       └── Database (sync_log table)
│
├── Contact Service
│   ├── Responsibilities:
│   │   ├── Validate contact coverage for required roles
│   │   ├── Query HubSpot for associated contacts
│   │   ├── Role mapping and gap detection
│   │   └── Generate coverage reports
│   ├── Endpoints:
│   │   ├── GET    /api/contacts/{project_id}/coverage (summary)
│   │   ├── GET    /api/contacts/{project_id}/gaps (missing roles)
│   │   ├── POST   /api/contacts/{project_id}/validate (run check)
│   │   └── PATCH  /api/contacts/{project_id}/roles (update mapping)
│   └── Dependencies:
│       ├── HubSpot Connector (contacts API)
│       └── Database (contacts_coverage table)
│
├── Analytics Service
│   ├── Responsibilities:
│   │   ├── Query ParentSquare usage data
│   │   ├── Aggregate metrics for health scoring
│   │   ├── Generate reports and dashboards
│   │   └── Export functionality
│   ├── Endpoints:
│   │   ├── GET    /api/analytics/usage/{project_id}
│   │   ├── GET    /api/analytics/portfolio (aggregates)
│   │   ├── GET    /api/analytics/reports/{type}
│   │   └── POST   /api/analytics/export
│   └── Dependencies:
│       ├── ParentSquare Connector
│       └── Database (read replicas for reporting)
│
├── Event Service
│   ├── Responsibilities:
│   │   ├── Webhook management (inbound from HubSpot)
│   │   ├── Event logging and audit trail
│   │   ├── Event-driven workflows
│   │   └── Activity timeline generation
│   ├── Endpoints:
│   │   ├── POST   /api/events/webhook/{source} (receive webhooks)
│   │   ├── GET    /api/events/{project_id} (activity feed)
│   │   └── GET    /api/events/audit (admin view)
│   └── Dependencies:
│       ├── Event Queue
│       └── Database (events/audit tables)
│
└── Notification Service
    ├── Responsibilities:
    │   ├── Send Slack alerts
    │   ├── Send email notifications
    │   ├── Alert routing based on rules
    │   └── Notification preferences management
    ├── Endpoints:
    │   ├── POST   /api/notifications/send
    │   ├── GET    /api/notifications/preferences
    │   └── PATCH  /api/notifications/preferences
    └── Dependencies:
        ├── Slack API
        ├── Email service (SendGrid/SES)
        └── Database (notification_preferences)
```

---

## Database Schema

### Core Tables

```sql
-- ============================================================================
-- IMPLEMENTATION PROJECTS (Source of Truth)
-- ============================================================================
CREATE TABLE implementation_projects (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- HubSpot linkage
    hubspot_deal_id             VARCHAR(255) UNIQUE NOT NULL,

    -- Core identifiers
    account_name                VARCHAR(255) NOT NULL,
    district_org_id             VARCHAR(255),  -- ParentSquare internal ID
    product_type                VARCHAR(100) NOT NULL,  -- Comms, Remind, SmartSites

    -- Ownership
    im_owner_id                 UUID NOT NULL,  -- FK to users table
    im_owner_name               VARCHAR(255) NOT NULL,
    csm_owner_name              VARCHAR(255),  -- From HubSpot (read-only)

    -- Phase and status (SOURCE OF TRUTH)
    implementation_phase        VARCHAR(50) NOT NULL
                                  CHECK (implementation_phase IN
                                    ('Kickoff', 'In Progress', 'Launched',
                                     'Stabilizing', 'Completed')),
    health_status               VARCHAR(20) NOT NULL
                                  CHECK (health_status IN
                                    ('Green', 'Yellow', 'Red')),
    health_override             BOOLEAN DEFAULT FALSE,
    health_override_reason      TEXT,
    system_suggested_health     VARCHAR(20),  -- Before override

    -- Dates
    deal_close_date             DATE,
    target_go_live_date         DATE,
    actual_go_live_date         DATE,
    kickoff_date                DATE,

    -- Milestones (v1 boolean flags)
    milestone_kickoff_complete      BOOLEAN DEFAULT FALSE,
    milestone_kickoff_date          DATE,
    milestone_data_import_complete  BOOLEAN DEFAULT FALSE,
    milestone_data_import_date      DATE,
    milestone_sis_integration_complete BOOLEAN DEFAULT FALSE,
    milestone_sis_integration_date  DATE,
    milestone_staff_training_complete BOOLEAN DEFAULT FALSE,
    milestone_staff_training_date   DATE,
    milestone_soft_launch_complete  BOOLEAN DEFAULT FALSE,
    milestone_soft_launch_date      DATE,
    milestone_go_live_complete      BOOLEAN DEFAULT FALSE,
    milestone_go_live_date          DATE,

    -- HubSpot snapshot (cached, read-only reference)
    hubspot_deal_stage          VARCHAR(100),
    hubspot_deal_amount         DECIMAL(15,2),

    -- Tracking
    last_im_update_date         TIMESTAMP WITH TIME ZONE,
    days_since_last_update      INTEGER GENERATED ALWAYS AS
                                  (EXTRACT(DAY FROM NOW() - last_im_update_date)) STORED,
    status_notes                TEXT,

    -- Audit
    created_at                  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at                  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_by                  UUID,
    updated_by                  UUID,

    -- Indexes
    INDEX idx_im_owner (im_owner_id),
    INDEX idx_phase (implementation_phase),
    INDEX idx_health (health_status),
    INDEX idx_go_live_date (target_go_live_date),
    INDEX idx_hubspot_deal (hubspot_deal_id)
);


-- ============================================================================
-- HEALTH SCORES (Historical tracking)
-- ============================================================================
CREATE TABLE health_scores (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id                  UUID NOT NULL REFERENCES implementation_projects(id) ON DELETE CASCADE,

    -- Score calculation
    calculated_health           VARCHAR(20) NOT NULL,
    score_components            JSONB,  -- Breakdown of contributing factors
    /*
    Example score_components:
    {
      "days_behind_schedule": 5,
      "open_critical_cros": 2,
      "ticket_volume_high": true,
      "usage_below_threshold": false,
      "manual_flag": true
    }
    */

    -- Inputs snapshot (for historical analysis)
    jira_open_cros              INTEGER,
    jira_critical_cros          INTEGER,
    zendesk_tickets_30d         INTEGER,
    zendesk_high_priority       INTEGER,
    parentsquare_login_rate     DECIMAL(5,2),  -- percentage
    parentsquare_feature_adoption DECIMAL(5,2),
    days_behind_schedule        INTEGER,

    -- Metadata
    calculated_at               TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    calculation_version         VARCHAR(20),  -- Algorithm version for audit

    INDEX idx_project (project_id),
    INDEX idx_calculated_at (calculated_at)
);


-- ============================================================================
-- CONTACTS COVERAGE
-- ============================================================================
CREATE TABLE contacts_coverage (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id                  UUID NOT NULL REFERENCES implementation_projects(id) ON DELETE CASCADE,

    -- Required role
    role_name                   VARCHAR(100) NOT NULL,
    /*
      Standard roles:
      - District Admin / Executive Sponsor
      - SIS Owner / Data Lead
      - Communications / Parent Engagement Lead
      - Technical Admin / IT Contact
      - Training Coordinator
    */

    -- Contact info (from HubSpot)
    hubspot_contact_id          VARCHAR(255),
    contact_name                VARCHAR(255),
    contact_email               VARCHAR(255),
    contact_phone               VARCHAR(50),

    -- Status
    is_filled                   BOOLEAN DEFAULT FALSE,
    identified_date             TIMESTAMP WITH TIME ZONE,

    -- Audit
    created_at                  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at                  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    UNIQUE(project_id, role_name),
    INDEX idx_project (project_id),
    INDEX idx_filled (is_filled)
);


-- ============================================================================
-- SYNC LOG (Audit trail for HubSpot synchronization)
-- ============================================================================
CREATE TABLE sync_log (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id                  UUID REFERENCES implementation_projects(id) ON DELETE SET NULL,

    -- Sync metadata
    sync_direction              VARCHAR(20) NOT NULL
                                  CHECK (sync_direction IN ('TO_HUBSPOT', 'FROM_HUBSPOT')),
    sync_type                   VARCHAR(50) NOT NULL,
                                  -- 'FIELD_UPDATE', 'STAGE_CHANGE', 'ACTIVITY_LOG', 'INITIAL_IMPORT'

    -- What changed
    fields_changed              JSONB,  -- List of fields that were updated
    old_values                  JSONB,  -- Before state
    new_values                  JSONB,  -- After state

    -- HubSpot specifics
    hubspot_deal_id             VARCHAR(255),
    hubspot_api_response        JSONB,  -- Full API response for debugging

    -- Result
    status                      VARCHAR(20) NOT NULL
                                  CHECK (status IN ('SUCCESS', 'FAILED', 'RETRYING', 'SKIPPED')),
    error_message               TEXT,
    retry_count                 INTEGER DEFAULT 0,

    -- Timing
    synced_at                   TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    duration_ms                 INTEGER,

    INDEX idx_project (project_id),
    INDEX idx_status (status),
    INDEX idx_synced_at (synced_at),
    INDEX idx_hubspot_deal (hubspot_deal_id)
);


-- ============================================================================
-- EVENTS / ACTIVITY FEED
-- ============================================================================
CREATE TABLE events (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id                  UUID REFERENCES implementation_projects(id) ON DELETE CASCADE,

    -- Event details
    event_type                  VARCHAR(100) NOT NULL,
    /*
      Event types:
      - PROJECT_CREATED
      - PHASE_CHANGED
      - MILESTONE_COMPLETED
      - HEALTH_CHANGED
      - HEALTH_OVERRIDDEN
      - CONTACT_GAP_IDENTIFIED
      - SYNC_COMPLETED
      - MANUAL_NOTE_ADDED
    */
    event_source                VARCHAR(50),  -- 'USER', 'SYSTEM', 'WEBHOOK', 'SCHEDULED_JOB'

    -- Event data
    event_data                  JSONB,  -- Flexible payload
    summary                     TEXT,   -- Human-readable description

    -- Actor
    user_id                     UUID,  -- FK to users table (if user-triggered)
    user_name                   VARCHAR(255),

    -- Timing
    occurred_at                 TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    INDEX idx_project (project_id),
    INDEX idx_event_type (event_type),
    INDEX idx_occurred_at (occurred_at)
);


-- ============================================================================
-- INTEGRATION CACHE (External system data)
-- ============================================================================
CREATE TABLE integration_cache (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id                  UUID REFERENCES implementation_projects(id) ON DELETE CASCADE,

    -- Source system
    integration_source          VARCHAR(50) NOT NULL,  -- 'JIRA', 'ZENDESK', 'PARENTSQUARE'

    -- Cached data
    cache_key                   VARCHAR(255) NOT NULL,  -- e.g., 'open_cros', 'tickets_30d'
    cache_value                 JSONB NOT NULL,

    -- Cache metadata
    cached_at                   TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at                  TIMESTAMP WITH TIME ZONE,
    is_stale                    BOOLEAN GENERATED ALWAYS AS
                                  (expires_at < NOW()) STORED,

    UNIQUE(project_id, integration_source, cache_key),
    INDEX idx_project (project_id),
    INDEX idx_integration (integration_source),
    INDEX idx_expires (expires_at)
);


-- ============================================================================
-- USERS (IM owners and admin users)
-- ============================================================================
CREATE TABLE users (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email                       VARCHAR(255) UNIQUE NOT NULL,
    full_name                   VARCHAR(255) NOT NULL,
    role                        VARCHAR(50) NOT NULL,  -- 'IM', 'ADMIN', 'READONLY'
    is_active                   BOOLEAN DEFAULT TRUE,

    -- Notification preferences
    notification_preferences    JSONB DEFAULT '{"slack": true, "email": true}',
    slack_user_id               VARCHAR(100),

    -- Audit
    created_at                  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login_at               TIMESTAMP WITH TIME ZONE,

    INDEX idx_email (email),
    INDEX idx_role (role)
);


-- ============================================================================
-- CONFIGURATION (System settings and integration configs)
-- ============================================================================
CREATE TABLE configuration (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    config_key                  VARCHAR(255) UNIQUE NOT NULL,
    config_value                JSONB NOT NULL,
    description                 TEXT,
    is_sensitive                BOOLEAN DEFAULT FALSE,  -- Encrypted values
    updated_at                  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_by                  UUID REFERENCES users(id)
);
```

---

## Technical Stack Recommendations

### Frontend Technology Stack

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FRONTEND STACK (Recommended)                         │
└─────────────────────────────────────────────────────────────────────────────┘

Framework:
──────────
• React 18+ with TypeScript
  Rationale: Industry standard, large ecosystem, excellent for complex UIs
  Alternative: Vue 3 if already standardized at ParentSquare

State Management:
─────────────────
• Redux Toolkit (for global state)
  - Implementation projects list
  - User session and preferences
  - Cached integration data
• React Query / TanStack Query (for server state)
  - API data fetching and caching
  - Automatic refetching and invalidation
  - Optimistic updates

UI Component Library:
─────────────────────
• Material-UI (MUI) or Ant Design
  Rationale: Comprehensive components, accessibility, customizable theming
  Key components needed: DataGrid, DatePicker, Select, Modal, Alert, Stepper

Data Visualization:
───────────────────
• Recharts or Chart.js
  For health trends, phase distribution, analytics dashboards

Form Management:
────────────────
• React Hook Form + Zod (validation)
  For milestone forms, health override forms, settings

Styling:
────────
• CSS-in-JS (Emotion - comes with MUI) or Tailwind CSS
  For custom styling beyond component library

Build Tool:
───────────
• Vite (modern, fast) or Create React App / Next.js
  Depending on existing ParentSquare standards

Testing:
────────
• Jest + React Testing Library (unit/integration)
• Playwright or Cypress (E2E)
```

### Backend Technology Stack

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         BACKEND STACK (Recommended)                          │
└─────────────────────────────────────────────────────────────────────────────┘

Primary Options (choose one based on existing standards):

Option A: Node.js/TypeScript Stack
──────────────────────────────────
Framework: Express.js or NestJS (preferred for structure)
Language: TypeScript
ORM: Prisma or TypeORM
Validation: Zod or Joi
Testing: Jest + Supertest

Pros:
• Shared language with frontend (TypeScript)
• Large ecosystem for integrations
• Fast development velocity
• Good for serverless deployment

Cons:
• Less robust for heavy computational workloads


Option B: Python Stack
──────────────────────
Framework: FastAPI or Django + DRF
Language: Python 3.10+
ORM: SQLAlchemy or Django ORM
Validation: Pydantic (built into FastAPI)
Testing: pytest

Pros:
• Excellent for data processing and ML (future AI features)
• Strong async support (FastAPI)
• Great for integrations (many API clients available)
• Type hints with Pydantic

Cons:
• Different language from frontend


Option C: Go Stack
──────────────────
Framework: Gin or Echo
Language: Go 1.21+
ORM: GORM or sqlx
Testing: Go testing + testify

Pros:
• Excellent performance
• Strong concurrency for sync operations
• Static typing and compilation
• Good for microservices

Cons:
• Smaller ecosystem for rapid development
• Steeper learning curve

RECOMMENDATION: Node.js/TypeScript with NestJS
──────────────────────────────────────────────
Rationale:
• Shared TypeScript across stack = code reuse
• NestJS provides structure similar to backend frameworks (modules, DI)
• Rich integration library ecosystem (HubSpot, Jira, Zendesk clients)
• Easy to find developers
• Good balance of performance and development speed
```

### Database

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              DATABASE                                        │
└─────────────────────────────────────────────────────────────────────────────┘

Primary Database:
─────────────────
• PostgreSQL 15+
  Rationale:
  - JSONB support for flexible fields (integration cache, event data)
  - Strong ACID compliance for critical sync operations
  - Excellent JSON querying capabilities
  - Full-text search (for notes, activity feed)
  - Robust ecosystem and extensions
  - Supports both relational and document patterns

Hosting Options:
────────────────
• AWS RDS PostgreSQL (managed, auto-backups, read replicas)
• Google Cloud SQL
• Azure Database for PostgreSQL
• Supabase (if wanting more managed features)

Cache Layer:
────────────
• Redis 7+
  Use cases:
  - API response caching (HubSpot data)
  - Session storage
  - Rate limit tracking
  - Job queue (via BullMQ if Node.js)

Alternative: Memcached (simpler, but less features)

Event Queue:
────────────
• Option A: Redis + BullMQ (Node.js)
• Option B: RabbitMQ (language-agnostic)
• Option C: AWS SQS + SNS (serverless)

Recommendation: Redis + BullMQ
- Simplifies stack (Redis already needed for caching)
- Built-in retry, scheduling, rate limiting
- Web UI for monitoring
```

### Infrastructure & Deployment

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      INFRASTRUCTURE & DEPLOYMENT                             │
└─────────────────────────────────────────────────────────────────────────────┘

Deployment Architecture:
────────────────────────
Recommendation: Containerized deployment on cloud platform

Option A: AWS (Full Suite)
───────────────────────────
Frontend:
• S3 + CloudFront (static hosting with CDN)
  OR
• Amplify Hosting (includes CI/CD)

Backend:
• ECS Fargate (serverless containers) or EKS (if Kubernetes experience)
• Application Load Balancer

Database:
• RDS PostgreSQL (Multi-AZ for HA)
• ElastiCache Redis

Jobs/Workers:
• ECS Fargate tasks (for background sync jobs)
• Lambda (for webhooks, event processing)

Monitoring:
• CloudWatch (logs, metrics, alarms)
• AWS X-Ray (distributed tracing)


Option B: Platform-as-a-Service (Simpler)
──────────────────────────────────────────
• Heroku, Render, or Railway
  Pros: Faster setup, less infrastructure management
  Cons: Less control, potentially higher costs at scale


Option C: Kubernetes (Advanced)
────────────────────────────────
• GKE, EKS, or AKS
• Helm charts for deployments
  Only recommended if existing Kubernetes expertise


RECOMMENDATION: AWS ECS Fargate + RDS
─────────────────────────────────────
Rationale:
• Balance of control and managed services
• Serverless containers = no server management
• Scales automatically
• Cost-effective for internal tools
• Strong integration with AWS services


CI/CD Pipeline:
───────────────
Tool: GitHub Actions or GitLab CI

Pipeline stages:
1. Lint & Format Check
2. Unit Tests
3. Integration Tests
4. Build Docker Images
5. Push to Container Registry (ECR)
6. Deploy to Staging
7. E2E Tests (Staging)
8. Manual Approval
9. Deploy to Production
10. Post-deployment smoke tests


Environments:
─────────────
• Development (local Docker Compose)
• Staging (mirrors production)
• Production

Infrastructure as Code:
───────────────────────
• Terraform or AWS CDK
  For: VPC, ECS clusters, RDS, Redis, IAM roles, etc.
```

### Integration Approaches

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       INTEGRATION PATTERNS & APPROACHES                      │
└─────────────────────────────────────────────────────────────────────────────┘

HubSpot Integration (PRIMARY - Bidirectional)
══════════════════════════════════════════════

Authentication:
───────────────
• Private App Access Token (recommended for server-to-server)
  - Scoped permissions: deals, contacts, properties (read/write)
  - Stored in environment variables / secrets manager
  - Rotated every 90 days

API Client:
───────────
• Official HubSpot Node.js SDK: @hubspot/api-client
  OR
• Build custom wrapper with axios for more control

Sync Pattern:
─────────────
1. PULL (HubSpot → Dashboard):
   • Webhook subscription for deal updates
     - Deal created
     - Deal stage changed
     - Deal owner changed
   • Fallback: Scheduled polling (every 15 min) for new deals
   • One-time bulk import at initial deployment

2. PUSH (Dashboard → HubSpot):
   • Event-driven: Listen for dashboard changes
   • Queue-based: Add to job queue for async processing
   • Batch similar updates to reduce API calls
   • Retry logic: Exponential backoff (1s, 2s, 4s, 8s, 16s)

Rate Limiting Strategy:
───────────────────────
• HubSpot limit: 100 requests/10 seconds (burst), daily limits
• Dashboard strategy:
  - Track API calls in Redis with sliding window
  - Queue excess requests
  - Implement request throttling
  - Use batch endpoints where available (update multiple deals)

Error Handling:
───────────────
• 429 (Rate Limit): Retry after header, exponential backoff
• 401/403 (Auth): Alert admin, log to error tracking
• 400 (Bad Request): Log request/response, alert IM
• 500 (HubSpot Error): Retry up to 3 times, then alert
• Network errors: Retry with exponential backoff


Jira Integration (READ-ONLY)
═════════════════════════════

Authentication:
───────────────
• Jira Cloud: OAuth 2.0 or API Token
• Jira Data Center: Personal Access Token

API Client:
───────────
• jira-client (npm) or custom REST client

Data Flow:
──────────
• Polling approach: Query for CROs linked to account (every 30 min)
• JQL query: project = CRO AND customer = "Account Name"
• Cache results in integration_cache table
• Display in dashboard Project Detail → Jira Tab

Data Needed:
────────────
• CRO issue key (e.g., CRO-123)
• Priority
• Status
• Created date / Age
• Summary
• Assignee


Zendesk Integration (READ-ONLY)
════════════════════════════════

Authentication:
───────────────
• API Token or OAuth 2.0

API Client:
───────────
• node-zendesk (npm) or custom REST client

Data Flow:
──────────
• Query tickets by organization (linked to account)
• Polling: Every 30 minutes
• Aggregate metrics:
  - Total tickets (last 30 days)
  - High-priority tickets
  - Average response time

Data Needed:
────────────
• Ticket volume (count)
• Priority distribution
• Status (open, pending, solved)
• Created date


ParentSquare Analytics Integration (READ-ONLY)
═══════════════════════════════════════════════

Authentication:
───────────────
• Internal API token or service account

API Client:
───────────
• Custom REST client (ParentSquare internal API)

Data Flow:
──────────
• Query usage metrics for district/organization
• Polling: Daily (overnight batch job)
• Real-time queries for milestone validation

Metrics Needed:
───────────────
• User login rate (% of total users)
• Feature adoption rates:
  - Comms sent
  - Forms completed
  - Events created
  - Directory usage
• Data import completion status
• SIS integration status


Notification Integrations
══════════════════════════

Slack:
──────
• Slack Incoming Webhooks (simple)
  OR
• Slack Web API + Bot User (richer interactions)

Use cases:
- Alert when project goes Yellow/Red
- Notify when milestone completed
- Daily digest of at-risk projects

Email:
──────
• SendGrid or AWS SES
• Templates for:
  - Health alert notifications
  - Weekly IM digest
  - Contact gap reminders
```

---

## Phase 1 Implementation Roadmap

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PHASE 1 IMPLEMENTATION ROADMAP                          │
│                      (Estimated 8-12 weeks to MVP)                           │
└─────────────────────────────────────────────────────────────────────────────┘

WEEK 1-2: FOUNDATION & SETUP
═════════════════════════════
□ Technical setup
  ├─ Repository creation and branch strategy
  ├─ Development environment setup (Docker Compose)
  ├─ CI/CD pipeline skeleton (GitHub Actions)
  └─ Infrastructure provisioning (Terraform: VPC, RDS, ECS clusters)

□ HubSpot preparation
  ├─ Create Private App in HubSpot
  ├─ Define custom properties (im_phase, im_health_status, etc.)
  ├─ Document existing deal stages and mapping strategy
  └─ Identify sample deals for testing

□ Requirements finalization
  ├─ Confirm required contact roles (4-5 roles)
  ├─ Finalize milestone checklist (6 milestones for v1)
  ├─ Define health score rules (simple G/Y/R logic)
  └─ Align on stage progression rules (1-2 transitions)

Deliverable: Development environment ready, HubSpot integration configured


WEEK 3-4: DATABASE & CORE BACKEND
══════════════════════════════════
□ Database implementation
  ├─ Set up PostgreSQL (RDS)
  ├─ Create schema (implementation_projects, sync_log, etc.)
  ├─ Seed test data
  └─ Set up Redis cache

□ Core backend services
  ├─ API gateway and routing
  ├─ Authentication middleware (SSO/OAuth)
  ├─ Project Service (CRUD operations)
  └─ Basic API endpoints
      • GET /api/projects (list)
      • GET /api/projects/{id} (detail)
      • PATCH /api/projects/{id} (update)

□ HubSpot Connector (initial)
  ├─ API client setup (@hubspot/api-client)
  ├─ Test connection and authentication
  └─ Implement deal import function

Deliverable: Core backend API functional, database schema deployed


WEEK 5-6: HUBSPOT SYNC SERVICE
═══════════════════════════════
□ Sync Service implementation
  ├─ Build sync engine with field mapping
  ├─ Implement TO_HUBSPOT sync logic
      • Field updates (im_phase, im_health_status, etc.)
      • Activity logging (notes to HubSpot)
  ├─ Implement FROM_HUBSPOT import logic
      • Initial deal import
      • Webhook receiver endpoint
  ├─ Add sync_log audit trail
  ├─ Rate limiting and retry logic
  └─ Background job: Poll new deals (scheduled)

□ Testing
  ├─ Unit tests for sync logic
  ├─ Integration tests with HubSpot sandbox
  └─ Test error handling and retries

Deliverable: Bidirectional HubSpot sync working, tested with sample deals


WEEK 7-8: FRONTEND FOUNDATION
══════════════════════════════
□ Frontend setup
  ├─ React + TypeScript project scaffolding
  ├─ UI component library integration (MUI/Ant Design)
  ├─ React Query setup for API calls
  ├─ Routing (React Router)
  └─ Authentication flow

□ Portfolio View (main page)
  ├─ Project list table with sorting/filtering
  ├─ Filter bar (IM owner, phase, health, date range)
  ├─ Summary stats widgets
  └─ Link to project detail view

□ API integration
  ├─ Connect to backend API
  ├─ Implement data fetching
  └─ Loading and error states

Deliverable: Portfolio View functional, displaying real projects from database


WEEK 9-10: PROJECT DETAIL VIEW & MILESTONES
════════════════════════════════════════════
□ Project Detail Page
  ├─ Project header (account info, owner, dates)
  ├─ Phase progress bar/stepper
  ├─ Milestones panel
      • Checklist with checkboxes
      • Completion dates
      • Notes per milestone
  ├─ HubSpot snapshot panel (read-only fields)
  ├─ Save functionality
  └─ Loading states and optimistic updates

□ Backend support
  ├─ PATCH /api/projects/{id}/milestones endpoint
  ├─ POST /api/projects/{id}/phase endpoint (advance phase)
  └─ Validation logic (e.g., can't advance phase without milestones)

□ Testing
  ├─ E2E test: Complete milestone → triggers HubSpot sync
  └─ Verify sync_log records updates correctly

Deliverable: IMs can manage milestones and see syncs to HubSpot


WEEK 11: HEALTH SCORING & CONTACT COVERAGE
═══════════════════════════════════════════
□ Health Service (simple v1)
  ├─ Implement basic health calculation
      • Rule: Days behind schedule
      • Rule: Manual IM flag
      • Output: Green / Yellow / Red
  ├─ Store in health_scores table (with history)
  ├─ API endpoints
      • GET /api/health/{project_id}
      • POST /api/health/{project_id}/override
  └─ Sync to HubSpot im_health_status field

□ Health Panel (frontend)
  ├─ Current health display (large visual indicator)
  ├─ System-suggested vs override toggle
  ├─ Reason text field (required for override)
  └─ Save functionality

□ Contact Coverage (basic v1)
  ├─ Contact Service implementation
      • Query HubSpot for contacts associated with deal
      • Map to required roles (5 roles)
      • Identify gaps
  ├─ contacts_coverage table population
  ├─ Contact Coverage Panel (frontend)
      • List of required roles
      • Status indicator (filled/missing)
      • Display missing roles alert
  └─ Sync coverage status to HubSpot (im_contact_coverage field)

Deliverable: Health scoring and contact coverage working, displayed in UI


WEEK 12: TESTING, POLISH & PILOT ROLLOUT
═════════════════════════════════════════
□ Comprehensive testing
  ├─ End-to-end tests for critical workflows
      • New deal import → project creation
      • Milestone completion → HubSpot sync → stage change
      • Health override → sync to HubSpot
  ├─ Load testing (simulate 50-100 concurrent users)
  ├─ Security testing (auth, API authorization)
  └─ Cross-browser testing

□ Monitoring & observability
  ├─ Set up CloudWatch dashboards
  ├─ Configure alerts
      • Failed sync jobs
      • API errors
      • High latency
  ├─ Error tracking (Sentry or similar)
  └─ Audit log viewer (admin page)

□ Documentation
  ├─ User guide for IMs (with screenshots)
  ├─ Admin guide (configuration, monitoring)
  ├─ Technical documentation (architecture, API docs)
  └─ Runbook for common issues

□ Pilot rollout
  ├─ Select 3-5 pilot IMs
  ├─ Import their active projects
  ├─ Training session (1 hour)
  ├─ Daily check-ins during pilot week
  ├─ Collect feedback
  └─ Iterate on UX issues

Deliverable: MVP ready for pilot, monitoring in place, documentation complete


POST-PHASE 1: ITERATION & EXPANSION
════════════════════════════════════
□ Gather pilot feedback and prioritize improvements

□ Phase 2 features (priority order):
  1. Enhanced health scoring (integrate Jira + Zendesk signals)
  2. ParentSquare usage analytics integration
  3. CRO visibility (Jira integration)
  4. Zendesk support metrics
  5. Automated Slack/email alerts
  6. Activity timeline/feed
  7. Advanced reporting and dashboards
  8. Multiple implementations per deal (data model extension)

□ Scale rollout to all IMs

□ Phase 3: AI/ML capabilities
  ├─ Predictive health scores
  ├─ Anomaly detection
  ├─ Smart recommendations
  └─ Natural language insights
```

---

## Key Features & Capabilities

### For Implementation Managers

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          IM USER EXPERIENCE                                  │
└─────────────────────────────────────────────────────────────────────────────┘

Daily Workflow:
═══════════════
1. Login → Land on Portfolio View
   ├─ See all projects assigned to them (default filter)
   ├─ At-a-glance health indicators (color-coded)
   ├─ Sort by: Go-live date (ascending) to prioritize imminent launches
   └─ Red/Yellow health projects surface at top

2. Select a project → Project Detail View
   ├─ Quickly review phase and milestone status
   ├─ Check off completed milestones (one click)
   ├─ Review HubSpot snapshot (no need to open HubSpot)
   └─ Update health if issues arise (with reason notes)

3. Save changes → Automatic sync to HubSpot
   ├─ IM sees confirmation: "Synced to HubSpot ✓"
   ├─ No manual HubSpot updates required
   └─ Activity logged for audit trail

4. Monitor alerts
   ├─ Slack notification if a project health degrades
   ├─ Email digest: Weekly summary of portfolio health
   └─ Contact gap alerts

Time Savings:
─────────────
• Before: 30-45 min/day updating HubSpot manually
• After: 5-10 min/day managing projects in dashboard
• Reduction: 75% time savings on administrative tasks


Key Capabilities:
═════════════════
□ Portfolio Management
  ├─ View all assigned implementations in one table
  ├─ Filter by phase, health, go-live date range
  ├─ Search by account name
  ├─ Quick status overview (# at risk, # launching soon)
  └─ Export portfolio to CSV for offline analysis

□ Milestone Tracking
  ├─ Standardized checklist across all implementations
  ├─ One-click completion (auto-timestamps)
  ├─ Add notes per milestone (context for future reference)
  ├─ Visual progress indicator (X of Y complete)
  └─ Phase auto-advances when criteria met

□ Health Management
  ├─ System-suggested health based on signals
  ├─ Override capability (IM knows context system doesn't)
  ├─ Required reason for override (audit trail)
  ├─ Historical health trend chart (spot patterns)
  └─ Contributing factors breakdown (why Yellow/Red?)

□ Contact Coverage Validation
  ├─ See which key roles are filled vs missing
  ├─ Visual gap indicators (red warning icon)
  ├─ Direct link to HubSpot contact (when filled)
  └─ Task/reminder to CSM to fill gaps

□ Multi-System Visibility (Phase 2+)
  ├─ ParentSquare usage: Login %, feature adoption
  ├─ Jira CROs: See open bugs/requests for this account
  ├─ Zendesk: Ticket volume and severity
  └─ All in one place (no context switching)
```

### For Customer Success & Sales

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     VALUE FOR CS/SALES (HUBSPOT USERS)                       │
└─────────────────────────────────────────────────────────────────────────────┘

What They See in HubSpot:
══════════════════════════
□ Always-accurate implementation fields
  ├─ im_phase: Current implementation phase (Kickoff → Completed)
  ├─ im_health_status: Green / Yellow / Red (real-time)
  ├─ im_target_go_live_date: IM's best estimate
  ├─ last_im_update_date: Freshness indicator
  ├─ im_contact_coverage: Complete / Partial / Incomplete
  └─ im_missing_roles: Specific gaps (e.g., "SIS Owner")

□ Automatic deal stage progression (optional Phase 1)
  ├─ When IM marks "Launched" → Deal stage → "Live"
  ├─ When IM marks "Completed" → Deal stage → "Implemented"
  └─ Eliminates manual stage updates

□ Activity notes (automatically logged)
  ├─ "[IM Dashboard] Health changed: Green → Yellow"
  ├─ "[IM Dashboard] Phase advanced: In Progress → Launched"
  ├─ "[IM Dashboard] Milestone completed: Go-live"
  └─ Full transparency into implementation progress

Benefits:
═════════
• Data Integrity: Trust that HubSpot deal pipeline reflects reality
• Visibility: See implementation health without asking IM
• Reporting: Accurate forecasting and pipeline reports
• Collaboration: Clear handoff points between Sales → CS → IM
• Retention Signals: Early warning (Red health) enables proactive outreach
```

### For Leadership & Operations

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     VALUE FOR LEADERSHIP & OPERATIONS                        │
└─────────────────────────────────────────────────────────────────────────────┘

Strategic Insights:
═══════════════════
□ Portfolio Health Dashboard
  ├─ Real-time view: % of implementations Green/Yellow/Red
  ├─ Trend analysis: Is health improving or degrading over time?
  ├─ IM workload distribution: Are some IMs overloaded?
  └─ At-risk implementations: Which accounts need intervention?

□ Implementation Performance Metrics
  ├─ Average time-to-launch by product type
  ├─ Milestone completion rates
  ├─ Bottleneck identification (which milestones take longest?)
  ├─ Contact coverage gaps (systemic issues?)
  └─ Success rate (% launched on time)

□ Predictive Analytics (Phase 3)
  ├─ Likelihood of delays (based on historical patterns)
  ├─ Churn risk indicators (low usage + high tickets)
  ├─ Resource allocation recommendations
  └─ Impact of early interventions on outcomes

Reports Available:
══════════════════
• Weekly Executive Summary (automated email)
  - Total active implementations
  - At-risk count and specific accounts
  - Implementations launching next 30 days
  - Key blockers and trends

• Monthly Operations Review
  - Phase distribution (how many in each phase)
  - Average time in each phase
  - Health score distribution
  - IM productivity metrics (projects per IM, update frequency)

• Quarterly Business Review
  - Implementations completed (vs target)
  - On-time launch %
  - Expansion opportunities (accounts ready for upsell)
  - Process improvement recommendations
```

---

## Technical Considerations & Best Practices

### Data Consistency & Synchronization

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DATA CONSISTENCY STRATEGY                                 │
└─────────────────────────────────────────────────────────────────────────────┘

Source of Truth Principles:
═══════════════════════════
1. Implementation Project Data (Phase, Health, Milestones)
   └─ SOURCE OF TRUTH: IM Dashboard database
   └─ HubSpot: Mirror/replica (read-only for CS/Sales)

2. Deal Metadata (Amount, Close Date, Owner)
   └─ SOURCE OF TRUTH: HubSpot
   └─ IM Dashboard: Cached snapshot (refreshed periodically)

3. Contacts
   └─ SOURCE OF TRUTH: HubSpot
   └─ IM Dashboard: Validates coverage, doesn't create/edit (v1)


Conflict Resolution:
════════════════════
Scenario: Field updated in both systems simultaneously

Resolution Strategy:
├─ Last-Write-Wins (timestamp-based)
│  └─ Compare updated_at timestamps
│  └─ Most recent change prevails
│
├─ IM Dashboard Always Wins (for implementation fields)
│  └─ im_phase, im_health_status always pushed from dashboard
│  └─ HubSpot updates to these fields ignored (webhook filtered)
│
└─ Conflict Logging
   └─ Log all conflicts to sync_log
   └─ Alert admin for review


Sync Timing & Strategy:
════════════════════════
□ Real-time Sync (event-driven):
  ├─ Trigger: IM updates phase, health, milestone
  ├─ Action: Immediate sync to HubSpot (queued)
  ├─ Latency: < 5 seconds (user sees "Syncing..." then "Synced ✓")
  └─ Fallback: If sync fails, retry in background

□ Scheduled Sync (batch):
  ├─ Frequency: Nightly (2 AM)
  ├─ Purpose: Catch any missed syncs, reconcile data
  ├─ Process:
  │   1. Compare dashboard vs HubSpot for all projects
  │   2. Identify discrepancies
  │   3. Re-sync using dashboard as source of truth
  │   4. Log reconciliation report
  └─ Alert if discrepancies > threshold (e.g., 5%)

□ HubSpot → Dashboard (webhooks):
  ├─ Listen for: Deal created, deal owner changed
  ├─ Action: Update dashboard cache
  ├─ Frequency: Real-time (as events occur)
  └─ Fallback: Poll every 15 min for new deals


Idempotency:
════════════
• All sync operations must be idempotent
  └─ Running sync twice = same result as once
  └─ Use unique sync_id or timestamp to detect duplicates
  └─ HubSpot API: Use record ID to update (not create duplicate)


Audit Trail:
════════════
• Every sync operation logged to sync_log table
  ├─ What changed (old value → new value)
  ├─ When (timestamp)
  ├─ Who triggered it (user_id or 'SYSTEM')
  ├─ Success/failure status
  └─ HubSpot API response

• Queryable for debugging:
  └─ "Show me all syncs for project X in last 24 hours"
  └─ "Why did this deal stage change?"
```

### Security & Compliance

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       SECURITY & COMPLIANCE                                  │
└─────────────────────────────────────────────────────────────────────────────┘

Authentication:
═══════════════
□ User Authentication:
  ├─ SSO (SAML or OAuth 2.0) - integrate with ParentSquare identity provider
  ├─ No local passwords (reduce risk)
  ├─ Session timeout: 8 hours
  └─ MFA enforcement (recommended)

□ Service-to-Service Authentication:
  ├─ HubSpot: Private App Access Token
  ├─ Jira/Zendesk: API tokens or OAuth
  ├─ Store in AWS Secrets Manager (encrypted at rest)
  ├─ Rotate every 90 days (automated)
  └─ Never log secrets (redact in logs)


Authorization:
══════════════
□ Role-Based Access Control (RBAC):
  ├─ IM: Full access to assigned projects, read-only for others
  ├─ IM Manager: Full access to all projects
  ├─ Admin: Full access + settings/configuration
  ├─ Read-Only: View portfolio, no edit access
  └─ Implemented at API layer (middleware checks role per endpoint)

□ HubSpot Permissions:
  ├─ Dashboard API token scoped to specific objects (Deals, Contacts)
  ├─ Write access limited to implementation fields only
  └─ No delete permissions


Data Privacy:
═════════════
□ Sensitive Data Handling:
  ├─ Contact information: Names, emails, phone numbers (from HubSpot)
  ├─ Notes: May contain sensitive district information
  ├─ Encryption: TLS 1.3 in transit, AES-256 at rest (RDS encryption)
  └─ Access logging: Track who viewed which projects (audit_log)

□ GDPR/CCPA Considerations:
  ├─ Data retention policy: Archive completed projects after 2 years
  ├─ Right to deletion: Ability to purge project data
  ├─ Data export: Users can export their project data
  └─ Consent: No direct customer data (B2B tool, district contact data only)


API Security:
═════════════
□ Rate Limiting:
  ├─ Per-user limits: 100 requests/min (prevents abuse)
  ├─ Per-IP limits: 1000 requests/min (DDoS protection)
  └─ HubSpot API: Respect their limits (100/10s)

□ Input Validation:
  ├─ All inputs validated (Zod/Joi schemas)
  ├─ SQL injection prevention (parameterized queries via ORM)
  ├─ XSS prevention (sanitize user input, CSP headers)
  └─ CSRF protection (tokens for state-changing requests)

□ API Authentication:
  ├─ JWT tokens (short-lived: 1 hour)
  ├─ Refresh tokens (stored in httpOnly cookies)
  └─ API key for service-to-service (internal integrations)


Monitoring & Incident Response:
════════════════════════════════
□ Security Monitoring:
  ├─ Failed login attempts (alert after 5 failures)
  ├─ Unusual API activity (spikes, off-hours access)
  ├─ Permission escalation attempts
  └─ Data export events (log all exports)

□ Incident Response Plan:
  ├─ Detection: Automated alerts + daily log review
  ├─ Containment: Ability to revoke API tokens immediately
  ├─ Investigation: Audit logs + sync logs for forensics
  ├─ Recovery: Database backups (point-in-time restore)
  └─ Post-mortem: Document and improve


Compliance:
═══════════
□ SOC 2 Considerations (if ParentSquare pursues certification):
  ├─ Audit logging (all data changes)
  ├─ Access controls (RBAC enforced)
  ├─ Encryption (in transit and at rest)
  ├─ Backup and recovery (tested quarterly)
  └─ Change management (all code changes reviewed)
```

### Performance & Scalability

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PERFORMANCE & SCALABILITY                               │
└─────────────────────────────────────────────────────────────────────────────┘

Performance Targets:
════════════════════
• Portfolio View load: < 2 seconds (for 100 projects)
• Project Detail load: < 1 second
• Save operation: < 500ms (optimistic UI update)
• HubSpot sync: < 5 seconds (async, user sees confirmation)
• API response time (p95): < 200ms


Scalability Assumptions:
════════════════════════
Phase 1:
├─ Users: 10-20 IMs + 30-50 CS/Sales (HubSpot users)
├─ Projects: 200-500 active implementations
├─ Requests: ~1,000 API requests/day
└─ HubSpot syncs: ~500 sync operations/day

Phase 2-3 (Growth):
├─ Users: 50 IMs + 200 CS/Sales
├─ Projects: 2,000 active implementations
├─ Requests: ~10,000 API requests/day
└─ HubSpot syncs: ~3,000 sync operations/day


Optimization Strategies:
════════════════════════
□ Database:
  ├─ Indexes on frequently queried columns (im_owner_id, phase, health)
  ├─ Connection pooling (pg-pool)
  ├─ Read replicas for reporting queries (Phase 2)
  └─ Materialized views for analytics dashboards

□ Caching:
  ├─ Redis cache for:
  │   • HubSpot deal snapshots (TTL: 5 min)
  │   • User sessions
  │   • API responses (GET /api/projects - TTL: 1 min)
  ├─ Browser cache for static assets (CDN)
  └─ Cache invalidation on updates (smart invalidation)

□ API:
  ├─ Pagination (limit 50 projects per page)
  ├─ Field selection (only return requested fields)
  ├─ Batch endpoints (update multiple milestones in one call)
  └─ GraphQL (Phase 2) for flexible querying

□ Frontend:
  ├─ Code splitting (lazy load Project Detail page)
  ├─ Virtual scrolling for long lists
  ├─ Optimistic UI updates (don't wait for server)
  ├─ React Query caching (minimize API calls)
  └─ Web workers for heavy client-side processing

□ Background Jobs:
  ├─ Queue-based processing (BullMQ)
  ├─ Job priority (health alerts > routine syncs)
  ├─ Concurrency limits (max 5 concurrent HubSpot syncs)
  └─ Job retry with exponential backoff


Monitoring Performance:
═══════════════════════
□ Key Metrics:
  ├─ API latency (p50, p95, p99)
  ├─ Database query time
  ├─ Cache hit rate (target: > 80%)
  ├─ HubSpot API call rate
  ├─ Job queue depth (backlog indicator)
  └─ Error rates (target: < 0.1%)

□ Tools:
  ├─ Application Performance Monitoring: New Relic or Datadog
  ├─ Database monitoring: pg_stat_statements
  ├─ CloudWatch dashboards: Custom metrics
  └─ Distributed tracing: AWS X-Ray or Jaeger


Scalability Plan:
═════════════════
Phase 1 (Small scale):
└─ Single ECS Fargate task, RDS instance (small), Redis (small)

Phase 2 (Medium scale):
├─ Horizontal scaling: 3-5 ECS tasks behind load balancer
├─ RDS read replica for reports
└─ Redis cluster (high availability)

Phase 3 (Large scale):
├─ Auto-scaling: Scale ECS tasks based on CPU/memory
├─ Database sharding (if > 100k projects - unlikely)
├─ CDN for static assets (CloudFront)
└─ Regional deployment (multi-region for global teams - future)
```

### Monitoring & Observability

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      MONITORING & OBSERVABILITY                              │
└─────────────────────────────────────────────────────────────────────────────┘

Three Pillars:
══════════════
1. Logs: What happened
2. Metrics: How much/how often
3. Traces: Where time was spent


Logging Strategy:
═════════════════
□ Structured Logging (JSON format):
  {
    "timestamp": "2025-01-15T10:30:00Z",
    "level": "INFO",
    "service": "sync-service",
    "project_id": "abc-123",
    "event": "hubspot_sync_completed",
    "duration_ms": 1234,
    "user_id": "user-456"
  }

□ Log Levels:
  ├─ DEBUG: Detailed diagnostic info (dev only)
  ├─ INFO: Key events (sync completed, user login)
  ├─ WARN: Potential issues (retry triggered, cache miss)
  ├─ ERROR: Failures requiring attention (API error, validation failed)
  └─ FATAL: System-level failures (database connection lost)

□ Log Aggregation:
  ├─ CloudWatch Logs (central repository)
  ├─ Log retention: 30 days (operational), 1 year (audit logs)
  ├─ Log groups by service (project-service, sync-service, etc.)
  └─ Search/filter capabilities (CloudWatch Insights)


Metrics to Track:
═════════════════
□ Application Metrics:
  ├─ API request count (by endpoint)
  ├─ API latency (p50, p95, p99)
  ├─ Error rate (% of requests failing)
  ├─ Active users (concurrent sessions)
  └─ Project operations (creates, updates, deletes)

□ Integration Metrics:
  ├─ HubSpot sync success rate (target: > 99%)
  ├─ HubSpot API call count (track against rate limit)
  ├─ Sync latency (time from trigger to completion)
  ├─ Failed sync count (alert threshold: > 10/hour)
  └─ Retry count (indicator of API instability)

□ Infrastructure Metrics:
  ├─ CPU utilization (ECS tasks)
  ├─ Memory utilization
  ├─ Database connections (pool usage)
  ├─ Database query time
  ├─ Redis hit rate
  └─ Job queue depth (BullMQ)

□ Business Metrics:
  ├─ Active implementations count
  ├─ Health distribution (% Green/Yellow/Red)
  ├─ Average time to launch
  ├─ Daily active users (IMs logging in)
  └─ Portfolio coverage (% of deals with IM project)


Alerting Rules:
═══════════════
Priority 1 (Page on-call):
├─ Application down (health check fails)
├─ Database connection lost
├─ Error rate > 5% (sustained 5 min)
└─ HubSpot sync failing 100% (sustained 10 min)

Priority 2 (Slack alert):
├─ Error rate > 1%
├─ API latency p95 > 1 second
├─ Failed syncs > 10/hour
├─ Job queue backlog > 1000
└─ Cache hit rate < 60%

Priority 3 (Email digest):
├─ Daily summary of warnings
├─ Weekly performance report
└─ Monthly trend analysis


Dashboards:
═══════════
□ Operations Dashboard:
  ├─ Real-time health status (green/yellow/red indicator)
  ├─ API request rate (last 1 hour)
  ├─ Error rate (last 24 hours)
  ├─ HubSpot sync status (last 100 syncs)
  └─ Job queue depth

□ Business Dashboard:
  ├─ Active implementations count
  ├─ Health score distribution (pie chart)
  ├─ Implementations by phase (bar chart)
  ├─ IM workload distribution
  └─ Daily/weekly active users

□ Performance Dashboard:
  ├─ API latency trends (line chart)
  ├─ Database query performance (slowest queries)
  ├─ Cache hit rate (gauge)
  └─ Resource utilization (CPU/memory)


Distributed Tracing:
════════════════════
• Tool: AWS X-Ray or Jaeger
• Trace ID: Propagated through all services
• Visualize:
  ├─ User request → API Gateway → Project Service → Database
  ├─ Sync trigger → Queue → Sync Service → HubSpot API
  └─ Identify bottlenecks (where is time spent?)


Incident Response:
══════════════════
1. Detect: Alert fires (Slack/PagerDuty)
2. Acknowledge: On-call engineer acknowledges
3. Investigate: Check dashboards, logs, traces
4. Mitigate: Apply fix (rollback, restart, hotfix)
5. Resolve: Verify resolution, close incident
6. Post-mortem: Document, identify root cause, prevent recurrence
```

---

## Success Metrics

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SUCCESS METRICS (Phase 1)                           │
└─────────────────────────────────────────────────────────────────────────────┘

User Adoption:
══════════════
Target: 90% daily active usage among IMs
Measurement:
├─ Daily logins / Total IMs
├─ Projects updated daily / Total active projects
└─ Time spent in dashboard (average session length)

Success Criteria:
├─ Week 1 (pilot): 80% DAU
├─ Week 4: 90% DAU
└─ Week 12: 95% DAU + IMs report it as primary tool


Efficiency Gains:
═════════════════
Target: 75% reduction in time spent on manual HubSpot updates
Measurement:
├─ Survey IMs: "How much time do you spend updating HubSpot per week?"
│   • Before: Baseline (estimated 3-5 hours/week)
│   • After: Measured (target: < 1 hour/week)
└─ Time saved = (Baseline - After) / Baseline * 100%

Success Criteria:
└─ 75% reduction achieved within 8 weeks of full rollout


Data Integrity:
═══════════════
Target: 95%+ accuracy in HubSpot deal stages
Measurement:
├─ Audit sample: Random sample of 50 deals/month
├─ Compare: IM Dashboard phase vs HubSpot stage
├─ Discrepancies / Total * 100% = Error rate
└─ Target error rate: < 5%

Success Criteria:
├─ Week 4: < 10% error rate
└─ Week 12: < 5% error rate (sustained)


Sync Reliability:
═════════════════
Target: 99% sync success rate
Measurement:
├─ Successful syncs / Total sync attempts * 100%
├─ Track failed syncs in sync_log table
└─ Alert threshold: < 95% success rate

Success Criteria:
├─ Phase 1 launch: > 95%
└─ Steady state (12 weeks): > 99%


Contact Coverage:
═════════════════
Target: 100% of implementations have complete contact coverage
Measurement:
├─ Projects with all required roles filled / Total projects * 100%
├─ Gap closure rate (how fast are gaps filled after identification)

Success Criteria:
├─ Baseline: Measure current state (likely 60-70% complete)
├─ 12 weeks: 90% of projects have complete coverage
└─ 24 weeks: 95%+ complete coverage


User Satisfaction:
══════════════════
Target: Net Promoter Score (NPS) > 50
Measurement:
├─ Quarterly survey: "How likely are you to recommend this dashboard? (0-10)"
├─ NPS = % Promoters (9-10) - % Detractors (0-6)

Success Criteria:
├─ Initial survey (week 4): Collect baseline
├─ 3 months: NPS > 30
└─ 6 months: NPS > 50


Business Impact:
════════════════
Target: Improved implementation launch predictability
Measurement:
├─ % of implementations launched within ±1 week of target date
├─ Baseline: Measure current state
├─ After: Track quarterly

Success Criteria:
├─ 6 months: 10% improvement in on-time launch rate
└─ 12 months: 20% improvement


Return on Investment (ROI):
═══════════════════════════
Calculation:
├─ Cost: Development + Infrastructure + Maintenance
│   • Development: 8-12 weeks * team cost (estimate $80k-$120k)
│   • Infrastructure: ~$500/month (ECS, RDS, Redis)
│   • Maintenance: 0.5 FTE (~$60k/year)
│   • Total Year 1: ~$150k-$200k
│
└─ Savings: IM time savings + Data quality improvements
    • 15 IMs * 4 hours/week saved * 50 weeks * $50/hour = $150k/year
    • Improved retention (1-2 fewer churns) = $500k-$1M/year
    • Better expansion (2-3 more upsells) = $200k-$500k/year
    • Total Annual Value: $850k-$1.65M

ROI = (Annual Value - Cost) / Cost * 100%
    = ($850k - $150k) / $150k * 100%
    = 467% ROI (conservative estimate)

Payback Period: ~2-3 months
```

---

## Appendix: Alternative Approaches Considered

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ALTERNATIVE APPROACHES CONSIDERED                       │
└─────────────────────────────────────────────────────────────────────────────┘

Approach 1: HubSpot-Native Solution
════════════════════════════════════
Description:
├─ Build everything within HubSpot (custom objects, workflows)
├─ Use HubSpot's UI for IM interface
└─ Leverage HubSpot's automation and reporting

Pros:
├─ No separate application to build/maintain
├─ Native integration (no sync needed)
├─ Use existing HubSpot licenses
└─ Sales/CS already familiar with HubSpot

Cons:
├─ Limited customization (constrained by HubSpot capabilities)
├─ Complex milestone tracking (not HubSpot's strength)
├─ Can't integrate ParentSquare, Jira, Zendesk easily
├─ HubSpot licensing costs (may need higher tier)
├─ Vendor lock-in
└─ Poor experience for IMs (not optimized for their workflow)

Decision: REJECTED
Rationale: HubSpot is great for CRM, but not designed for implementation
project management. Need dedicated tool optimized for IM workflows.


Approach 2: BI Tool Solution (Looker, Tableau, etc.)
═════════════════════════════════════════════════════
Description:
├─ Build dashboards in BI tool (read-only views)
├─ IMs continue updating HubSpot manually
└─ BI tool aggregates data from HubSpot + other systems

Pros:
├─ Faster to build (no application development)
├─ Good for reporting and analytics
└─ Can pull from multiple data sources

Cons:
├─ Read-only (doesn't solve manual update problem)
├─ No workflow management (just reporting)
├─ IMs still have to update HubSpot
├─ No automation (no sync, no alerts)
└─ Doesn't address core problem (data fragmentation)

Decision: REJECTED
Rationale: BI tools are for reporting, not operational workflows. Doesn't
reduce IM workload or improve data accuracy.


Approach 3: No-Code Platform (Airtable, Notion, etc.)
══════════════════════════════════════════════════════
Description:
├─ Use Airtable or Notion as implementation tracking database
├─ Build custom views and forms
├─ Use Zapier/Integromat for HubSpot sync
└─ Lightweight, quick to implement

Pros:
├─ Very fast to build (days, not weeks)
├─ No code required (accessible to non-developers)
├─ Flexible data model
└─ Good for prototyping/MVP

Cons:
├─ Limited customization (UI constrained by platform)
├─ Complex integrations require Zapier (additional cost, brittle)
├─ Scalability concerns (performance, rate limits)
├─ Security and compliance (data in third-party SaaS)
├─ Vendor lock-in (hard to migrate later)
└─ Limited support for complex workflows and logic

Decision: REJECTED (for production; OK for prototype)
Rationale: Good for quick prototype/pilot, but not durable for long-term
production use. ParentSquare needs control, customization, and scalability.


Approach 4: Spreadsheet + Scripts
══════════════════════════════════
Description:
├─ Google Sheets as implementation tracking database
├─ Google Apps Script for HubSpot sync
└─ Shared sheet for all IMs

Pros:
├─ Extremely fast to build (hours)
├─ Familiar interface (everyone knows spreadsheets)
└─ Free (no infrastructure costs)

Cons:
├─ No multi-user concurrency (conflicts, data loss)
├─ No access control (anyone can edit anything)
├─ No audit trail (who changed what?)
├─ Poor UX (not designed for this use case)
├─ Fragile scripts (breaks easily)
└─ Not scalable (performance degrades with data)

Decision: REJECTED
Rationale: Fine for 1-2 users, not suitable for team of 15+ IMs. No
production-grade reliability or security.


Approach 5: Modify Existing Internal Tool
══════════════════════════════════════════
Description:
├─ Extend existing ParentSquare internal tool (if one exists)
├─ Add implementation tracking module
└─ Leverage existing auth, infrastructure

Pros:
├─ Reuse existing infrastructure and authentication
├─ IMs already familiar with tool (lower learning curve)
├─ Faster development (some components already built)
└─ Unified internal tools experience

Cons:
├─ Depends on existing tool architecture (may not fit)
├─ Technical debt (if existing tool is legacy)
├─ Risk of bloating existing tool (scope creep)
└─ May inherit limitations of existing system

Decision: CONSIDER IF APPLICABLE
Rationale: If ParentSquare has a suitable internal tool (e.g., customer
operations platform), this could be viable. Evaluate case-by-case.


CHOSEN APPROACH: Net-New Internal Application
══════════════════════════════════════════════
Rationale:
├─ Purpose-built for IM workflows (optimal UX)
├─ Full control over features, integrations, roadmap
├─ Scalable and extensible (supports Phase 2, 3 expansions)
├─ Secure and compliant (meets ParentSquare standards)
├─ Durable (can evolve with business needs)
└─ Best ROI over 3-5 year horizon

Trade-off:
├─ Higher upfront investment (8-12 weeks development)
├─ Ongoing maintenance required
└─ Need dedicated ownership (team/person responsible)

Verdict: Best long-term solution for ParentSquare's needs.
```

---

## Document Metadata

```
Document: Implementation Manager Dashboard - Visual Architecture & Technical Specification
Version: 1.0
Date: 2025-01-15
Author: Enterprise Architecture Team
Status: DRAFT - For Review & Alignment

Change Log:
───────────
v1.0 (2025-01-15): Initial comprehensive specification based on:
  - Centralized Implementation Manager Dashboard and HubSpot Synchronization feedback
  - IM HubSpot API Proposal

Next Steps:
───────────
1. Review with stakeholders:
   - Implementation Manager team (validate workflows)
   - RevOps/Sales (validate HubSpot integration approach)
   - Engineering (validate technical feasibility)
   - Leadership (validate investment and ROI)

2. Address open questions from source document (Section 5)

3. Finalize Phase 1 scope and begin sprint planning

4. Provision HubSpot sandbox environment for development/testing

5. Initiate technical discovery (existing ParentSquare stack audit)

Approval Required From:
───────────────────────
□ Head of Implementation / Customer Success
□ VP of RevOps / Sales
□ CTO / VP of Engineering
□ VP of Product (if applicable)

Questions or Feedback:
──────────────────────
Contact: [Your contact information]
Slack Channel: #im-dashboard-project (suggested)
Document Updates: [Link to living document]
```

---

## Conclusion

This Implementation Manager Dashboard represents a strategic investment in operational excellence and data integrity. By creating a single source of truth for implementation project data, ParentSquare will:

1. **Empower Implementation Managers** with a purpose-built tool that reduces administrative burden by 75%, allowing them to focus on customer success rather than manual data entry.

2. **Improve Data Quality** across HubSpot, ensuring Sales, CS, and Leadership have accurate, real-time visibility into implementation health and progress.

3. **Enable Proactive Management** through automated health scoring and alerting, catching at-risk implementations before they become escalations.

4. **Scale Operations** with a flexible, extensible architecture that supports future integrations (Jira, Zendesk, product analytics, AI) without major refactoring.

5. **Drive Business Outcomes**: Better on-time launch rates, improved customer retention, and data-driven insights for strategic decision-making.

The recommended architecture balances pragmatism (getting to MVP quickly) with long-term vision (Phase 2-3 roadmap). The hub-and-spoke design with HubSpot as the primary integration ensures that existing workflows are respected while establishing the dashboard as the operational command center for implementation teams.

**Estimated Timeline**: 8-12 weeks to Phase 1 MVP, with pilot rollout by Week 12 and full deployment by Week 16.

**Estimated Investment**: $150k-$200k Year 1 (development + infrastructure + maintenance).

**Expected ROI**: 467% (conservative), with 2-3 month payback period.

This is not just a dashboard—it's a transformation of how ParentSquare manages implementation success.

---
