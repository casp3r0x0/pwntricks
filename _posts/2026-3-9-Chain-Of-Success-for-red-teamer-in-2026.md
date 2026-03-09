## About

In this blog, I will introduce a philosophy for red teamers that I call it **Chain of Success Framework** - a structured methodology designed to measure how effective and mature a red team truly is.

I developed this framework independently. The motivation was straightforward: there is no standardized measurement to evaluate how capable a red team is when it comes to conducting real-world offensive operations.

Every phase included in this framework focuses exclusively on the team's ability to achieve impact and accurately simulate real APTs and nation-state adversaries. The higher your team scores, the more operationally capable you are.

> This blog post is based on my experience and personal opinion.
---

### Figure 1

![](https://www.pwntricks.com/assets/images/7/1.jpg)

---

## 1. Initial Access

I have divided the initial access phase into the following categories:

- **Zero-Day / N-Day Exploitation**
- **Phishing**
    - Credential harvesting using private or heavily modified tools — must be fully undetectable.
    - Remote code execution via proprietary techniques or well-known methods that are highly customized to evade detection.
- **OSINT** — focused on leaked data and credentials for the target organization.

---

### 1.1 Zero-Day / N-Day Exploitation — *Max Points: 60*

**Philosophy**

This capability is extremely difficult for small-to-medium tier red teams. It requires a well-funded team with dedicated time and resources for vulnerability research or N-day exploit development. An N-day vulnerability refers to a flaw that has been discovered and patched but for which no public exploit exists. These are among the most impactful exploits because organizations often delay patching. Some elite research teams can rediscover a vulnerability and develop a working exploit within a single day.

At this stage, the red team should already have ready-to-deploy exploits in their arsenal.

**Scoring**

| Criteria | Points |
|---|:---:|
| Exploit targets a well-known product | 60 |
| Exploit targets a less common application used by the target | 50 |
| Exploit generates some alerts | 40 |
| No exploit capability | 0 |

**Detections**

When developing exploits, OPSEC must be a primary consideration. No IOCs should be generated that could alert the SOC team, unless it is absolutely necessary for the exploit to function.

**Tools / Resources**

A dedicated research team focused exclusively on vulnerability research and exploit development.

---

### 1.2 Phishing

#### 1.2.1 Credential Harvesting — *Max Points: 10*

**Philosophy**

Credential harvesting should utilize in-house developed or heavily modified tools. The tooling must be fully undetectable.

**Scoring**

| Criteria | Points |
|---|:---:|
| Tool is custom-built and undetected | 10 |
| Tool has known detections | 0 |

**Detections**

CTI platforms, threat intelligence tools, firewalls, and email gateway security solutions (e.g., Proofpoint).

**Tools / Resources**

Build a custom tool capable of bypassing MFA and harvesting credentials from any target website — not limited to Microsoft or other well-known platforms.

---

#### 1.2.2 Remote Code Execution — *Max Points: 30*

**Philosophy**

The red team must have the ability to gain code execution while bypassing EDRs, CTI, firewalls, IDS, IPS, NDS, and SOC monitoring. Deduct 10 points if using a well-known or commercial C2 framework is used, even if the loader itself is undetected.
for example shortcut link to RCE but it is undetectable delivered as phishing.

**Scoring**

| Criteria | Points |
|---|:---:|
| Code execution method is novel | 30 |
| Some EDRs can block or detect the method with special configruation | 20 |
| Uses a well-known TTP but no current detections exist | 10 |
| Detected by at least 3 well-known EDRs | 0 |

**Detections**

EDRs, CTI, firewalls, IDS, IPS, NDS, SOC monitoring.

**Tools / Resources**

An in-house C2 framework with the capability to bypass any EDR on the market. I do not recommend purchasing commercial C2 frameworks, as most of them are heavily signatured. The time and effort required to make a commercial beacon undetectable including Cobalt Strike .

---

### 1.3 OSINT — Leaked Credentials — *Max Points: 10*

**Philosophy**

The team must have the ability to search for leaked credentials on the dark web. In most cases, if the target organization has a mature security posture, there will be minimal leaked credentials. However, overlooked third-party credential leaks are always worth investigating.

A common scenario: a user signs into their personal Gmail account on both their corporate laptop and personal device, saving credentials via Google Password Manager. If the personal device is compromised, all synced credentials — including corporate ones — are exposed.

**Scoring**

| Criteria | Points |
|---|:---:|
| Access to platforms such as Flare, Resecurity, or Recorded Future | 10 |
| No access to any CTI platform | 0 |

**Detections**

If the target has a robust CTI program, they may identify and remediate leaked credentials before the red team can leverage them.

**Tools / Resources**

Flare, Resecurity, Recorded Future (for use during red team operations).

---

## 2. Exfiltration & C2 Communication Channel — *Max Points: 25*

**Philosophy**

During a red team engagement, once you land on a machine with an EDR and successfully bypass it, the C2 will begin beaconing traffic. You must remain undetectable. In mature environments, consider setting the sleep interval to once per day — the C2 checks for new commands only once within a 24-hour period.

**Scoring**

| Criteria | Points |
|---|:---:|
| C2 communication uses well-known, legitimate network traffic patterns | 10 |
| Slow command execution (high sleep intervals) | 5 |
| Utilizing target-owned domains for C2 infrastructure | 10 |

**Detections**

IPS, IDS, SOC monitoring, CTI.

**Tools / Resources**

Custom-developed external C2 channels for your in-house C2 framework or as extensions for commercial C2 platforms.

---

## 3. Post-Exploitation — *Max Points: 50*

### Credential Stealers

**Philosophy**

All of the following capabilities must be tested and confirmed to be undetected by well-known IOC signatures and EDR solutions.

**Scoring**

| Capability | Points |
|---|:---:|
| Chromium-based browser credential stealers (Edge, Chrome) | 10 |
| Password vault stealers (1Password, Bitwarden, etc.) | 10 |
| Cookie stealers | 10 |
| Keyloggers | 10 |
| Hidden VNC | 10 |

**Detections**

SOC monitoring, EDRs.

**Tools / Resources**

Custom-implemented modules within your C2 framework (in-house or commercial) with proprietary techniques.

---

## 4. Lateral Movement — *Max Points: 40*

**Philosophy**

The red team must be able to move within the target network without triggering any detections. This requires custom implementations for lateral movement techniques to minimize IOCs and avoid alerting defensive teams.

**Scoring**

| Criteria | Points |
|---|:---:|
| Lateral movement technique is novel | 40 |
| Lateral movement technique is undetected | 30 |
| Using built-in or publicly known lateral movement methods | 0 |

**Detections**

EDRs, SOC monitoring.

**Tools / Resources**

Develop a custom lateral movement technique that is undetected by modern security solutions.

---

## 5. LSASS / SAM Dump — *Max Points: 60*

**Philosophy**

In certain scenarios, the engagement reaches a dead end — no exploitable vulnerabilities and no misconfigurations available for privilege escalation. In these situations, dumping LSASS or SAM becomes essential to enable lateral movement or achieve privilege escalation.

**Scoring**

| Criteria | Points |
|---|:---:|
| Using a novel TTP to dump LSASS and SAM | 60 |
| Using BYOVD (Bring Your Own Vulnerable Driver) — blind the EDR without killing it | 60 |
| Using BYOVD — killing or suspending the EDR process | 40 |

**Detections**

EDRs, SOC monitoring.

**Tools / Resources**

Custom-developed tools leveraging novel TTPs or BYOVD techniques. The preferred approach is to blind the EDR's ability to observe malicious activity rather than terminating the EDR process entirely.

---

## 6. Persistence — *Max Points: 30*

**Philosophy**

You need to maintain access within the network for as long as possible without any detection.

**Scoring**

| Criteria | Points |
|---|:---:|
| Novel persistence technique | 30 |
| Undetectable persistence technique | 20 |
| Standard persistence that could be detected by some EDRs | 0 |

**Detections**

EDRs, SOC monitoring.

**Tools / Resources**

Custom BOF (Beacon Object File) or plugin to execute persistence mechanisms.

---

## Chain of Success — Red Team Maturity Scoring Table

| Phase | Sub-Phase | Max Points | Elite (275–315) | Advanced (180–274) | Intermediate (80–179) | Beginner (0–79) |
|---|---|:---:|:---:|:---:|:---:|:---:|
| **Initial Access** | Zero-Day / N-Day (well-known product, no alerts) | 60 | 60 | 50 (less common app) | 40 (generates alerts) | 0 (no exploit) |
| **Initial Access** | Phishing — Credential Harvesting (custom undetected tool) | 10 | 10 | 10 | 0 (tool has detections) | 0 |
| **Initial Access** | Phishing — RCE (novel method, custom C2) | 30 | 30 | 20 (some EDRs detect) | 10 (known TTP, no detection) | 0 (detected by 3+ EDRs) |
| **Initial Access** | OSINT — Leaked Credentials (CTI access) | 10 | 10 | 10 | 10 | 0 (no CTI access) |
| **Exfil & C2 Comms** | Legitimate traffic + slow execution + target domains | 25 | 25 (all three) | 15 (traffic + slow exec) | 10 (traffic only) | 0 |
| **Post-Exploitation** | Chromium browser credential stealers | 10 | 10 | 10 | 10 | 0 |
| **Post-Exploitation** | Password vault stealers | 10 | 10 | 10 | 0 | 0 |
| **Post-Exploitation** | Cookie stealers | 10 | 10 | 10 | 10 | 0 |
| **Post-Exploitation** | Keyloggers | 10 | 10 | 10 | 0 | 0 |
| **Post-Exploitation** | Hidden VNC | 10 | 10 | 0 | 0 | 0 |
| **Lateral Movement** | Custom lateral movement technique | 40 | 40 (novel) | 30 (undetected) | 0 (built-in tools) | 0 |
| **LSASS / SAM Dump** | Novel TTP or BYOVD (blind EDR) | 60 | 60 (novel TTP) | 40 (BYOVD kill EDR) | 0 | 0 |
| **Persistence** | Custom persistence mechanism | 30 | 30 (novel) | 20 (undetectable) | 0 (detected by EDRs) | 0 |
| | **TOTAL** | **315** | **~315** | **~235** | **~90** | **~0** |

---

## Tier Summary

| Tier | Points Range | Description |
|---|:---:|---|
| **Elite** | 275 – 315 | Full zero-day capability, custom C2, novel TTPs across all phases. Nation-state or top-tier private red team level. |
| **Advanced** | 180 – 274 | Strong custom tooling, some novel techniques, BYOVD capability. Well-funded professional red team. |
| **Intermediate** | 80 – 179 | Uses known TTPs with modifications, limited zero-day or novel capability. Growing team with some custom tools. |
| **Beginner** | 0 – 79 | Relies on public tools and built-in OS features. Detected by most EDRs. No exploit development or custom C2. |