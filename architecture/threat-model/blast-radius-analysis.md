# Blast Radius Analysis - AxiomCorp Identity Tier Model

## Overview

This document analyses the potential impact of a credential compromise 
at each identity tier within the AxiomCorp Entra ID environment. 
Blast radius analysis is a core threat modelling technique used to 
evaluate what an attacker can realistically achieve following an 
account compromise, and to validate that access controls are 
appropriately scoped.

Each tier is assessed against the following criteria:

- What can the attacker read or enumerate?
- What can the attacker create, modify, or delete?
- Can the attacker escalate privileges from this position?
- What does Entra ID log that could indicate compromise?
- What is the realistic detection window?
- What is the recommended response action?

---

## Tier 0 - Break-Glass Global Admin

**Account:** breakglass@alphinshibu2012gmail.onmicrosoft.com
**Role:** Global Administrator
**Assigned RBAC:** Full subscription access

### Impact if Compromised

| Category | Detail |
|---|---|
| Read access | Full visibility across entire Entra ID tenant and all Azure resources |
| Write access | Can create, modify, or delete any user, group, role, or resource |
| Privilege escalation | Not applicable, already the highest privilege level. Can create additional backdoor admin accounts |
| Lateral movement | Can access all connected Microsoft 365 services, all subscriptions, all resource groups |

### What Entra ID Logs

- Sign-in event from break-glass account (this should never appear under normal operations)
- Any admin activity under this account
- IP address and location of sign-in
- Device compliance status

### Detection Window

**Immediate** - any sign-in event from this account should trigger a 
high-priority alert. Under normal operations this account is never 
used, so any authentication event is an anomaly by definition. In 
Project 2 a detection rule will be built specifically for this.

### Severity Rating

**Critical** - full tenant and infrastructure compromise. 
Equivalent to owning the entire environment.

### Recommended Response

1. Immediately revoke all active sessions via Entra ID
2. Reset credentials and re-seal under break-glass procedure
3. Review all admin activity logs for the past 72 hours
4. Audit all user and group changes made during the compromise window
5. Escalate to security leadership immediately

### Controls Mitigating This Risk

- Account is not assigned to any standard RBAC group
- Password stored offline and not in a shared document or password manager
- No Conditional Access policies bypass this account
- Sign-in monitoring configured (Project 2)

---

## Tier 1 - Security Operations (SOC Analyst)

**Account:** soc.analyst1@alphinshibu2012gmail.onmicrosoft.com
**Role:** Reader at subscription scope
**Group:** AxiomCorp-Tier1-SOC

### Impact if Compromised

| Category | Detail |
|---|---|
| Read access | Full read visibility across all Azure resources, resource groups, and configurations within the subscription |
| Write access | None. Reader role prohibits all resource modification |
| Privilege escalation | Not possible via RBAC alone. Attacker cannot assign roles or modify access policies |
| Lateral movement | Attacker can enumerate entire resource structure, identify storage accounts, VMs, network configs. Useful for reconnaissance but cannot act on findings |

### What Entra ID Logs

- Sign-in events with IP, location, device, and client application
- Failed MFA challenges
- Resource enumeration activity visible in Azure Activity Log
- Any attempt to perform write operations (will fail and be logged)

### Detection Window

**Moderate** - a compromised Reader account may go undetected 
longer than a privileged account because no destructive actions 
are taken. The attacker's value from this account is 
reconnaissance. Detection relies on anomalous sign-in patterns 
such as unusual location, sign-in outside business hours, or 
impossible travel. MFA provides significant friction against 
initial compromise.

### Severity Rating

**Medium** - no direct damage capability, but full environment 
visibility gives an attacker a detailed map of the infrastructure 
for use in further attacks.

### Recommended Response

1. Revoke active sessions and disable account immediately
2. Review sign-in logs for the past 7 days and identify what was accessed and from where
3. Check Azure Activity Log for any enumeration patterns
4. Assess whether attacker obtained enough information to inform a follow-on attack
5. Re-enable account with fresh credentials after investigation

### Controls Mitigating This Risk

- Reader-only RBAC - no write capability under any circumstance
- MFA enforced on this account
- No Entra ID admin permissions assigned
- Sign-in anomaly detection via Entra ID (sign-in logs)

---

## Tier 2 - IT Operations

**Account:** it.ops1@alphinshibu2012gmail.onmicrosoft.com
**Role:** Contributor at subscription scope (lab). Production design scopes this to non-critical resource groups only.
**Group:** AxiomCorp-Tier2-ITOps

