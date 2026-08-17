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
---

# John The Ripper (JtR) Foundations & Cracking Modes

## Overview

**John the Ripper (JtR)** is an open-source, highly adaptable password cracking tool originally developed for UNIX-based environments. The community-enhanced **JtR Jumbo** version extends core capabilities with OpenMP multi-threading, 64-bit architecture optimizations, expanded hash support (hundreds of formats), and an extensive suite of `*2john` file conversion utilities.

```
[ Target Data: /etc/shadow, Encrypted PDF, SSH Key, NTLM ]
                        │
                        ▼
           [ Conversion via *2john Utility ]
                        │
                        ▼
       [ Format Identification (hashid / context) ]
                        │
                        ▼
      ┌─────────────────┴─────────────────┐
      ▼                                   ▼
[ Wordlist + Rules Mode ]      [ Incremental / Single Mode ]
      │                                   │
      └─────────────────┬─────────────────┘
                        ▼
        [ Plaintext Credential Recovery ]
```

## 1. Core Cracking Modes

```
                                  ┌──────────────────────────────────────────────┐
                                  │           JtR Cracking Strategies            │
                                  └──────────────────────┬───────────────────────┘
                                                         │
         ┌───────────────────────────────────────────────┼───────────────────────────────────────────────┐
         ▼                                               ▼                                               ▼
+--------------------------------+              +--------------------------------+              +--------------------------------+
|          Single Mode           |              |         Wordlist Mode          |              |        Incremental Mode        |
|  `john --single <file>`        |              |  `john --wordlist=<list> <f>`  |              |  `john --incremental <file>`   |
| Rules based on GECOS & user    |              | Dictionary + optional rules    |              | Markov model based brute-force |
+--------------------------------+              +--------------------------------+              +--------------------------------+
```

### Mode Comparison Matrix

|**Cracking Mode**|**Mechanism & Strategy**|**Primary Target / Use Case**|**Syntax Example**|
|---|---|---|---|
|**Single Mode**|Leverages username, home directory path, and GECOS information (real name, phone) using mutating permutation rules.|Linux `/etc/passwd` or `/etc/shadow` entries where users pick passwords based on personal names.|`john --single passwd`|
|**Wordlist Mode**|Standard dictionary attack testing pre-existing password lists line-by-line; supports dynamic rule mutations.|General password auditing against real-world breach wordlists (`rockyou.txt`).|`john --wordlist=rockyou.txt hashes.txt`|
|**Incremental Mode**|Statistical brute-force using Markov chain models to test character combinations in order of probability.|Exhaustive recovery when wordlist attacks fail and keyspace parameters can be constrained.|`john --incremental hashes.txt`|

## 2. Practical Execution: Single Mode against Unix Passwd

Given a standard Linux shadow/passwd entry containing user and GECOS metadata:

```
r0lf:$6$ues25dIanlctrWxg$nZHVz2z4kCy1760Ee28M1xtHdGoy0C2cYzZ8l2sVa1kIa8K9gAcdBP.GI6ng/qA4oaMrgElZ1Cb9OeXO4Fvy3/:0:0:Rolf Sebastian:/home/r0lf:/bin/bash
```

### Launching Single Mode

```bash
enamto@htb[/htb]$ john --single passwd

Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
1g 0:00:00:00 DONE (2026-08-17) 12.50g/s 5400p/s 5400c/s 5400C/s
```

### Inspecting Cracked Results

```bash
enamto@htb[/htb]$ john --show passwd
```

## 3. Hash Identification & Format Flags

When the hashing algorithm cannot be inferred from the prefix (e.g., `$6$` for SHA-512 crypt, `$1$` for MD5 crypt), use `hashid` with the `-j` flag to output candidate JtR formats:

```bash
enamto@htb[/htb]$ hashid -j 193069ceb0461e1d40d216e32c79c704

Analyzing '193069ceb0461e1d40d216e32c79c704'
[+] MD5 [JtR Format: raw-md5]
[+] NTLM [JtR Format: nt]
[+] RIPEMD-128 [JtR Format: ripemd-128]
```

### Common JtR Format Flags (`--format=<format>`)

|**Target Algorithm / Protocol**|**JtR Format Identifier**|**Example Invocation**|
|---|---|---|
|**Raw MD5**|`raw-md5`|`john --format=raw-md5 hashes.txt`|
|**Raw SHA-256 / SHA-512**|`raw-sha256` / `raw-sha512`|`john --format=raw-sha256 hashes.txt`|
|**Windows NT (NTLM)**|`nt`|`john --format=nt ntlm_hashes.txt`|
|**NetNTLMv2**|`netntlmv2`|`john --format=netntlmv2 captured_v2.txt`|
|**Linux crypt(3) SHA-512**|`sha512crypt`|`john --format=sha512crypt shadow.txt`|
|**Kerberos 5 TGS (Kerberoast)**|`krb5tgs`|`john --format=krb5tgs krb_tgs.txt`|

## 4. Extracting & Cracking Encrypted Files (`*2john`)

JtR includes extraction utilities that convert encrypted file containers, archives, and keys into format-compliant hash strings.

$$\text{[utility]}2\text{john [encrypted\_target]} > \text{target.hash}$$

### Essential `*2john` Conversion Tools

