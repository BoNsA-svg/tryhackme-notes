
# Penetesting Fundamentals

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

The framework provides guidelines on security controls & benchmarks for success for organisations from critical infrastructure (power plants, etc.) all through to commercial.  There is a limited section on a standard guideline for the methodology as we penetration testers should take.


#### NCSC CAF

 is an extensive framework of fourteen principles used to assess the risk of various cyber threats and an organisation's defences against these.

  
  The framework applies to organisations considered to perform "vitally important services and activities" such as critical infrastructure, banking, and the likes. The framework mainly focuses on and assesses the following topics:

- Data security
- System security
- Identity and access control
- Resiliency
- Monitoring
- Response and recovery planning


### Boxes' Of Penetration Testing

- *==BLACK BOX==* - No Knowledge of the target we are testing, so we should spend a lot of time taking our time on Info. Gathering (Stage 1) and enumeration (Stage 2). 

- **==Grey Box==** - Partial Knowledge of the system, most popular way if testing. Grey-Box testing, the limited knowledge given saves time, and is often chosen for extremely well-hardened attack surfaces.
- **White Box** - is a low-level process usually done by a software developer who knows programming and application logic.


# Principle Of Security

## CIA Triad

The CIA triad is an information security model that is used in consideration throughout creating a security policy.

**Confidentiality**
Protection of data from unauthorized access and misuse. Basically it means to only give access to the intended person only.

**Integrity**
This element of CIA is to make sure information is kept accurate and consistent unless authorized changes are made. Again Basically it means to keep the data unaltered.

**Availability**
