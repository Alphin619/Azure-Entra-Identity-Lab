# MITRE ATT&CK Mapping - AxiomCorp Identity Controls

## Overview

This document maps each security control implemented in the 
AxiomCorp Entra ID lab to the MITRE ATT&CK technique it 
mitigates. The purpose is to contextualise identity security 
controls within the real threat landscape and validate that 
the access architecture addresses known attacker behaviours 
rather than implementing controls arbitrarily.

MITRE ATT&CK Framework reference: https://attack.mitre.org

---

## Control-to-Technique Mapping

| Control Implemented | MITRE Technique | Technique ID | Tactic | Why It Mitigates It |
|---|---|---|---|---|
| Tiered RBAC model with least privilege | Valid Accounts | T1078 | Defence Evasion, Persistence | Limits what an attacker can do post-compromise based on which tier is breached. A compromised low-tier account cannot reach high-value resources |
| Reader-only role for SOC analysts | Valid Accounts - Cloud Accounts | T1078.004 | Persistence | Compromised SOC account cannot modify resources, create backdoors, or alter access controls |
| MFA enforced on Tier 1 and Tier 2 | Brute Force | T1110 | Credential Access | Valid credentials alone are insufficient and attacker must also compromise the second factor |
| MFA enforced on Tier 1 and Tier 2 | Phishing | T1566 | Initial Access | Credential harvesting via phishing is neutralised if MFA is required |
| No permanent Global Admin assignment | Account Manipulation | T1098 | Persistence | Break-glass account not in daily use reduces persistent privileged access surface |
| No permanent Global Admin assignment | Create Account | T1136.003 | Persistence | Limits ability to create persistent cloud backdoor accounts |
| Tier 3 assigned no Azure RBAC roles | Permission Groups Discovery | T1069.003 | Discovery | Standard users cannot enumerate cloud groups, roles, or resource structure |
| Block legacy authentication (design) | Valid Accounts | T1078 | Defence Evasion | Closes MFA bypass via IMAP, POP3, and basic auth protocols |
| Group-based access assignment only | Valid Accounts | T1078 | Persistence | Access is tied to group membership and removing a user from a group revokes all associated permissions immediately |
| Break-glass account isolation | Privilege Escalation | TA0004 | Privilege Escalation | Emergency account not usable for routine access, minimising exposure |
| Conditional Access - block high-risk sign-ins (design) | Valid Accounts | T1078 | Defence Evasion | Real-time risk signals block compromised credential use before authentication completes |
| Conditional Access - MFA outside UK (design) | Valid Accounts - Cloud Accounts | T1078.004 | Persistence | Challenges sign-ins from unexpected geographies consistent with credential theft |

---

## Techniques Addressed by Tier

### Tier 0 - Break-Glass
- T1078 - Valid Accounts (persistent privileged access minimised)
- T1098 - Account Manipulation (no permanent assignment)
- T1136.003 - Create Cloud Account (limited by isolation)

### Tier 1 - SOC Analyst
- T1110 - Brute Force (MFA enforced)
- T1078.004 - Cloud Accounts (Reader-only limits post-compromise value)
- T1069.003 - Cloud Groups (no write access to enumerate or modify groups)

### Tier 2 - IT Ops
- T1110 - Brute Force (MFA enforced)
- T1078.004 - Cloud Accounts (Contributor scope limited to non-critical RGs in production)
- T1098 - Account Manipulation (cannot modify RBAC assignments)

### Tier 3 - Standard Users
- T1069.003 - Cloud Groups (no Azure Portal access)
- T1078 - Valid Accounts (no Azure resource access even with valid credentials)

---

## Residual Risks

The following techniques are not fully addressed by the 
current control set and represent areas for further hardening:

| Technique | ID | Gap | Recommended Mitigation |
|---|---|---|---|
| Adversary-in-the-Middle | T1557 | MFA can be bypassed via AiTM phishing proxies (e.g. Evilginx) | Phishing-resistant MFA such as FIDO2 keys or certificate-based auth. Requires Entra ID P1 |
| Steal Application Access Token | T1528 | OAuth token theft bypasses MFA entirely | Token lifetime policies and Continuous Access Evaluation. Requires P1 |
| Multi-Factor Authentication Request Generation | T1621 | MFA fatigue attacks, attacker spams MFA prompts until user approves | Number matching and additional context in MFA prompts. Configurable in Entra ID free tier |

---

## Notes on Free Tier Constraints

Some controls in this mapping are design specifications rather 
than enforced policies due to free tier licensing limitations. 
These are clearly marked as (design) in the mapping table above. 
The residual risk section identifies where these gaps leave 
exposure and what the production mitigation would be.
