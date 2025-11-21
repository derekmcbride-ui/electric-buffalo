# **Centralized Implementation Manager (IM) Dashboard and HubSpot Synchronization feedback**

### **Framing, v1 Scope, and Questions for Discussion**

## **1\. Project framing**

This project is a **net-new, durable internal application** that will:

* Own core Implementation project data and workflows.

* Integrate with HubSpot for visibility and reporting.

* Extend over time to incorporate additional systems (e.g., Jira, Zendesk, product usage) and AI.

Think of it as the **“Implementation Command Console” for ParentSquare**, with **HubSpot as the external-facing CRM surface** for Sales, CS, and leadership.

### **Core problems we are solving**

* HubSpot deal stages and fields do **not consistently reflect reality** of implementation progress.

* Implementation Managers (IMs) must piece together status from multiple places (HubSpot, internal tools, notes), which leads to **inconsistent reporting and manual updates**.

* There is **no single, trusted “source of truth”** for implementation phase and health that both IMs and CS/Sales can rely on.

* Contact coverage (key district roles) is **uneven**, and gaps are hard to see early.

### **Core goals for Phase 1**

1. Provide a **single place for IMs** to manage and track implementations (their “home base”).

2. Ensure **implementation phase and health** are captured in one system and **mirrored into HubSpot**.

3. Reduce **manual HubSpot updates** for implementation details.

4. Start enforcing **basic contact coverage** for key roles during implementation.

---

## **2\. v1 vision: What this dashboard is**

For v1, the IM Dashboard is:

* An internal web application (using our standard internal stack) where IMs:

  * See and manage their portfolio of implementations.

  * Update milestones, phases, and health.

  * See a minimal, relevant subset of HubSpot information.

* A small integration layer that:

  * Syncs core implementation fields to HubSpot deal properties.

  * Optionally advances select HubSpot stages based on implementation phase.

Later phases can add:

* Jira and Zendesk signals into health.

* Deeper alerting and automation (Slack, email, AI helpers).

* Support for multiple implementations per deal (e.g., separate projects per product/site).

---

## **3\. v1 scope: What is in and out**

### **In scope for v1**

**A. IM portfolio view (IM “home screen”)**

A table of active implementations with filters such as:

* IM owner

* Implementation phase (e.g., Kickoff, In Progress, Launched, Stabilizing, Completed)

* Health (Green / Yellow / Red)

* Target go-live window (e.g., due in the next 30 days)

Example columns:

* District / account name

* Product type (Comms, Remind, SmartSites, etc.)

* Implementation phase

* Health (icon/flag)

* Target go-live date

* Days since last IM status update

**B. Implementation detail view**

For each implementation project, show:

* Core identifiers:

  * HubSpot Deal ID

  * District / account info

  * Product(s) in scope

  * IM owner

  * Target go-live date

* Milestone checklist (v1, simplified), for example:

  * Kickoff completed

  * Data import complete

  * SIS integration configured

  * Staff training completed

  * Soft launch / pilot complete

  * Go-live complete

* Health panel:

  * Current health (Green / Yellow / Red)

  * System-suggested status (optional v1) vs IM override

  * IM “reason” / notes when overriding

* HubSpot snapshot:

  * Read-only view of key fields (deal stage, amount, close date, etc.)

  * Direct link into the HubSpot deal record

**C. Basic HubSpot synchronization rules**

When an IM updates implementation data (for example, phase, target go-live date, health), the app updates corresponding **HubSpot deal properties**. Proposed v1 properties:

* `im_phase`

* `im_health_status`

* `im_target_go_live_date`

* `last_im_update_date`

Optional v1 behavior (if we agree it’s safe):

* For 1–2 major transitions, the app also updates the **HubSpot deal stage**, for example:

  * When implementation phase moves to `Launched`, update deal stage to something like `Live` / `Implemented` (final naming TBD with Sales/RevOps).