### Impact if Compromised

| Category | Detail |
|---|---|
| Read access | Full read visibility across all resources |
| Write access | Can create, modify, and delete resources across the subscription. Cannot modify RBAC role assignments |
| Privilege escalation | Cannot directly escalate via RBAC. Contributor does not include access management permissions. However an attacker could deploy a malicious VM or workload and use it as a foothold |
| Lateral movement | Can access and modify compute, storage, and networking resources. Could exfiltrate data from storage accounts, deploy backdoor resources, or disrupt services |

### What Entra ID Logs

- All resource creation and modification events in Azure Activity Log
- Sign-in events with full metadata
- Any IAM or role assignment attempts (will fail and be logged)

### Detection Window

**Fast to Moderate** - Contributor actions leave a clear audit 
trail in the Azure Activity Log. Unusual resource creation, 
unexpected deletions, or out-of-hours activity should trigger 
alerts. However an attacker who acts slowly and deliberately 
may blend into normal operational activity.

### Severity Rating

**High** - significant damage potential. A compromised IT Ops 
account can destroy resources, exfiltrate data, or deploy 
malicious infrastructure. Cannot directly compromise identity 
plane but poses serious risk to resource integrity.

### Recommended Response

1. Immediately revoke sessions and disable account
2. Audit Azure Activity Log for all actions taken under this account in the past 48 hours
3. Identify and roll back any unauthorised resource changes
4. Check for newly created resources that should not exist
5. Review storage account access logs for potential data exfiltration
6. Escalate based on scope of changes found

### Controls Mitigating This Risk

- Contributor role does not include access management as attacker cannot grant themselves additional privileges
- Production design scopes access to non-critical resource groups, limiting blast radius significantly
- MFA enforced on this account
- Azure Activity Log captures all resource operations

---

## Tier 3 - Standard Users

**Account:** standard.user1@alphinshibu2012gmail.onmicrosoft.com
**Role:** None - no Azure Portal access
**Group:** AxiomCorp-Tier3-StandardUsers

### Impact if Compromised

| Category | Detail |
|---|---|
| Read access | No Azure resource visibility and cannot access the Azure Portal |
| Write access | None |
| Privilege escalation | Not possible from this tier via Azure RBAC |
| Lateral movement | Limited to Microsoft 365 and business applications only |

### What Entra ID Logs

- Sign-in events to Microsoft 365 and connected applications
- Any attempt to access Azure Portal (will fail, no role assigned)

### Detection Window

**Slow without dedicated monitoring** - a compromised standard 
user account has no Azure access, so cloud resource monitoring 
will not surface the compromise. Detection relies on Microsoft 
365 audit logs, email activity anomalies, and Entra ID sign-in 
risk signals.

### Severity Rating

**Low (Azure context)** - no Azure resource access means no 
direct cloud infrastructure risk. Primary risk is data exposure 
within Microsoft 365 applications such as SharePoint or 
OneDrive.

### Recommended Response

1. Revoke sessions and reset credentials
2. Review Microsoft 365 audit log for data access or exfiltration
3. Check for forwarding rules added to mailbox
4. Notify user and assess phishing or social engineering vector

### Controls Mitigating This Risk

- No Azure RBAC role assigned. Zero Azure Portal access
- Group membership limits application access scope
- Entra ID sign-in risk signals available even on free tier

---

## Summary Table

| Tier | Severity if Compromised | Write Access | Escalation Possible | Detection Speed |
|---|---|---|---|---|
| Tier 0 - Break-Glass | Critical | Full tenant and infrastructure | N/A already highest | Immediate |
| Tier 1 - SOC Analyst | Medium | None | No | Moderate |
| Tier 2 - IT Ops | High | Resources only | No (via RBAC) | Fast to Moderate |
| Tier 3 - Standard User | Low (Azure) | None | No | Slow |

---

## Key Design Validation

This analysis confirms that the tiered access model successfully 
contains blast radius at each level:

- A compromised Tier 3 account cannot reach Azure at all
- A compromised Tier 1 account can see everything but change nothing
- A compromised Tier 2 account can damage resources but cannot 
  touch the identity plane or escalate privileges
- Tier 0 remains isolated from daily operations, minimising 
  exposure surface

The most significant residual risk is a compromised Tier 2 account 
operating slowly within normal-looking activity patterns. This is 
addressed in the detection engineering layer covered in the linked 
Project 2 repository.

---
