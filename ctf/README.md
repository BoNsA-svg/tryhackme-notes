# 🏁 CTF Writeups

This section contains sanitized notes and writeups from Capture The Flag challenges, intentionally vulnerable labs, and cybersecurity competitions.

> **Responsible use:** Only challenges and systems explicitly provided for CTF, lab, or authorized security-testing purposes belong here.

## Platforms

### Hack The Box

Writeups will be organized under:

```text
ctf/hack-the-box/<challenge-name>/
```

### TryHackMe

Writeups will be organized under:

```text
ctf/tryhackme/<room-name>/
```

### Competitions

Competition challenges will be organized by event and challenge:

```text
ctf/competitions/<event-name>/<challenge-name>/
```

## Challenge Categories

Challenges may include:

- Network / Machines
- Web
- Active Directory
- Pwn / Binary Exploitation
- Reverse Engineering
- Cryptography
- Forensics
- OSINT
- Miscellaneous

## Writeup Format

A typical challenge folder can contain:

```text
challenge-name/
├── README.md       # Sanitized writeup
├── scripts/        # Scripts created for the challenge
├── screenshots/    # Useful screenshots
└── artifacts/      # Safe challenge artifacts when redistribution is allowed
```

Each writeup should focus on:

1. Challenge overview
2. Initial observations
3. Enumeration / analysis
4. Key hypotheses
5. Exploitation or solution path
6. Lessons learned
7. References or related notes

## Publishing Rules

Before publishing a CTF workspace:

- Remove flags unless publication is allowed and desired.
- Remove credentials, tokens, VPN material, and session data.
- Do not publish private competition artifacts or content prohibited by event rules.
- Do not publish raw loot from unrelated systems.
- Prefer a clean educational writeup over dumping raw terminal output.

## Relationship to the Rest of the Repository

Use [`../cheatsheets/`](../cheatsheets/) for fast competition commands and references.

Use the topic folders in the repository for deeper technical notes.

Use this `ctf/` section for challenge-specific solutions, lessons, and sanitized writeups.
