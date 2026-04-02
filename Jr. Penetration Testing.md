
## Penetration Testing Methodologies

Penetration tests can have a wide variety of objectives and target within scope. **The General theme** which are 5 :

| Stage                | Description                                                                                                                                                                                                      |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Info Gathering       | We collect info about the target from publically available resource, we use OSINT, and research, But keep in mind we don't yet ==scan== .                                                                        |
| Enumeration/Scanning | Discovering applications and services running on the systems, it could be finding a web server that might be potentially vulnerable                                                                              |
| Exploitation         | Once we find vulnerabilities on systems we try to leverage that to get inside.                                                                                                                                   |
| Privilege Escalation | Once exploited which is called a ==foothold==. We could go horizontally (to user's we the same permission) or go vertically (To administrator).                                                                  |
| Post Exploitation    | We could pivot to other hosts within the same network, we could collect information from the host that we are in, we have to cover our track, or report. **This part is mostly influenced by the ROE document**. |

The Table above is more of a general theme type of thing, but we got methodologies for every system lets begin :

#### OSSTMM
This framework is mainly used for systems, software, applications, communications and the human aspect of cybersecurity.

The methodology focuses primarily on how these systems, applications communicate, so it includes a methodology for:

1. Telecommunications (phones, VoIP, etc.)
2. Wired Networks
3. Wireless communications

#### Owasp
It's updated frequently and it's community-driven and it's solely used to test web application and services


#### NIST
is a popular framework used to improve an organisations cybersecurity standards and manage the risk of cyber threats.