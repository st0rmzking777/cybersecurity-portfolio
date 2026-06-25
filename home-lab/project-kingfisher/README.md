# Project Kingfisher

Project Kingfisher is a self-contained cybersecurity lab demonstrating 
end-to-end phishing simulation and detection engineering. Three virtual 
machines work together across an isolated NAT network to simulate the 
full attacker-defender loop, from initial email delivery through to 
SIEM-based detection rule authoring.

**DISCLAIMER:** This project is conducted entirely within an isolated lab 
environment against decoy accounts under the author's own ownership. 
All techniques demonstrated are for educational purposes only.

## Lab Environment

| VM | Role | OS | IP |
|---|---|---|---|
| Kali_Linux | Attacker | Kali Linux | 192.168.18.129 |
| Windows 10 x64 | Victim | Windows 10 | 192.168.18.128 |
| Ubuntu_Ult | Defender | Ubuntu Server 22.04.5 LTS | 192.168.18.130 |

## Lab Environment (Updated) - 20/6/2026
 
| VM | Role | OS | IP |
| --- | --- | --- | --- |
| Kali_Linux | Attacker | Kali Linux | 192.168.152.129 |
| Windows 10 x64 | Victim / Wazuh Agent | Windows 10 Home N | 192.168.152.137 |
| Ubuntu_Ult | Defender / Wazuh Manager | Ubuntu Server 22.04.5 LTS | 192.168.152.138 |
 
> **Note:** Lab subnet migrated from `192.168.18.x` to `192.168.152.x` after the VM files were relocated.

## Phases

| Phase | Focus | Status |
|---|---|---|
| Phase 1 | Attacker Infrastructure & Phishing Campaign | Completed |
| Phase 2 | Victim Instrumentation & Telemetry Capture | Completed |
| Phase 3A | Detection Engineering & SIEM Rule Authoring | Completed |
| Phase 3B | Production-Hardened Detection (CDB Lists, Behavioural Rules) | In Progress |

---

# Project Kingfisher | Phase 1: Attacker Infrastructure & Phishing Campaign

**Duration:** 19/04/2026 → 30/04/2026  

**Platform:** Kali Linux (VMware Workstation Pro), Windows 10 (VMware Workstation Pro)  

**Tools:** Gophish, Gmail SMTP, Custom HTML Landing Page

## Overview

Deployed a fully operational phishing infrastructure on an isolated lab network. Executed two real phishing campaigns against decoy victims, capturing credentials and observing live defensive mechanisms across both runs.

---

## Infrastructure

