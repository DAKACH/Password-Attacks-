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

# Introduction to Password Cracking

## Overview

Operating systems and web applications store credentials using **cryptographic hash functions** rather than plain text. A hash function is a one-way mathematical algorithm that converts an arbitrary stream of input bytes into a fixed-length output string (digest).

Because cryptographic hashing is irreversible by design, retrieving the original plaintext password requires **Password Cracking**—systematically computing candidate inputs and matching their hash digests against the target hash.

```
                      +─────────────────────────────────────────+
                      |           Input: Plaintext Pass         |
                      +────────────────────┬────────────────────+
                                           │ One-Way Function
                                           ▼
                      +─────────────────────────────────────────+
                      |         Output: Fixed Hash Digest       |
                      +─────────────────────────────────────────+
```


```bash
# MD5 Hash Example (128-bit digest)
bmdyy@htb:~$ echo -n Soccer06! | md5sum
40291c1d19ee11a7df8495c4cccefdfa  -

# SHA-256 Hash Example (256-bit digest)
bmdyy@htb:~$ echo -n Soccer06! | sha256sum
a025dc6fabb09c2b8bfe23b5944635f9b68433ebd9a1a09453dd4fee00766d93  -
```

## 1. Password Cracking Attack Methodologies

```
                          ┌──────────────────────────────────────────────┐
                          |          Password Cracking Vectors           |
                          └──────────────────────┬───────────────────────┘
                                                 │
         ┌───────────────────────────────────────┼───────────────────────────────────────┐
         ▼                                       ▼                                       ▼
+-----------------------+              +-----------------------+               +-----------------------+
|    Rainbow Tables     |              |     Brute-Force       |               |   Dictionary / List   |
| (Precomputed lookup)  |              | (Exhaustive keyspace) |               | (Statistically likely)|
+-----------------------+              +-----------------------+               +-----------------------+
```

### Method Comparison Matrix

|**Attack Method**|**Mechanism**|**Speed / Efficiency**|**Feasibility & Limitations**|
|---|---|---|---|
|**Rainbow Tables**|Large precomputed lookup tables mapping hashes back to plaintext.|Extremely fast (instant lookup).|Completely neutralized by **Salting**; massive storage footprint required.|
|**Dictionary Attack**|Iterates through wordlists containing statistically common real-world passwords.|High efficiency; optimal under assessment time constraints.|Limited strictly to passwords present inside the chosen wordlist or rule permutations.|
|**Brute-Force Attack**|Systematically attempts every character combination across a specified keyspace.|100% mathematically effective given infinite time.|Computationally prohibitive for passwords $\ge$ 9 characters or computationally heavy algorithms (e.g., bcrypt, Argon2).|
|**Mask Attack**|Hybrid brute-force focusing only on specific character sets or pattern locations.|Highly targeted; significantly faster than pure brute-force.|Requires prior knowledge or structural assumptions about password policy.|

## 2. Cryptographic Salting

A **salt** is a non-secret, randomly generated sequence of bytes appended or prepended to a password before running the hash algorithm.

$$\text{Final Hash} = \text{Hash}(\text{Salt} \parallel \text{Password})$$

```bash
enamto@htb[/htb]$ echo -n Th1sIsTh3S@lt_Soccer06! | md5sum
90a10ba83c04e7996bc53373170b5474  -
```

### Why Salting Defeats Rainbow Tables

- **No Precomputation Reusability:** Every unique salt generates a completely different digest, forcing an attacker to recalculate the entire rainbow table for each individual salt.
- **Exponential Keyspace Growth:** Adding just a single random byte of salt expands a 15-billion-entry table by a factor of 256 ($15\text{B} \times 256 = 3.84\text{T}$ entries).

## 3. Wordlists & Resource Standards

Penetration testers prioritize **Dictionary Attacks** using vetted real-world breach wordlists.

- **[rockyou.txt](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt):** Over 14 million plaintext credentials leaked from the 2009 RockYou database breach (`/usr/share/wordlists/rockyou.txt`).
- **[SecLists](https://github.com/danielmiessler/SecLists):** The industry-standard collection of security assessment lists, including discovery paths, usernames, common passwords, and pattern dictionaries.

```bash
enamto@htb[/htb]$ head -n 10 /usr/share/wordlists/rockyou.txt

123456
12345
123456789
password
iloveyou
princess
1234567
rockyou
12345678
abc123
```
