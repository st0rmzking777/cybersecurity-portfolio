# Security Advisory Case Study

## SMB Workplace Security Assessment

**Date:** 24 May 2026 -> 26 May 2026

**Role:** Informal Security Advisor (Volunteer)

**Status:** Anonymised for portfolio purposes

---

## Overview

A small to medium business (SMB) was preparing to deploy an internally developed workforce management application to track casual staff shifts and payments. The application was being built by an external developer with limited independent security oversight prior to launch.

After a colleague raised concerns about the security of an upcoming internal application, I took it upon myself to assess the situation from a cybersecurity standpoint. This case study covers the risks I identified, the threat modelling I performed, the ethical decisions I stood by, and how I ensured everything was escalated through the right channels.

---

## Context

- **Business type:** A fitness business with a strong local and community presence
- **Application purpose:** Internal workforce time tracking and shift management
- **Developer:** External contractor with no third-party security validation
- **Stakeholders:** Business owner (privileged access), middle management, casual staff
- **Sensitive data handled:** Employee identity records, shift logs, payment records
- **Deployment status at time of review:** Pre-launch

---

## Risks Identified

### Application Risks
- New build with no independent security testing
- Internal-only testing conducted by the developer's own team (bias risk)
- Unknown encryption status for sensitive data at rest and in transit
- No confirmed authentication architecture or MFA enforcement
- No documented audit logging or tamper-evident records
- No defined incident response or breach notification process

### Compliance Risks
- Privacy Act 1988 considerations for employee record storage
- Mandatory data breach notification obligations under the Notifiable Data Breaches scheme
- Employment law compliance considerations for record keeping

### Operational Risks
- Information leak: the developer publicly disclosed that the business owner held privileged access; telegraphing the highest-value target to potential attackers
- Limited cybersecurity awareness across general staff
- Weak password hygiene observed on existing systems
- Absence of MFA on key accounts
- No Zero Trust policy in operation

---

## Threat Modelling

Informal threat modelling was performed using attacker-perspective reasoning.

**Key finding:** The business owner represents the highest-value target.

The "king in the castle" principle applies; whoever holds privileged access to the application becomes the priority target for any attacker seeking maximum impact. This was reinforced by the developer's casual disclosure of the owner's access level, which constituted an unintentional reconnaissance leak.

### Likely Attack Paths
- Credential compromise targeting the business owner via phishing or password reuse
- Social engineering of staff members with privileged contacts
- Direct application exploitation if vulnerabilities exist in the new code

### MITRE ATT&CK Mapping
- **Initial Access:** T1566 (Phishing)
- **Credential Access:** T1078 (Valid Accounts)
- **Privilege Escalation:** T1078.004 (Cloud Accounts) where applicable
- **Collection:** T1213 (Data from Information Repositories)

---

## Recommendations Provided

### Primary Recommendation
- **Third-party penetration testing** by an independent qualified provider before deployment
- Written authorisation, scope, and statement of work must be issued by the business owner directly

### Secondary Recommendations
- **Bitwarden** for centralised password management among managers
- **Brave browser** for privacy-focused day-to-day operations, with uBlock Origin if otherwise
- Mandatory **MFA** on any account with privileged access
- **Encryption of sensitive fields** at rest with encrypted backups stored separately
- **Tamper-evident audit logging** with a defined retention policy
- **HTTPS enforcement** and certificate management
- **Defined incident response plan** with breach notification procedure
- **Compliance review** of underlying business processes

---

## Ethical Decisions Made

### Declined Unauthorised Testing
I was informally asked to perform penetration testing on the application by parties who did not have legal authority to authorise such testing. I declined for the following reasons:

- Under the **Cybercrime Act 2001 (Cth)**, unauthorised access or modification of data carries serious legal consequences
- Verbal permission from a developer or middle management does not constitute legal authorisation — only the asset owner can authorise testing in writing
- Unauthorised testing, even with informal consent, would constitute a serious professional and legal breach
- Future career credibility in cybersecurity requires strict adherence to authorised testing only

I recommended that all testing be commissioned formally through the business owner with a written scope of work and engagement letter.

### Declined Social Engineering Shortcut
A suggestion was made to approach the developer under a pretext to test the application's defences. I declined because social engineering, even in authorised penetration tests, requires explicit scope and consent. Acting on the suggestion would have constituted deception, not testing.

---

## Communication & Escalation Path

- Recommendations were communicated verbally to middle management
- Formal escalation was made through a management contact to the business owner
- I positioned myself as a security advisor, not as a decision-maker or implementer
- All recommendations were framed in business-impact language for non-technical audiences
- Attacker-perspective framing was kept internal; upward communication used risk and protection language

---

## Outcome

The business owner acknowledged the risks and engaged in dialogue regarding the recommendations. Discussions remain ongoing regarding implementation of secondary recommendations and formal commissioning of independent testing.

---

## Skills Demonstrated

- Threat modelling with MITRE ATT&CK framework
- Pattern recognition (identifying privileged-access information leakage)
- Professional ethics under social pressure
- Executive communication (translating technical risk into business language)
- GRC thinking (advocating for compliance over workaround)
- Stakeholder management and proper escalation
- Awareness of legal frameworks (Cybercrime Act, Privacy Act)

---

## Reflection

Real-world cybersecurity work is far less about dramatic technical action and more about judgement under pressure; navigating people, politics, ambiguity, and incomplete information while making sound calls. The hardest decision in this situation was not technical. It was declining a request from people I work with daily while still maintaining the relationships and credibility needed to keep advising them.

This case reinforced that authorisation, communication, and ethics are inseparable from technical work in security. Tools are taught. Judgement is built through situations like this.

---

*This document has been fully anonymised. All names, organisations, and identifying details have been removed or generalised.*