**D. Contact coverage panel (v1 “lite”)**

For each implementation:

* Define a small set of **required roles**, for example:

  * District admin / executive sponsor

  * SIS owner / data lead

  * Communications / parent engagement lead

* The dashboard checks HubSpot for contacts that match these roles and shows:

  * Coverage (role filled)

  * Gaps (e.g., “Missing SIS owner contact”)

v1 behavior:

* **Flag gaps only** (no requirement to build full contact creation flows in v1).

* Optionally, provide:

  * A simple way to capture contact details that can later be used to create/update HubSpot contacts, or

  * Guidance/tasks for CSM/Sales to fill the gaps.

---

### **Explicitly out of scope for v1 (Phase 2+)**

* Integrating Jira and Zendesk into the health score.

* Complex health scoring (0–100) with weighted inputs from multiple systems.

* Advanced automation (Slack notifications, playbooks, auto-generated comms).

* Fully supporting multiple independent implementation projects per deal (we will design with this in mind but not fully implement it).

* Large-scale HubSpot workflow changes beyond a few clear, agreed-upon stage sync rules.

---

## **4\. Draft v1 data model (high level)**

**Implementation Project (core object)**

At minimum:

* Implementation Project ID

* HubSpot Deal ID

* District / account name

* Product type

* IM owner

* Implementation phase

* Health status (G/Y/R)

* Target go-live date

* Last IM update date

* Key milestone flags (boolean fields for the v1 checklist)

* Notes / status summary

This object is the **source of truth** for implementation phase and health. HubSpot mirrors a subset of these fields.

---

## **5\. Open questions for alignment**

These are the key questions for us to align on before detailed spec and build.

### **A. Architecture and ownership**

1. Do we agree this should be a **net-new internal web application** (IM’s primary home), rather than trying to do everything directly in HubSpot or a BI tool?

2. Where should this app live technically (stack, hosting) given our existing standards?

3. Which team should **own this application long-term** (Implementation/CS Ops, RevOps/Data, internal tools, or shared ownership)?

### **B. Data model and record linkage**

4. For v1, is it acceptable to assume **one Implementation Project per HubSpot Deal**?

   * Are there known deal structures (multiple products per deal) where this will cause immediate issues?

5. Besides the HubSpot Deal ID, which other IDs should we standardize on for the implementation object (e.g., a District/Org ID shared with other ParentSquare systems)?

### **C. HubSpot sync and stage mapping**

6. How aggressive should v1 be about **changing HubSpot deal stages** automatically?

   * Are we comfortable with 1–2 specific transitions tied to clear implementation phases (for example, “Launched” → “Live” stage)?

7. Which HubSpot fields do we all agree are “safe” to treat as **mirror-only fields** from the IM Dashboard (implementation phase, health, target go-live date, last IM update date, anything else)?

### **D. Health and risk**

8. What is the **simplest useful v1 health model** we can all agree on?

   * If we start with G/Y/R, what 2–3 rules (e.g., days behind plan, repeated delays) should suggest Yellow or Red?

9. Should the **IM’s manual judgment** always be able to override the system-suggested health in v1, as long as they provide a reason?

### **E. Contact coverage**

10. Which **roles are truly required** for a healthy implementation (for example, 3–5 standard roles the dashboard should check for in HubSpot)?

11. For v1, do we only want to **flag missing contacts**, or is there appetite to also support **creating/updating contacts** in HubSpot from the dashboard?

### **F. Adoption and rollout**

12. What has to be true for IMs to **live in this dashboard daily** (not just treat it as another reporting tool)?

13. Should we plan for:

    * A **pilot group** of IMs first, or

    * A broader rollout once v1 is stable?

---

Once we align on these points, we can turn this into a more detailed v1 spec with:

* Final field lists and data model.

* Specific mapping tables (implementation phase → HubSpot properties/stages).

* Concrete user stories and acceptance criteria for the initial build.

