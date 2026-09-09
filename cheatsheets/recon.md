# 🔎 Recon Cheat Sheet

## Fast Workflow

```bash
export IP=<TARGET_IP>
nmap -Pn -p- --min-rate 3000 -T4 $IP -oA nmap/all-ports
nmap -Pn -sC -sV -p<PORTS> $IP -oA nmap/services
```

UDP when relevant:

```bash
sudo nmap -sU --top-ports 50 $IP -oA nmap/udp-top50
```

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

Detailed notes: [Active Recon](../recon/active-recon.md) · [Passive Recon](../recon/passive-recon.md)
