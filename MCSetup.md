# Enterprise Salesforce Marketing Cloud Next Implementation Playbook

## Section 1: Architecture Overview

To maintain data silos while utilizing Data Cloud’s power, we use a **Hybrid Identity Architecture**.

### The "Unified" Concept

* **CRM Layer (Siloed)**: Each office uses its own **Record Type** and **Sharing Rules**. This allows duplicate records for the same person to exist independently in the CRM, ensuring private data points remain separate.
* **Data Cloud Layer (Unified)**: **Identity Resolution** matches these "duplicates" using attributes like Email or Phone to create a single **Unified Individual**.
* **UI Layer (Aggregated)**: Admins use the **Unified Individual ID** to show total engagement scores on siloed record pages via the **Profile Insights** component.

---

## Section 2: Global Core Enablement (Office Agnostic)

*Required once for the entire Salesforce Org.*

| Task | Description | LOE (Hrs) | Complexity |
| --- | --- | --- | --- |
| **Enable Data Cloud** | Activate Data Cloud in Setup; requires a Data Cloud Architect . | 1–2 | Low |
| **Install Marketing Data Kits** | Install standard packages for Consent, Flow, and Messaging Channels . | 2–4 | Medium |
| **Core Identity Resolution** | Generate initial ruleset for the `Individual` object . | 2–3 | High |
| **Global Email Auth** | Configure DKIM and authenticate root sending domains . | 4–6 | Medium |
| **AI Feature Enablement** | Turn on Agentforce and STO Global Models . | 2–3 | Low |
| **Performance App** | Install the **Marketing Performance** app for embedded reporting . | 2–4 | Medium |

---

## Section 3: CRM Silo Construction

*Ensures offices only see their own contacts in list views.*

| Task | Description | LOE (Hrs) | Complexity |
| --- | --- | --- | --- |
| **Record Type Creation** | Create unique Record Types for the Contact object to manage program-specific fields . | 4–6 | Medium |
| **Sharing & Scoping Rules** | Implement criteria-based rules (e.g., `Program_Office__c`) for isolation. | 3–5 | High |
| **Dynamic Record Pages** | Build Lightning pages with **Dynamic Forms** to show/hide sections based on user program . | 3 | Medium |
| **Component Placement** | Add **Profile Insights** and **Engagement** components to record pages . | 1 | Low |

---

## Section 4: Program Office Onboarding (Repeatable)

*These tasks must be repeated for each of the 20 program offices.*

| Task | Description | LOE (Hrs) | Complexity |
| --- | --- | --- | --- |
| **Data Space Config** | Create a dedicated **Data Space** to silo segments, scores, and DMOs . | 2–4 | Medium |
| **User Access & CMS** | Assign permission sets and add users to specific CMS Workspaces . | 2–3 | Low |
| **Identity Resolution** | Configure custom rulesets for specific program Individual/Account needs . | 3–5 | High |
| **Channels & Senders** | Verify program-specific "From" addresses and SMS/WhatsApp codes . | 4–8 | Medium |
| **Subscriptions** | Create unique Communication Subscriptions and preference pages . | 2–3 | Medium |
| **Scoring Model Build** | Create Engagement/Fit scoring models tailored to the program's KPIs . | 3–5 | High |
| **Web Tracking Setup** | Configure office-specific landing page tracking or external site connectors . | 2–4 | Medium |
| **Analytics Setup** | Create Analytics Collections and share folders/dashboards with the team . | 1–2 | Low |

---

## Section 5: Field Audit Guide (Global vs. Local)

| Field Category | Definition | Examples |
| --- | --- | --- |
| **Global (Shared)** | Standard fields used for Identity Resolution . | Email, Mobile Phone, First Name, Last Name. |
| **Local (Unique)** | Fields unique to a specific Office Record Type . | Veteran ID, Student Status, Program Eligibility. |
| **Calculated (Unified)** | Data pulled from Data Cloud to the UI . | Overall Marketing Score, Engagement Score. |

---

## Section 6: Artifacts & Checklists

### Artifact A: Admin Training Prompt Templates

* **Campaign Brief**: "I need to create a campaign for [Program Office]. Target audience is [X]. Please draft a brief."
* **Content Generation**: "Based on the brief, draft a three-part email series with a [Tone] voice."
* **Segment Suggestion**: "Suggest a segment of users in our Data Space who interacted with [Landing Page] but haven't subscribed."

### Artifact B: Scoring Cheat Sheet

* **Form Submit**: +10 Points
* **Search/Link Click**: +3 Points
* **Unsubscribe**: -5 Points
* **Fit (US Resident)**: +3 Points

### Artifact C: Go-Live Readiness Survey

1. Is the **Data Space** active and selected for all marketing activities ?
2. Are **Identity Resolution Rules** published for your specific Individual records ?
3. Have **Communication Subscriptions** been created and mapped to the preference page ?
4. Has the **Physical Address** been verified in Company Information for compliance ?

---
