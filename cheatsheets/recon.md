# 🔎 Recon Cheat Sheet

## Fast Workflow

```bash
export IP=<TARGET_IP>
mkdir -p nmap

# Discover first; use -Pn only when discovery is blocked or inappropriate
sudo nmap -sn --reason $IP -oA nmap/discovery

# Full TCP exposure, then focused service enumeration
sudo nmap -sS -p- $IP -oA nmap/all-ports
sudo nmap -sC -sV -p<PORTS> $IP -oA nmap/services
```

Fast flags such as `--min-rate` and `-T4/-T5` can miss slow responses or trigger controls. Add them only when the environment and Rules of Engagement support the trade-off.

UDP when relevant:

```bash
sudo nmap -sU --top-ports 100 $IP -oA nmap/udp-top100
sudo nmap -sU -sV -p<UDP_PORTS> $IP -oA nmap/udp-services
```

## Understand the Result

```bash
sudo nmap -sS -p<PORTS> --reason $IP
sudo nmap -p<PORT> --packet-trace -Pn -n $IP
```

```text
SYN-ACK       → open
RST           → closed
No response   → possibly filtered or rate-limited
open|filtered → common UDP ambiguity; validate with version/protocol probes
```

## Save and Compare

```bash
sudo nmap -p- $IP -oA nmap/target
xsltproc nmap/target.xml -o nmap/target.html
```

`-oA` preserves normal, grepable, and XML output. Record exact commands and compare fast scans against a slower baseline.

## Domain, Cloud, and Staff OSINT

```bash
whois <DOMAIN>
dig <DOMAIN> NS
dig <DOMAIN> MX
dig <DOMAIN> TXT
```

Passive checks:

```text
Certificate Transparency names
Historical DNS and cached pages
Shodan/Censys-style third-party results
Cloud-provider URLs in HTML, JavaScript, CSS, and documents
Public repositories and commit history
Job postings and technical employee profiles
```

Treat job-posting technologies as leads, not proof. Confirm cloud ownership and written scope before listing or downloading storage contents. Escalate exposed secrets immediately and collect only minimal evidence.

## Web Ports

```bash
curl -i http://$IP/
whatweb http://$IP/
```

Check:

```text
robots.txt
sitemap.xml
page source
JavaScript
cookies
headers
server/framework banners
```

## DNS

```bash
dig @<DNS_IP> <DOMAIN> ANY
dig axfr @<DNS_IP> <DOMAIN>
nslookup -type=SRV _ldap._tcp.dc._msdcs.<DOMAIN> <DNS_IP>
```

## SMB

```bash
smbclient -L //$IP -N
smbmap -H $IP
nxc smb $IP
```

## NFS

```bash
showmount -e $IP
sudo mount -t nfs $IP:/<EXPORT> /mnt/nfs -o nolock
```

## HTTP Content Discovery

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://$IP/FUZZ
```

## Virtual Hosts

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
-H 'Host: FUZZ.<DOMAIN>' -u http://$IP/ -fs <BASELINE_SIZE>
```

## Things to Record

```text
Hostnames/domains
Ports/services/versions
Web technologies
Users
Shares
Interesting files
Credentials/tokens
Potential vulnerabilities
Potential attack paths
```

## If Stuck

- Re-scan all ports.
- Revisit alternate web ports.
- Check vhosts and DNS names.
- Search source/JS for hidden endpoints.
- Inspect service versions only after confirming them.
- Look for credentials before chasing exploits.

Detailed notes: [Active Recon](../knowledge-base/reconnaissance/active-recon.md) · [Nmap Network Enumeration](../knowledge-base/reconnaissance/nmap-network-enumeration.md) · [Domain, Cloud, and Staff OSINT](../knowledge-base/reconnaissance/domain-cloud-staff-osint.md) · [Service Footprinting](../knowledge-base/reconnaissance/service-footprinting.md) · [Passive Recon](../knowledge-base/reconnaissance/passive-recon.md)
