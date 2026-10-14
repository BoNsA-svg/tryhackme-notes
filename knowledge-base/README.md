# Unified Cybersecurity Knowledge Base

This is the topic-first knowledge layer of the repository. It combines durable lessons from TryHackMe, Hack The Box, independent research, labs, and future sources into one reusable body of knowledge.

Platform notes remain under `platforms/` as source records. Material is promoted here when it adds a reusable concept, workflow, technique, or defensive lesson.

## How the Layers Work

```text
THM / HTB / labs / research
            |
            v
platforms/  - source-specific study records
            |
            v
knowledge-base/  - merged concepts organized by topic
            |
            v
pentesting-methodology/  - ordered professional workflows
            |
            v
cheatsheets/  - fast commands, checks, and decisions
```

When two sources teach the same concept, the knowledge base keeps one clear explanation and retains any useful differences. The cheat sheets contain execution guidance, not full lesson prose.

## Penetration Testing

- [Foundations and Testing Process](penetration-testing/foundations-and-process.md)
- [Active Reconnaissance](reconnaissance/active-recon.md)
- [Passive Reconnaissance](reconnaissance/passive-recon.md)
- [Network Security](network-security/network-security.md)
- [Web Enumeration](web-security/web-enumeration.md)
- [Web Walking](web-security/web-walking.md)
- [Web Vulnerability Knowledge](web-security/vulnerability-knowledge.md)
- [Active Directory](active-directory/active-directory.md)
- [Linux Privilege Escalation](privilege-escalation/linux-privilege-escalation.md)
- [Windows Privilege Escalation](privilege-escalation/windows-privilege-escalation.md)
- [Metasploit](exploitation/metasploit.md)
- [Payload Generation](exploitation/payload-generation.md)
- [Shells and Listeners](exploitation/shells-and-listeners.md)
- [Password Attacks](exploitation/password-attacks.md)

## Specialized Security Domains

- [AI and LLM Security](ai-security/README.md)
- [Cloud Security](cloud-security/cloud-security.md)
- [Wireless Security](wireless-security/wireless-security.md)
- [Mobile Application Security](mobile-security/mobile-app-security.md)
- [Phishing and Social Engineering](social-engineering/phishing.md)

## Programming for Security

- [Python for Penetration Testing](programming/python-for-pentesting/README.md)
  - [Core Concepts](programming/python-for-pentesting/fundamentals/python-core-concepts.md)
  - [Building Scripts](programming/python-for-pentesting/fundamentals/python-building-scripts.md)
  - [Pentesting Scripts](programming/python-for-pentesting/security-scripting/python-pentesting-scripts.md)

## Companion Layers

- [Penetration Testing Methodologies](../pentesting-methodology/README.md)
- [Super Cheat Sheet](../cheatsheets/README.md)
- [Platform Source Notes](../platforms/README.md)
- [Reporting System](../reporting/README.md)

## Maintenance Rule

For every new THM or HTB lesson:

1. Preserve the organized source note under its platform path.
2. Merge reusable knowledge into the matching topic file.
3. Update a methodology if the lesson changes the testing workflow.
4. Add only fast, operational material to the relevant cheat sheet.
5. Update indexes when a new topic is introduced.

> All offensive-security material is for authorized assessments, labs, and CTF environments only.
