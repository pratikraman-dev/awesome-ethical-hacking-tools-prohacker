# Ethical Hacking & Rules of Engagement Guidelines

This document outlines the industry-standard ethical guidelines, legal boundaries, and rules of engagement (RoE) that all security researchers, penetration testers, and users of this repository must adhere to.

---

## 1. The Core Principle: Authorization

> [!IMPORTANT]
> **Never conduct any security testing, port scanning, or vulnerability exploitation without explicit, written, and signed authorization from the target system's owner.**

Before scanning or testing any target, you must obtain a **written agreement** specifying:
*   The exact scope of the assessment (IP addresses, domain names, network ranges).
*   The permitted testing techniques (e.g., active scanning, manual exploitation, social engineering).
*   The designated testing window (authorized dates and hours).
*   Points of contact for reporting critical findings or system outages.

---

## 2. Rules of Engagement (RoE) Matrix

When conducting a penetration test or vulnerability assessment, follow these standard operational guidelines:

| Action Category | Permitted (Authorized Scope Only) | Prohibited (Violates Ethics/Law) |
| :--- | :--- | :--- |
| **Port Scanning** | Slow, throttled scans during scheduled windows. | Aggressive, high-speed scans that cause service disruption. |
| **Exploitation** | Proof of Concept (PoC) validation showing vulnerability presence. | Destructive exploits that modify databases or crash services. |
| **Denial of Service (DoS)** | Only when explicitly requested and scheduled by the client. | Any unscheduled resource exhaustion attacks. |
| **Data Extraction** | Accessing the minimum data necessary to prove the vulnerability. | Mass exfiltration of user records, PII, or trade secrets. |
| **Social Engineering** | Phishing simulations targeting pre-approved user cohorts. | Phishing general public or using illegal extortion/blackmail. |

---

## 3. Responsible & Coordinated Vulnerability Disclosure (CVD)

If you discover a vulnerability in a third-party application or public website:

```mermaid
graph TD
    A["Vulnerability Discovered"] --> B["Document Proof of Concept (PoC)"]
    B --> C["Locate Security Contact (security.txt or Bug Bounty page)"]
    C --> D["Submit Encrypted Draft Report to Vendor"]
    D -->|Wait 90 Days (Coordinated Window)| E["Vendor Patches Vulnerability"]
    E --> F["Coordinated Public Disclosure"]
    D -->|Critical Outage / No Contact| G["Notify CERT/CC / Regulatory Body"]
```

1.  **Draft a Detailed Report:** Document the vulnerability description, steps to reproduce, impact assessment, and remediation suggestions.
2.  **Locate the Security Contact:** Check `/.well-known/security.txt` on the target website or search for their official bug bounty/vulnerability disclosure page.
3.  **Coordinate Patch Time:** Give the vendor a reasonable timeframe (standard is **90 days** as per Google Project Zero guidelines) to patch the vulnerability before disclosing it publically.
4.  **Encrypt Communication:** Use PGP keys to encrypt reports containing sensitive proof-of-concepts.

---

## 4. Legal Safe Harbor

A **Safe Harbor** policy protects researchers from legal action (such as prosecution under the Computer Fraud and Abuse Act (CFAA) or DMCA) as long as they act in good faith and follow the rules of the organization's Vulnerability Disclosure Policy (VDP).
*   Always check if the target has a "Vulnerability Disclosure Policy" or "Bug Bounty Program" that guarantees Safe Harbor.
*   If no VDP or Bug Bounty exists, do not conduct any active testing, as it carries severe legal risks.

---

## 5. Summary Legal Disclaimer

> [!WARNING]
> Using tools listed in this repository to access networks or systems without prior written consent constitutes unauthorized access and is punishable under cybercrime laws (e.g., the CFAA in the United States, the Computer Misuse Act in the United Kingdom, and equivalent international legislation). 
>
> The developers of this repository assume no liability for misuse, damage, or legal actions resulting from the execution of these security tools.
