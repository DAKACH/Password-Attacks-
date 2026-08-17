# Password Attacks: Introduction to Authentication & Passwords


## 1. Core Concepts

### The CIA Triad & Identity Governance
* **Confidentiality, Integrity, and Availability (CIA):** The foundational pillar of information security. Most security breaches originate from a failure in maintaining one of these three states.
* **Authentication vs. Authorization:**
  * **Authentication (AuthN):** The verification of an identity claim.
  * **Authorization (AuthZ):** The permissions and access rights granted to an identity *after* successful authentication.

### Authentication Factors
Authentication mechanisms validate identity using one or a combination of four distinct factors:

| Factor | Description | Examples |
| :--- | :--- | :--- |
| **Something you know** | Knowledge-based secrets | Passwords, Passphrases, PINs |
| **Something you have** | Possession-based tokens | Smart cards (CAC), Hardware tokens, Authenticator apps |
| **Something you are** | Inherent biometric traits | Fingerprints, Facial recognition, Retina/Iris scans |
| **Somewhere you are** | Physical/Logical location | Geolocation data, Trusted IP address ranges |

---

## 2. Password Security & Statistical Landscape

### Password Entropy & Key Space
* **Combinatorial Complexity:** A standard 8-character password using only uppercase alphanumeric characters ($A-Z, 0-9$) yields:
  $$36^8 = 208,827,064,576 \text{ possible combinations}$$
* **Passphrases:** High-entropy alternatives utilizing chained dictionary words (e.g., `TreeDogEvilElephant`), offering high resistance to brute-force attacks while remaining memorable.
* **Usability vs. Security Trade-off:** Increasing authentication complexity enhances security posture but introduces operational friction and degraded User Experience (UX).

### Real-World Statistics & Behavioral Trends

| Metric / Trend | Google & Harris Poll (2019) | Panda Security (2025) | Security Implication |
| :--- | :--- | :--- | :--- |
| **Weak Passwords** (`123456`, `password`) | 24% adoption rate | Top breached password (4.5M+ leaks) | High susceptibility to default credential attacks & wordlist matching. |
| **Password Re-use** | 66% across multiple platforms | 23% across $\ge 3$ accounts | Enables credential stuffing attacks across unrelated services. |
| **Password Managers** | 15% adoption | 36% adoption | Upward trend in password randomness and management hygiene. |
| **Post-Breach Behavior** | 45% changed password | 55% retained compromised passwords | Extended window of opportunity for leaked credential reuse. |

---

## 3. Threat Landscape & Verification Tools

* **Credential Stuffing:** Leveraging compromised username/password pairs against secondary platforms due to high password reuse rates.
* **Breach Verification:**
  * **Resource:** [`HaveIBeenPwned`](https://haveibeenpwned.com/) (HIBP)
  * **Mechanism:** Queries centralized databases of publicly disclosed data breaches to identify compromised email addresses and credentials.

---

