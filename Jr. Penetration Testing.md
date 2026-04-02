
## Penetration Testing Methodologies

Penetration tests can have a wide variety of objectives and target within scope. **The General theme** which are 5 :

| Stage                | Description                                                                                                                                                                                                      |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Info Gathering       | We collect info about the target from publically available resource, we use OSINT, and research, But keep in mind we don't yet ==scan== .                                                                        |
| Enumeration/Scanning | Discovering applications and services running on the systems, it could be finding a web server that might be potentially vulnerable                                                                              |
| Exploitation         | Once we find vulnerabilities on systems we try to leverage that to get inside.                                                                                                                                   |
| Privilege Escalation | Once exploited which is called a ==foothold==. We could go horizontally (to user's we the same permission) or go vertically (To administrator).                                                                  |
| Post Exploitation    | We could pivot to other hosts within the same network, we could collect information from the host that we are in, we have to cover our track, or report. **This part is mostly influenced by the ROE document**. |

The Table above is more of 