|**Utility**|**Input File Type**|**Extraction Command Example**|
|---|---|---|
|**`ssh2john`**|Encrypted SSH Private Key|`ssh2john id_rsa > id_rsa.hash`|
|**`zip2john`**|Password-Protected ZIP Archive|`zip2john secret.zip > zip.hash`|
|**`rar2john`**|Encrypted RAR Archive|`rar2john backup.rar > rar.hash`|
|**`pdf2john`**|Password-Protected PDF Document|`pdf2john locked.pdf > pdf.hash`|
|**`keepass2john`**|KeePass Database (`.kdbx`)|`keepass2john database.kdbx > keepass.hash`|
|**`putty2john`**|Encrypted PuTTY Key (`.ppk`)|`putty2john key.ppk > putty.hash`|
|**`office2john`**|Encrypted MS Office Document|`office2john report.docx > office.hash`|

### Execution Workflow: SSH Key Cracking Example

```bash
# 1. Extract hash from encrypted RSA key
ssh2john id_rsa > id_rsa.hash

# 2. Run dictionary attack using JtR
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash

# 3. View cracked passphrase
john --show id_rsa.hash
```

---

# Hashcat Foundations, Hash Modes, & Attack Strategies

## Overview

[**Hashcat**](https://hashcat.net/) is the industry-standard, GPU-accelerated password cracking utility designed for high-throughput recovery of cryptographic hashes across Linux, Windows, and macOS. It leverages massive parallelism across OpenCL/CUDA compute cores, making it orders of magnitude faster than CPU-only tools when attacking raw algorithms (e.g., MD5, NTLM, SHA-256).

```
                            ┌──────────────────────────────────────────────┐
                            │           Hashcat Execution Syntax           │
                            └──────────────────────┬───────────────────────┘
                                                   │
                hashcat -a [AttackMode] -m [HashType] [Hashes] [Wordlist / Rules / Mask]
                                │               │
             ┌──────────────────┴────────┐      └──────────────────┬──────────────────┐
             ▼                           ▼                         ▼                  ▼
    -a 0 (Dictionary)            -a 3 (Mask/Brute)          -m 0 (MD5)         -m 1000 (NTLM)
```

## 1. Syntax & Core Parameters

$$\text{hashcat} \quad -a \text{ [Mode]} \quad -m \text{ [Hash ID]} \quad \text{[Target Hash / File]} \quad \text{[Attack Inputs]}$$

### Parameter Definitions

|**Flag**|**Parameter**|**Function**|
|---|---|---|
|`-a`|Attack Mode|Specifies the cracking strategy (`0` = Straight/Dictionary, `3` = Mask/Brute-Force).|
|`-m`|Hash Type|Specifies the numeric identifier of the hashing algorithm (e.g., `0` for MD5, `1000` for NTLM).|
|`-r`|Rule File|Applies dynamic mutation rules to words from a wordlist (e.g., `best64.rule`).|
|`-1, -2, -3, -4`|Custom Charsets|Defines user-specified character sets for mask attacks.|
|`-o`|Output File|Writes cracked `hash:plaintext` pairs directly to a destination file.|
|`--show`|Show Cracked|Displays previously cracked hashes stored in the local `hashcat.potfile`.|

## 2. Common Hash Types & Identification

Hashcat assigns unique numeric IDs to hundreds of hash formats. Identifying the target algorithm can be done via known context, the official Hashcat example table, or the [`hashid`](https://github.com/psypanda/hashID) tool.

```bash
enamto@htb[/htb]$ hashid -m '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'

Analyzing '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'
[+] MD5 Crypt [Hashcat Mode: 500]
[+] Cisco-IOS(MD5) [Hashcat Mode: 500]
```

### Essential Hashcat Mode Reference (`-m`)

| **Mode (-m)** | **Algorithm / Protocol** | **Format Example / Description**                           |
| ------------- | ------------------------ | ---------------------------------------------------------- |
| **`0`**       | MD5                      | Raw 128-bit hexadecimal digest (`e3e3ec5831ad5e72...`)     |
| **`100`**     | SHA1                     | Raw 160-bit hexadecimal digest                             |
| **`1000`**    | NTLM                     | Windows NT password hash (`b4b9b02e6f09a9bd...`)           |
| **`1800`**    | sha512crypt `$6$`        | Standard modern Linux `/etc/shadow` hash                   |
| **`5600`**    | NetNTLMv2                | Captured Windows network authentication challenge/response |
| **`13100`**   | Kerberos 5 TGS-REP       | Kerberoasting service ticket format (`$krb5tgs$23$...`) `  |
| **`?a`**      | All standard ASCII       | `?l?u?d?s`                                                 |
| **`?b`**      | All binary bytes         | `0x00 - 0xff`                                              |

### Practical Execution: [Custom Mask Attack](https://hashcat.net/wiki/doku.php?id=mask_attack)

Cracking a hash where the password format is known to be **[Upper][4 Lower][Digit][Special]** (e.g., `Admin1!`):

```bash
enamto@htb[/htb]$ hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
```

### Custom User-Defined Charsets (`-1` through `-4`)

```bash
# ?1 = Lowercase + Digits; Length = 6 characters
hashcat -a 3 -m 1000 target.hash -1 '?l?d' '?1?1?1?1?1?1'
```

---

