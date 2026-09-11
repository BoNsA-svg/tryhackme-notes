# 🛡️ TryHackMe Cybersecurity Notes

> My personal cybersecurity knowledge base for penetration testing, red teaming, AI/LLM security, networking, Active Directory, privilege escalation, Python security scripting, cloud, mobile, wireless, and hands-on TryHackMe learning.

## 🏁 Competition Mode

Need something fast during a CTF or competition? Start here:

### 👉 [CTF Start Here](cheatsheets/00-ctf-start-here.md)

### 📝 [CTF Writeups](ctf/README.md)

Quick references:

- [🔌 Service Enumeration](cheatsheets/services.md)
- [🔎 Recon](cheatsheets/recon.md)
- [🌐 Web](cheatsheets/web.md)
- [🪟 Active Directory](cheatsheets/active-directory.md)
- [🐚 Shells & Listeners](cheatsheets/shells.md)
- [📦 File Transfer](cheatsheets/file-transfer.md)
- [🐧 Linux PrivEsc](cheatsheets/linux-privesc.md)
- [🪟 Windows PrivEsc](cheatsheets/windows-privesc.md)
- [🔐 Passwords](cheatsheets/passwords.md)
- [🐍 Python Fundamentals](python-for-pentesting/reference/python-fundamentals-cheatsheet.md)

> **Competition layer:** command-first references for fast lookup. The detailed folders below remain the study/knowledge layer, while `ctf/` stores challenge-specific writeups.

---

## 📚 About This Repository

This repository contains notes collected while studying and practicing cybersecurity. The goal is to turn lessons, commands, methodologies, and security concepts into a practical reference that I can continue improving over time.

> **Note:** These materials are for education, CTFs, labs, and authorized security testing only.

---

## 🧭 Detailed Notes Index

### 🏁 CTF Writeups
- [CTF Writeups & Challenge Notes](ctf/README.md)

### 🐍 Python for Penetration Testing
- [Python for Pentesting Hub](python-for-pentesting/README.md)
- [Python Core Concepts](python-for-pentesting/fundamentals/python-core-concepts.md)
- [Python Fundamentals Quick Reference](python-for-pentesting/reference/python-fundamentals-cheatsheet.md)

### 🤖 AI & LLM Security
- [AI-Augmented Web Applications](ai-security/ai-augmented-web-applications.md)
- [LLM Pentesting](ai-security/llm-pentesting.md)

### 🔎 Reconnaissance
- [Active Reconnaissance](recon/active-recon.md)
- [Passive Reconnaissance](recon/passive-recon.md)

### 🪟 Active Directory
- [Active Directory](active-directory/active-directory.md)

### 🌐 Web Security
- [Web Walking](web-security/web-walking.md)
- [Vulnerability Knowledge](web-security/vulnerability-knowledge.md)

### 🌐 Network & Wireless Security
- [Network Security](network-security/network-security.md)
- [Wireless Security](wireless-security/wireless-security.md)

### 🐧 Privilege Escalation
- [Linux Privilege Escalation](privilege-escalation/linux-privilege-escalation.md)
- [Windows Privilege Escalation](privilege-escalation/windows-privilege-escalation.md)

### ⚔️ Exploitation & Offensive Security
- [Metasploit](exploitation/metasploit.md)
- [Payload Generation](exploitation/payload-generation.md)
- [Shells & Listeners](exploitation/shells-and-listeners.md)
- [Password Attacks](exploitation/password-attacks.md)

### ☁️ Cloud Security
- [Cloud Security](cloud-security/cloud-security.md)

### 📱 Mobile Security
- [Mobile Application Security](mobile-security/mobile-app-security.md)

### 🎣 Social Engineering
- [Phishing](social-engineering/phishing.md)

### 🎓 Learning Paths
- [Junior Penetration Testing](learning-paths/junior-penetration-testing.md)

---

## 🗂️ Repository Structure

```text
tryhackme-notes/
├── README.md
├── cheatsheets/                 # Fast competition references
├── ctf/                         # CTF writeups and challenge notes
│   └── README.md
├── python-for-pentesting/       # Python study + reference + scripts
│   ├── fundamentals/
│   ├── networking/
│   ├── security-scripting/
│   ├── examples/
│   └── reference/
├── ai-security/
├── active-directory/
├── recon/
├── web-security/
├── network-security/
├── wireless-security/
├── privilege-escalation/
├── exploitation/
├── cloud-security/
├── mobile-security/
├── social-engineering/
└── learning-paths/
```

---

## 🧠 How to Use This Repo

```text
Learning / studying
      ↓
Detailed topic notes

CTF / competition
      ↓
cheatsheets/00-ctf-start-here.md
      ↓
Open the relevant quick reference
      ↓
Follow links to detailed notes only when needed

Solved challenge / lessons learned
      ↓
ctf/
      ↓
Sanitized writeup + safe scripts/artifacts
```

For Python, detailed explanations live under `python-for-pentesting/fundamentals/`, while fast syntax reminders live under `python-for-pentesting/reference/`. Complete scripts will be kept separately under `examples/`.

---

## ✍️ Naming Standard

Notes use lowercase kebab-case filenames for consistency:

```text
active-recon.md
linux-privilege-escalation.md
python-core-concepts.md
```

---

## 🚧 Repository Status

This is an actively maintained learning repository. Notes may be reorganized, expanded, corrected, or combined as my knowledge grows.

### Improvements

- [x] Organize notes into topic-based folders
- [x] Standardize filenames
- [x] Create reusable competition cheat sheets
- [x] Add a CTF start-here workflow
- [x] Add a CTF writeups section
- [x] Add service-specific competition reference
- [x] Add Python for penetration-testing study/reference structure
- [ ] Expand Python security scripting as new concepts are learned
- [ ] Expand service references as new material is learned
- [ ] Improve cross-linking between related detailed notes

---

## ⚠️ Responsible Use

The techniques documented here should only be used in environments you own or have explicit authorization to test, including CTF platforms and intentionally vulnerable labs.

---

### ⭐ Keep Learning. Keep Testing. Keep Building.
