# 🔐 Passwords Cheat Sheet

## Identify Hash Type

```bash
hashcat --identify '<HASH>'
```

Common Hashcat modes:

| Type | Mode |
|---|---:|
| MD5 | 0 |
| NTLM | 1000 |
| SHA-1 | 100 |
| SHA-256 | 1400 |
| bcrypt | 3200 |
| Net-NTLMv2 | 5600 |

## Fast Wordlist Pass

```bash
hashcat -m <MODE> -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

## Rules

```bash
hashcat -m <MODE> -a 0 hash.txt /usr/share/wordlists/rockyou.txt \
-r /usr/share/hashcat/rules/best64.rule
```

## Mask Attack

```text
?l lowercase
?u uppercase
?d digit
?s special
```

Example:

```bash
hashcat -m <MODE> -a 3 hash.txt '?u?l?l?l?l?l?d?d?d?d?s'
```

## Show Recovered Passwords

```bash
hashcat -m <MODE> hash.txt --show
john --show hash.txt
```

## Targeted Wordlists

```bash
cewl -d 2 -m 3 --lowercase -w cewl_words.txt http://<TARGET>/
```

Normalize:

```bash
tr '[:upper:]' '[:lower:]' < words.txt | sort -u > words-clean.txt
```

## Credential Reuse Checklist

When you find credentials, test them against **services actually exposed by the target**:

```text
SSH
SMB
WinRM
RDP
Web login
Database
FTP
```

Record every combination so you do not repeatedly test the same thing.

## Safety / Competition Rule

Before password spraying against AD or any lockout-capable service, understand the competition rules and account lockout policy.

Detailed notes: [Password Attacks](../exploitation/password-attacks.md)
