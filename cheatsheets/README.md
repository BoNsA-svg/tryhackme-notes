# Cybersecurity Super Cheat Sheet

This is the master quick-reference layer for the repository. It combines practical material learned from TryHackMe, Hack The Box, labs, and research, but organizes it by topic rather than platform.

Use the smallest relevant sheet during a lab. Open the linked knowledge note or methodology when you need deeper reasoning.

## Start Here

- [CTF Start Here](00-ctf-start-here.md)
- [Penetration Testing Process](pentest-process.md)
- [Planning and Scoping](planning-and-scoping.md)
- [Reconnaissance](recon.md)
- [Service Enumeration](services.md)
- [Web Enumeration](web-enumeration.md)
- [Complete Web Testing](web.md)

## Foothold and Exploitation

- [Passwords](passwords.md)
- [Shells and Listeners](shells.md)
- [File Transfer](file-transfer.md)
- [Linux Privilege Escalation](linux-privesc.md)
- [Windows Privilege Escalation](windows-privesc.md)
- [Active Directory](active-directory.md)

## Assessment Types

- [External Network](external-network.md)
- [Internal Network](internal-network.md)
- [Wireless](wireless.md)
- [Cloud](cloud.md)
- [Mobile](mobile.md)
- [Pentester Threat Modelling](threat-modelling-for-pentesters.md)

## Reporting and Verification

- [Pentest Reporting](pentest-reporting.md)
- [Retesting](retesting.md)

## AI and LLM Security

- [AI Threat Modelling](ai-threat-modelling.md)
- [AI System Reconnaissance](ai-system-reconnaissance.md)
- [Prompt Injection](prompt-injection.md)
- [Jailbreaking](jailbreaking.md)
- [Prompt Defence](prompt-defence.md)
- [AI Supply Chains](ai-supply-chain.md)
- [AI Supply-Chain Attack Vectors](ai-supply-chain-attack-vectors.md)
- [Securing the AI Supply Chain](securing-ai-supply-chain.md)
- [RAG Security](rag-security.md)

## Programming References

- [Python Fundamentals](../knowledge-base/programming/python-for-pentesting/reference/python-fundamentals-cheatsheet.md)
- [Python for Pentesting](../knowledge-base/programming/python-for-pentesting/reference/python-pentesting-cheatsheet.md)

## Fast Decision Loop

```text
Confirm scope -> Enumerate -> Form hypothesis -> Test safely -> Record evidence
             -> Re-enumerate or pivot -> Validate impact -> Report and clean up
```

## Maintenance Rule

When new source material arrives:

- Add commands only if they are useful and understood.
- Merge duplicates instead of stacking equivalent commands.
- Preserve placeholders such as `TARGET`, `DOMAIN`, and `PORT`.
- Keep safety gates and evidence reminders beside high-impact actions.
- Link to detailed knowledge rather than copying long explanations.

> Use every technique only in systems you own or have explicit authorization to assess.