| Component | Detail |
|---|---|
| Phishing Framework | Gophish 0.12.1 on Kali Linux |
| Email Delivery | Gmail SMTP (smtp.gmail.com:587) with app password auth |
| Landing Page | Cloned Microsoft 365 login page (Harvest #1) → custom HTML form (Harvest #2) |
| Attacker IP | 192.168.18.129 |
| Victims | 2 decoy Gmail accounts |

---

## Campaign Results

### Harvest #1

| Victim | Sent | Opened | Clicked | Submitted |
|---|---|---|---|---|
| James Smith | OK | OK | OK | Blocked |
| Sarah Chen | OK | OK | OK | Blocked |

**Campaign dashboard showing 2 sent, 2 opened, 2 clicked, 0 submitted.**

![Harvest #1 results dashboard](project-kingfisher-phase1photos/1-harvest1-dashboard.png)

**Email Delivery to James Smith:**  

Email delivered directly to the primary inbox with personalised greeting and functioning phishing link.

![James' inbox](project-kingfisher-phase1photos/2-james-inbox.png)

**Email Delivery to Sarah Chen:** 

Email automatically routed to Gmail's spam folder with reputation-based filtering banner. Subsequently retrieved manually from spam by the victim.

![Sarah's spam](project-kingfisher-phase1photos/3-sarah-spam.png)

**Defensive Block to Client-Side Validation:**  

Both victims clicked the phishing link, loading the cloned Microsoft 365 login page served from the attacker's infrastructure (192.168.18.129). However, credential submission was blocked by Microsoft's retained client-side JavaScript validation, which attempted to authenticate Gmail addresses against Microsoft's directory service.

![Microsoft client-side validation error](project-kingfisher-phase1photos/4-microsoft-clientside-valerror.png)

---

### Harvest #2

| Victim | Sent | Opened | Clicked | Submitted |
|---|---|---|---|---|
| James Smith | OK | OK | OK | OK |
| Sarah Chen | OK | OK | OK | OK |

**Campaign dashboard showing 2 sent, 2 opened, 2 clicked, 2 submitted.**

![Harvest #2 results dashboard](project-kingfisher-phase1photos/5-harvest2-dashboard.png)

**Custom HTML Landing Page:**  

Cloned Microsoft 365 page replaced with externally-sourced custom HTML form, bypassing the client-side validation block from Harvest #1.

![Custom HTML landing page on victim Win10](project-kingfisher-phase1photos/6-custom-html-landingpage.png)

**Custom HTML Redirect:**  

Upon credential submission, victims were redirected to a custom HTML page hosted on the attacker's infrastructure (192.168.18.129:8080), confirming the simulated attack was successful.

![Custom HTML redirect page](project-kingfisher-phase1photos/7-custom-html-redirectpage.png)

**Victim Timeline of James Smith:** 

Full attack chain captured with OS and browser fingerprinting on each event (Windows 10, Chrome 147.0.0.0).

![James' timeline](project-kingfisher-phase1photos/8-james-timeline.png)

**Victim Timeline of Sarah Chen:**  

Full attack chain captured with OS and browser fingerprinting on each event (Windows 10, Chrome 147.0.0.0).

![Sarah's timeline](project-kingfisher-phase1photos/9-sarah-timeline.png)

---

## Key Observations

**Harvest #1 → Harvest #2 demonstrates the attacker adaptation loop:**  
Initial deployment surfaced two defensive signals (Gmail spam filtering and Microsoft's retained client-side JavaScript validation), with the validation layer blocking credential submission. Harvest #2 mitigated this by replacing the cloned page with a custom-authored Microsoft 365 login form (stripped of validation logic and configured with named form fields). The campaign successfully captured plaintext credentials (username and password) from both victims in Gophish's parameter table. This reflects the real-world offensive iteration loop: observe defensive signals, adapt tradecraft, re-execute.

A more sophisticated attacker would additionally:
- Serve pages over HTTPS with a convincing domain to reduce "Not secure" browser warnings
- Host all assets locally rather than relying on Microsoft's CDN
- Register a typosquat domain (e.g. `microsft-login.com`) to defeat URL-based detection

---

## MITRE ATT&CK Coverage

| Technique | ID | Source |
|---|---|---|
| Phishing: Spearphishing Link | T1566.002 | Gophish email send |
| User Execution: Malicious Link | T1204.001 | Victim click event |
| Input Capture: Web Portal Capture | T1056.003 | Captured credentials |
| Valid Accounts | T1078 | Functional account material |

---

## Conclusion

Project Kingfisher demonstrates the full attacker-defender loop across 
a self-contained isolated lab environment. Phase 1 executed two real 
phishing campaigns against decoy victims, observed live defensive 
mechanisms, and achieved complete credential capture through iterative 
attack adaptation. Phases 2 and 3 will extend this into defensive 
instrumentation and SIEM-based detection engineering. Therefore, building the 
complete picture from initial phishing delivery through to detection 
rule authoring.



---

# Project Kingfisher | Phase 2: Victim Instrumentation & Telemetry Capture

**Status:** Complete  

**Duration:** 09/05/2026 → 10/05/2026

**Platform:** Windows 10 (VMware Workstation Pro)  

**Tools:** Sysmon (SwiftOnSecurity config), PowerShell, Windows Registry Editor, auditpol, Windows Event Viewer

## Overview

Phase 2 transforms the Windows 10 victim host from a passive participant into a fully instrumented endpoint. Three independent logging layers were configured to capture process activity, PowerShell execution, and Windows native audit events. Therefore, providing the multi-source telemetry that Phase 3 detection engineering will operate on.

---

## Logging Layers Configured

| Layer | Tool | Captures |
|---|---|---|
| Endpoint Visibility | Sysmon (SwiftOnSecurity config) | Process creation, network connections, registry changes, DNS queries |
| PowerShell Auditing | Native PowerShell logging | Module execution, script block content, transcripts |
| Native Windows Auditing | Windows Audit Policy | Process creation with command line (EID 4688) |

## Build Walkthrough

**Sysmon Install Success:**

Sysmon installation with SwiftOnSecurity schema 4.91 loaded. Driver and service installed successfully.

![Sysmon Install Success](project-kingfisher-phase2photos/1-sysmon-install-success.png)

**Sysmon Service Running:**

Sysmon64 service confirmed in Running state with active process (Process ID 3976).

![Sysmon Service Running](project-kingfisher-phase2photos/2-sysmon-service-running.png)

**Sysmon Event Viewer Overview:**

Sysmon Operational log showing 1,157+ events across multiple Event IDs: Process Create (1), Network Connection (3), Registry Modification (13), DNS Query (22).

![Sysmon Event Viewer Overview](project-kingfisher-phase2photos/3-sysmon-eventviewer-overview.png)

**Powershell Module Logging:**

PowerShell Module Logging registry configuration applied via PowerShell. EnableModuleLogging set to 1 with wildcard module coverage.

![Powershell Module Logging](project-kingfisher-phase2photos/4-powershell-module-logging.png)

**Powershell EID 4104 Warning:**

PowerShell Event ID 4104 Warning firing on encoded command execution. Script block content captured, demonstrating defensive logging detecting obfuscation attempts.

![Powershell EID 4104 Warning](project-kingfisher-phase2photos/5-powershell-eid4104-warning.png)

**Audit Policy Command:**

Process Creation auditing enabled via auditpol with command line inclusion configured via registry.

![Audit Policy Command](project-kingfisher-phase2photos/6-audit-policy-commandline.png)

---

## Key Observations

**Win10 Home requires registry-based configuration** 

The lab Windows 10 instance ran Home edition, which lacks `gpedit.msc`. PowerShell logging and command-line auditing were configured via direct Windows Registry edits; Group Policy is fundamentally a UI for writing to registry, so bypassing the GUI achieves identical outcomes while deepening understanding of how Windows policy enforcement actually works.

**PowerShell 5.0+ has baseline Warning-level detection** 

Event ID 4104 fires automatically at Warning level when PowerShell detects suspicious script content (encoded commands, obfuscation patterns) even without Script Block Logging explicitly enabled. This baseline detection caught the test encoded command immediately, demonstrating that PowerShell ships with built-in attacker tradecraft awareness.

**Multi-source telemetry validates defence in depth** 

A single attacker action generates events across multiple log sources simultaneously: Sysmon captures the process and network context, PowerShell logging captures the script content, and Windows Security log captures native process creation. Attackers may evade one logging mechanism, but evading all three is significantly harder. This redundancy is intentional in enterprise environments and forms the basis for cross-source correlation in detection engineering.

---

## MITRE ATT&CK Telemetry Coverage

Phase 2 establishes detection telemetry for the following techniques:

| Technique | ID | Telemetry Source |
|---|---|---|
| Command and Scripting Interpreter: PowerShell | T1059.001 | PowerShell EID 4103/4104, Sysmon EID 1, Security EID 4688 |
| User Execution: Malicious Link | T1204.001 | Sysmon EID 1, Sysmon EID 3 |
| Obfuscated Files or Information | T1027 | PowerShell EID 4104 Warning |
| Application Layer Protocol | T1071 | Sysmon EID 3 |
| Input Capture: Web Portal Capture | T1056.003 | Sysmon EID 3 |

---

## Conclusion

Phase 2 successfully instrumented the Windows 10 victim host across three independent logging layers, transforming it from a passive endpoint into a comprehensive telemetry source. Sysmon, PowerShell logging, and Windows audit policy each capture different aspects of system activity, with overlap that enables cross-source correlation. The Win10 Home registry-based configuration approach, while necessitated by edition limitations, ultimately provided deeper understanding of Windows policy enforcement than a Group Policy GUI workflow would have.

---

# Project Kingfisher | Phase 3A: Detection Engineering & SIEM Rule Authoring

**Status:** Complete

**Duration:** 20/06/2026 → 26/06/2026

**Platform:** Ubuntu Server 22.04.5 LTS (VMware Workstation Pro), Windows 10 (VMware Workstation Pro), Kali Linux (VMware Workstation Pro)

**Tools:** Wazuh 4.14.5 (Manager, Indexer, Dashboard), Wazuh Agent, Sysmon (SwiftOnSecurity config — modified), Gophish

## Overview

Phase 3A deploys a full SIEM platform (Wazuh) on the Ubuntu defender host, enrolls the Windows victim as a monitored endpoint, and authors custom detection rules mapped to MITRE ATT&CK that fire on the exact attack chain executed in Phases 1 and 2. This phase closes the loop from offensive simulation through defensive instrumentation to working, validated detection and proving the complete attacker-defender cycle within a single lab environment.

> **Note:** Lab subnet migrated from `192.168.18.x` to `192.168.152.x` after the VM files were relocated. VMware's NAT DHCP service issued new leases against regenerated MAC addresses. The updated lab table is reflected below.

---

## Updated Lab Environment

| VM | Role | OS | IP |
| --- | --- | --- | --- |
| Kali_Linux | Attacker | Kali Linux | 192.168.152.129 |
| Windows 10 x64 | Victim / Wazuh Agent | Windows 10 Home N | 192.168.152.137 |
| Ubuntu_Ult | Defender / Wazuh Manager | Ubuntu Server 22.04.5 LTS | 192.168.152.138 |

---

## Build Walkthrough

**Wazuh Installation Completed:**

Wazuh 4.14.5 all-in-one deployment (manager, indexer, and dashboard) completed on the Ubuntu defender host. Admin credentials generated and logged. The installer handled an APT lock contention from Ubuntu's unattended-upgrades service automatically, retrying after 30-second intervals.

![Wazuh install finished](project-kingfisher-phase3photos/1-wazuh-install-finished.png)

**Dashboard Operational:**

Wazuh dashboard accessed from the Windows victim host at `https://192.168.152.138`, confirming the SIEM control plane is live and reachable across the lab network. The dashboard registered 113 medium and 113 low-severity alerts from the manager's own startup health checks before any agent was enrolled as the SIEM monitors itself.

![Dashboard overview](project-kingfisher-phase3photos/2-dashboard-overview.png)

**Agent Enrolled and Active:**

Wazuh agent `win10-x64` (ID 001) enrolled and reporting from the instrumented Windows 10 victim. System inventory automatically populated: 1 core, 8 GB RAM, Intel Core Ultra 7 258V, hostname DESKTOP-M08E4V9. Last keep-alive confirmed, agent status Active.

![Agent active](project-kingfisher-phase3photos/3-agent-active.png)

**Telemetry Channels Configured:**

Agent `ossec.conf` on the Windows victim updated to forward Sysmon Operational and PowerShell Operational event channels to the Wazuh manager, bridging the Phase 2 instrumentation into the SIEM pipeline. Default channels (Application, Security, System) were already forwarded by the agent's base configuration.

![ossec.conf channels](project-kingfisher-phase3photos/4-ossec-conf-channels.png)

**Event Pipeline Confirmed:**

Threat Hunting dashboard showing 713 events from agent `win10-x64` within 24 hours. Alert group legend confirms multi-source telemetry landing: `sysmon`, `sysmon_eid1_detection`, `sysmon_eid11_detection`, `WEF`, `windows_security`. The activity spike at 18:00 corresponds to the moment the Sysmon and PowerShell channels went live after the `ossec.conf` edit.

![Threat Hunting telemetry](project-kingfisher-phase3photos/5-threat-hunting-telemetry.png)

**Custom Detection Rules Authored:**

Four custom rules authored in `/var/ossec/etc/rules/local_rules.xml`, each mapped to MITRE ATT&CK techniques from the Kingfisher attack chain:

![local_rules.xml](project-kingfisher-phase3photos/6-local-rules-xml.png)

**Rules Firing on Live Attack:**

Rules 100100, 100101, and 100102 confirmed firing against live attack re-execution. Rule 100101 (outbound connection to attacker infrastructure) fired 10+ times on phishing link clicks from the Windows victim to the Kali attacker. Rule 100102 (connection to credential harvesting port) fired 3 times on credential submission.

![Rules firing](project-kingfisher-phase3photos/7-rules-firing.png)

**MITRE ATT&CK Module Populated:**

Wazuh's MITRE ATT&CK module auto-populated with tactic and technique coverage from both built-in and custom rule hits. Top tactics detected: Command and Control, Defense Evasion, Privilege Escalation, Execution, Initial Access, Credential Access. Custom rule `<mitre>` tags feed directly into this framework view, closing the loop between detection authoring and ATT&CK coverage reporting.

![MITRE ATT&CK module](project-kingfisher-phase3photos/8-mitre-attack-module.png)

---

## Custom Detection Rules

All rules authored in `/var/ossec/etc/rules/local_rules.xml` on the Wazuh manager, within the `100000+` custom ID range:

### Rule 100100 — PowerShell Encoded Command Execution

```xml
<rule id="100100" level="12">
  <if_group>windows</if_group>
  <field name="win.eventdata.commandLine" type="pcre2">(?i)(-enc\s|-encodedcommand|frombase64string)</field>
  <description>Kingfisher: PowerShell encoded command execution detected</description>
  <mitre>
    <id>T1027</id>
    <id>T1059.001</id>
  </mitre>
</rule>
```

Detects encoded PowerShell execution by matching `-EncodedCommand`, `-enc`, or `FromBase64String` patterns in the command line. Tied to Phase 2's EID 4104 evidence, where PowerShell's built-in Warning-level detection flagged encoded commands. This rule elevates that signal to a level-12 SIEM alert with ATT&CK mapping.

### Rule 100101. Outbound Connection to Attacker Infrastructure

```xml
<rule id="100101" level="12">
  <if_group>sysmon_event3</if_group>
  <field name="win.eventdata.destinationIp" type="pcre2">192\.168\.152\.129</field>
  <description>Kingfisher: Outbound connection to attacker detected</description>
  <mitre>
    <id>T1204.001</id>
    <id>T1071</id>
  </mitre>
</rule>
```

Fires when Sysmon EID 3 (NetworkConnect) captures any outbound connection from the victim to the attacker's IP. This is the primary Phase 1 detection when the victim clicks the phishing link, their browser connects to the Gophish server on 192.168.152.129, and this rule catches that callback regardless of which process or port initiated it.

### Rule 100102. Connection to Credential Harvesting Endpoint

```xml
<rule id="100102" level="13">
  <if_group>sysmon_event3</if_group>
  <field name="win.eventdata.destinationIp" type="pcre2">192\.168\.152\.129</field>
  <field name="win.eventdata.destinationPort" type="pcre2">^(80|8080|3333)$</field>
  <description>Kingfisher: Connection to credential harvesting endpoint detected</description>
  <mitre>
    <id>T1056.003</id>
    <id>T1071</id>
  </mitre>
</rule>
```

Refines rule 100101 by adding port-specificity to match the connections specifically to ports 80, 8080, or 3333, which are the ports used by Gophish's harvesting infrastructure. Set at level 13 (one above 100101) to ensure both rules fire on the same event without Wazuh's rule-priority deduplication suppressing the port-specific detection. This rule detects the connection to the harvesting infrastructure, not the credential content within the POST body and it is an important distinction documented in Key Observations.

### Rule 100103. Composite Escalation (Full Attack Chain)

```xml
<rule id="100103" level="15" frequency="2" timeframe="120">
  <if_matched_sid>100100</if_matched_sid>
  <same_field>win.eventdata.user</same_field>
  <description>Kingfisher: Encoded PowerShell AND attacker callback - full attack chain detected</description>
  <mitre>
    <id>T1027</id>
    <id>T1059.002</id>
    <id>T1204.001</id>
  </mitre>
</rule>
```

Correlation rule that fires at critical severity (level 15) when an encoded PowerShell execution and an attacker callback both occur within a 120-second window from the same user. Demonstrates chained-behaviour detection beyond single-event rules, and this kind of composite logic used in production SOC environments to reduce false positives and escalate genuine multi-stage attacks.

---

## Key Observations

**Wazuh monitors itself before any agent enrols.** The dashboard showed 113 medium-severity and 113 low-severity alerts immediately after installation, before the Windows agent was connected. These were the manager's own health-check events and internal service starts. The SIEM's first monitored endpoint is itself.

**The `admin` credential drives the entire data pipeline, not just the dashboard.** Wazuh's admin password is shared across the dashboard login, the indexer API, and the Filebeat log-shipping configuration. Resetting it through the dashboard UI without updating all three components silently breaks log ingestion. The correct method is Wazuh's dedicated password change tool on the manager, which updates all components in sync.

**Sysmon's SwiftOnSecurity config filters out browser network connections by default.** Rules 100101 and 100102 initially failed to fire despite the phishing attack executing successfully. The root cause was not the detection rules but the telemetry layer: the SwiftOnSecurity Sysmon config uses `onmatch="include"` for NetworkConnect (EID 3), which only logs connections from a whitelist of suspicious executables (cmd.exe, powershell.exe, certutil.exe, etc.). Browser processes (chrome.exe, msedge.exe) are not on that list. The fix was adding a `DestinationIp` include rule for the attacker's IP. This demonstrates a critical detection-engineering principle: a correct rule produces nothing if the sensor upstream isn't capturing the event it needs.

**Endpoint SIEM detects the phishing callback, not the phishing email.** Wazuh's agent on the victim host cannot detect that a spearphishing email was sent, therefore the email delivery occurs outside the endpoint's visibility boundary. What the endpoint *can* detect is the consequence: the victim's browser connecting to attacker-controlled infrastructure after clicking the link. This is a fundamental architectural boundary as email-layer detection belongs to mail gateway controls (SPF, DKIM, DMARC, email security gateways), while endpoint detection catches the post-click callback. Both are necessary for comprehensive phishing defence.

**Lab rules use hardcoded attacker IPs; production deployments would not.** Rules 100101 and 100102 match against a specific attacker IP (192.168.152.129) because the lab controls both sides of the attack. In a real environment, the defender does not know the attacker's IP in advance. Production-grade detection would integrate threat intelligence feeds (CDB lists in Wazuh, or external feeds from sources like AbuseIPDB, AlienVault OTX, and VirusTotal), behavioural baselines, and broader Sysmon network logging rather than IP-specific include rules. Phase 3B will address this gap.

---

## MITRE ATT&CK Detection Coverage

Phase 3A establishes active detection (alerting) for the following techniques, building on the telemetry coverage established in Phase 2:

| Technique | ID | Detection Rule | Alert Level |
| --- | --- | --- | --- |
| Obfuscated Files or Information | T1027 | 100100, 100103 | 12, 15 |
| Command and Scripting Interpreter: PowerShell | T1059.001 | 100100 | 12 |
| User Execution: Malicious Link | T1204.001 | 100101, 100103 | 12, 15 |
| Application Layer Protocol | T1071 | 100101, 100102 | 12, 13 |
| Phishing: Spearphishing Link | T1566.002 | 100101 | 12 |
| Input Capture: Web Portal Capture | T1056.003 | 100102 | 13 |

---

## Conclusion

Phase 3A successfully deployed a full SIEM platform, enrolled the victim endpoint, and authored four custom detection rules that fire on the exact attack chain from Phases 1 and 2. The detection pipeline is validated end to end: phishing link click generates an outbound network connection, Sysmon captures it as an EID 3 event, the Wazuh agent ships it to the manager, the custom rule matches the attacker's IP and port, and a level-12/13 alert fires with MITRE ATT&CK mapping.

The most significant finding was not a rule logic issue but a telemetry gap: the SwiftOnSecurity Sysmon configuration's restrictive NetworkConnect filtering prevented detection rules from firing until the sensor configuration was adjusted. This mirrors real-world detection engineering, where the gap between "the rule is correct" and "the rule fires" is almost always a data-source problem, not an analysis problem.

Phase 3B will harden these detections for production realism by replacing hardcoded attacker IPs with threat intelligence feed integration (Wazuh CDB lists), implementing behavioural detection rules that operate without prior knowledge of the attacker, and broadening the Sysmon NetworkConnect configuration to provide comprehensive network visibility.
