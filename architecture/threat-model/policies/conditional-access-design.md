# Conditional Access Policy Design - AxiomCorp

## Overview

Conditional Access is a Microsoft Entra ID feature (requires P1 
licence minimum) that enforces access controls based on identity 
signals, device state, and location. It operates on an 
if-then model: if a defined condition is met, then enforce 
a specified control.

This document specifies the Conditional Access policies that 
would be deployed in a production AxiomCorp environment. 
These policies have been designed but not enforced in this lab 
due to free tier licensing constraints. Where policies require 
P2 features, this is noted explicitly.

Policies are ordered by deployment priority. Policy 1 would 
be the first deployed in any real environment.

---

## Policy 1 - Block Legacy Authentication

**Priority:** Deploy first, highest risk, easiest win
**Licence required:** P1
**Status in lab:** Designed, not enforced

### Specification

| Field | Value |
|---|---|
| Name | BLOCK-LegacyAuthentication-AllUsers |
| Target users | All users |
| Target apps | All cloud apps |
| Condition | Client apps: Exchange ActiveSync clients and other clients (legacy auth protocols including IMAP, POP3, basic SMTP auth) |
| Access control | Block access |

### Threat Rationale

Legacy authentication protocols were designed before MFA existed 
and cannot enforce it. Any MFA policy is entirely bypassed if 
legacy auth is permitted, an attacker can authenticate using 
stolen credentials via a legacy protocol and MFA will never be 
challenged.

This is one of the most commonly exploited identity weaknesses 
in Microsoft environments. Microsoft's own data indicates the 
majority of password spray attacks use legacy authentication.

**MITRE ATT&CK:** T1078 - Valid Accounts. Blocking legacy auth 
closes a permanent MFA bypass that would otherwise undermine 
every other Conditional Access policy deployed.

### Expected Impact

Low disruption risk in a modern environment. Any users or 
systems relying on legacy auth protocols would need to be 
migrated to modern authentication before enforcement. A 
report-only mode should be run for two weeks before enforcement 
to identify any dependencies.

---

## Policy 2 - Require MFA for Privileged Tiers Outside the UK

**Priority:** Deploy second
**Licence required:** P1
**Status in lab:** MFA configured per-user on soc.analyst1 as free-tier equivalent

### Specification

| Field | Value |
|---|---|
| Name | REQUIRE-MFA-Tier1Tier2-NonUK |
| Target users | AxiomCorp-Tier1-SOC and AxiomCorp-Tier2-ITOps groups |
| Target apps | Microsoft Azure Management |
| Condition | Location: Any location excluding named location UK-TrustedRanges |
| Access control | Require multi-factor authentication |

### Threat Rationale

Privileged accounts authenticating from outside the UK represent 
an elevated risk signal. This policy enforces an additional MFA 
step-up challenge for any Tier 1 or Tier 2 sign-in originating 
outside known UK IP ranges, directly challenging potential 
impossible travel scenarios where credentials have been 
compromised and used from a foreign location.

**MITRE ATT&CK:** T1078.004 - Cloud Accounts. Credentials 
obtained via phishing or credential stuffing are frequently 
used from attacker infrastructure located outside the 
victim organisation's geography.

### Named Location Configuration

A named location called UK-TrustedRanges would be configured 
with known UK corporate IP ranges. Sign-ins from within this 
range are treated as lower risk. Sign-ins from outside it 
trigger MFA step-up.

### Expected Impact

Minimal disruption for UK-based users. Users travelling 
internationally would be prompted for MFA, acceptable 
friction for privileged accounts.

---

## Policy 3 - Block High-Risk Sign-ins

**Priority:** Deploy third
**Licence required:** P2 (Entra ID Protection)
**Status in lab:** Not available on free tier, production design only

### Specification

| Field | Value |
|---|---|
| Name | BLOCK-HighRiskSignIn-AllUsers |
| Target users | All users |
| Target apps | All cloud apps |
| Condition | Sign-in risk level: High (Entra ID Protection) |
| Access control | Block access and require password reset |

### Threat Rationale

Entra ID Protection uses Microsoft's global threat intelligence 
to calculate a real-time risk score for every sign-in. A High 
risk score indicates signals such as the credential appearing 
in a known breach database, sign-in from an anonymous proxy or 
Tor exit node, or patterns consistent with an active attack.

Blocking high-risk sign-ins in real time prevents an attacker 
from completing authentication even when they hold valid 
credentials.

**MITRE ATT&CK:** T1078 - Valid Accounts. This policy 
specifically targets the scenario where credentials have been 
compromised but the attacker has not yet completed a 
successful sign-in.

### Licence Limitation Note

This policy requires an Entra ID P2 licence to access 
sign-in risk signals from Entra ID Protection. In this lab 
environment, this policy exists as a design specification only. 
It would be the third policy deployed in a production 
environment after Policies 1 and 2 are stable.

### Expected Impact

Low false positive rate when scoped to High risk only. 
Medium risk sign-ins may be handled separately with a 
require-MFA action rather than a full block.

---

## Deployment Order and Rationale

| Order | Policy | Why This Order |
|---|---|---|
| 1 | Block Legacy Authentication | Closes MFA bypass before any MFA policies are meaningful |
| 2 | Require MFA Outside UK | Adds location-aware challenge for privileged accounts |
| 3 | Block High-Risk Sign-ins | Adds real-time threat intelligence layer on top of existing controls |

Deploying in any other order risks creating gaps. For example, 
deploying Policy 2 before Policy 1 means MFA enforcement can 
still be bypassed via legacy auth protocols, making the 
policy ineffective.

---

## Free Tier Implementation Note

The following free-tier equivalent controls were implemented 
in this lab as substitutes for the full Conditional Access 
policies above:

| Production Policy | Free Tier Equivalent |
|---|---|
| REQUIRE-MFA Conditional Access policy | Per-user MFA enabled on soc.analyst1 |
| BLOCK-HighRiskSignIn | Entra ID free sign-in risk visibility only (no automated enforcement) |
| BLOCK-LegacyAuthentication | Not enforceable without P1, documented as production design intent |

---
