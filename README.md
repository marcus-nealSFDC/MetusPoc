# 🛡️ METUS Agentforce – Contact Guardian and Verification (Use Case 2)

## Overview
**Contact Guardian** is an internal Agentforce-powered assistant designed to prevent duplicate Contacts at the point of creation and to assist users in verifying potential duplicates before creating new records.  
This use case is part of the **METUS POC** led by Marcus Neal (Salesforce Innovation Technical Architect) to demonstrate measurable data-quality improvements and agentic automation in Salesforce.

---

## 🎯 Business Challenge
Sales and service teams frequently create duplicate or mis-linked Contacts due to inconsistent manual entry and weak guardrails. This causes:
- Data fragmentation across Accounts
- Confusing reports and attribution errors
- Reduced user confidence and slower service resolution

**Goal:**  
Automate verification and pre-creation checks to raise Contact data completeness to >95% and reduce duplicates by ≥40%.

---

## 🧩 Agent Setup Steps

### 1. **Agent Creation in Agentforce Builder**
1. Go to **Setup → Agentforce → Agent Builder**.
2. Create a new **Internal Agent**:
   - **Agent Name:** `Contact Guardian`
   - **Primary Topic:** `General CRM`
   - **Channel:** `Internal Console / Utility Bar`
3. Assign required permissions:
   - Object Access: `Contact`, `Account`, `AccountContactRelation`
   - Read/Write: Contact, Email, Phone, and Account fields
   - Ensure FLS/sharing compliance is active

---

### 2. **Topic Configuration**
Create a **Topic** named **General CRM** (if not existing) or reuse it.

| Field | Value |
|-------|--------|
| Topic Name | `General CRM` |
| Purpose | Handle CRM operations like record lookup, update, and creation |
| Guardrails | Honor FLS, suppress internal IDs, log all decisions |
| Associated Actions | `Get Record Details`, `Query Records`, `DraftOrReviseEmail`, `CreateContact`, `Contact Verification Email` |

---

### 3. **Prompt Template Creation**
Create a **Prompt Template** associated with the General CRM topic:

#### Template Name
`Contact Verification Prompt`

#### Description
Guides the Agent to identify duplicate Contacts by name, email, or phone, and to generate a verification email if needed.

#### Template Text
```text
You are the Contact Guardian Agent.
Before creating a new Contact, check Salesforce for existing records with the same name, email, or phone.
- If exact or probable matches exist, present them clearly with their associated Account names.
- Ask: "Would you like me to draft a quick verification email to confirm if this contact is the same person?"
- If user says "Yes," call the `DraftOrReviseEmail` action with prefilled recipient and subject.
- If user confirms new, call `CreateContact`.
- Always log the outcome, avoiding internal record IDs and respecting field-level security.
```

#### Input Variables
| Variable | Description |
|-----------|--------------|
| `contactName` | Name entered by user |
| `email` | Contact email |
| `phone` | Contact phone |
| `accountName` | Intended Account for association |

#### Output
- `duplicateContacts` (list)
- `selectedAction` (use existing / relate / create new)
- `verificationEmailDraft` (string)

---

### 4. **Flow Setup (Duplicate Check Flow)**
Use **Flow Builder** to create `afcCheckDuplicateContact`.

#### Steps:
1. **Start:** Triggered by `Agent Quick Action` or user input.
2. **Input:** Collect `Name`, `Email`, `Phone`, `AccountId`.
3. **Decision Node:**
   - If `Match Found` → Display related records.
   - Else → Continue to create new Contact.
4. **Screen Element:** Show potential duplicates with Account affiliation.
5. **Action Element (Optional):** Call `DraftOrReviseEmail` to generate verification message.
6. **Log Outcome:** Capture `SelectedOption`, `Timestamp`, and `UserId` for audit.

#### Output Variables
- `duplicateCount`
- `decision`
- `verifiedContactId`

---

### 5. **Action Configuration**
Attach these **Agent Actions** to the topic:

| Action Name | Purpose | Requires User Confirmation |
|--------------|----------|-----------------------------|
| `Get Record Details` | Retrieve Contact or Account details | No |
| `Query Records` | Find matching Contacts by name/email/phone | No |
| `DraftOrReviseEmail` | Generate verification email for duplicate | Yes |
| `CreateContact` | Create new Contact record if verified as new | No |
| `Contact Verification Email` | Specialized variant of `DraftOrReviseEmail` prefilled for verification | Yes |

---

### 6. **Email Draft Template**
When invoking `DraftOrReviseEmail`, use this message pattern:

**Subject:** `Quick verification: Were you previously with {{AccountName}}?`  
**Body:**
```
Hi {{FirstName}},

We noticed a contact record that may match your information in our system. 
Can you confirm if you were previously affiliated with {{AccountName}} or another organization?

Please reply to confirm or update your preferred details.

Thanks,  
{{AgentName}}  
Contact Guardian | Salesforce CRM Team
```

---

### 7. **Testing and Validation**
Use the **Agent Testing Center** or sandbox environment.

#### Validate:
- [ ] Duplicate Contacts detected accurately (≥90% exact matches)
- [ ] Verification email correctly drafted on consent
- [ ] Contact creation happens only after confirmation
- [ ] All responses logged (FLS-safe)
- [ ] p95 response time ≤10 seconds

#### KPIs:
| Metric | Target |
|--------|--------|
| Duplicate prevention accuracy | ≥90% |
| False positives (email) | ≤1% |
| Duplicate reduction | ≥40% |
| Decision latency (p95) | ≤10s |

---

### 8. **Deployment**
- **Environments:** Start in Sandbox → UAT → Production
- **Pilot Group:** 8–10 internal users (Sales & Ops)
- **Rollout Goal:** Validate user adoption and performance metrics
- **Telemetry:** Log agent decisions and verification outcomes to Data Cloud for analysis

---

## 📊 Success Criteria Summary
| Category | Success Metric |
|-----------|----------------|
| Detection | ≥90% correct duplicate matches |
| User Experience | ≤1 clarifying question |
| Accuracy | ≤1% false positives |
| Reduction | ≥40% fewer duplicate contacts |
| Compliance | 100% FLS/sharing honored |
| Latency | p95 ≤10s |

---

## 👥 Responsible Parties
| Role | Owner |
|------|--------|
| Product Owner | METUS Project Lead |
| Salesforce Admin | CRM Admin |
| Technical Architect | Marcus Neal |
| Agentforce Builder | Salesforce TA Team |
| Data Steward | Sales Ops |
| QA/UAT | Support Team |

---

## 🧠 References
- *METUS Use Case Deck (Nov 5, 2025)*
- *METUS Agentforce Data Cloud POC Document*
- *Salesforce Agentforce Builder Documentation*
- *Data Cloud Integration Guide*

---

**© 2025 METUS & Salesforce Collaboration Team**  
_“Guard your Contacts. Verify before you create.”_
