# 12. Pivoting, Tunneling, and Port Forwarding

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 18 / 18 sections
> **Outline metadata:** Medium, Tier 2, 18 sections, 14 interactive exercises, 1 assessment (as displayed), est. 2 days
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 28 question entries deliberately not attempted (§2 ×3, §3 ×2, §4 ×2, §5 ×2, §6 ×1, §7 ×1, §8 ×1 optional, §9 ×1 optional, §10 ×3, §11 ×1, §12 ×1, §13 ×1, §14 ×1, §15 ×1, §16 ×7 — the skills assessment) and no target spawned. "Reveal Answer" buttons in §8–9 were not clicked.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Using one compromised host to reach networks you cannot route to directly: port forwarding and tunneling traffic through the foothold so internal services become testable.

**Key concepts to own:** local vs remote forwarding; dynamic / SOCKS forwarding; tunneling concepts; double pivots.

## The module's frame (read this first)

- **Pivoting** = using a compromised host with more than one network interface (the **pivot host** / foothold / beachhead) as a stepping stone into networks your attack host cannot reach directly. **Tunneling** = pivoting with the traffic wrapped inside another protocol so its nature is hidden. **Lateral movement** is the broader term — pivoting specifically crosses *network boundaries* toward a target.
- **Before choosing a technique (§1–2):** on the new host, check privileges, enumerate every NIC/IP (`ipconfig`/`ifconfig`), the routing table (`ip route`, `netstat -r`), and gateways. A second NIC or a tunnel interface is the signal that another network is reachable. Prefer ports the firewall already allows. **Draw the network map** and keep it updated — the module repeats this in §16 and §17.
- **Proxychains discipline (recurs everywhere):** through proxychains only full TCP-connect scans work (`nmap -Pn -sT`); ping sweeps silently fail against Windows hosts (ICMP blocked) — a "no hosts" result may be a tooling artifact, not an empty network.

## Deep summary — section by section (original words)

**1. Introduction to Pivoting, Tunneling, and Port Forwarding** — Definitions above. First moves on any foothold: privilege level, network connections, reachable VPN/remote-access software, which adapters reach which segments. Failure mode: assuming the attack host can reach an internal network directly when the pivot is the only path. *Use: both.*

**2. The Networking Behind Pivoting** — IPs live on NICs; hosts can have several of each. Static addresses are typical for servers/routers/printers/critical services; the subnet mask splits network vs host parts; off-network traffic goes via the default gateway, and the routing table shows which interface serves which destination. Tunneling through a port a service legitimately uses (e.g. the web port) blends with allowed traffic. *(3 Q — gap.)* *Use: both.*

**3. Dynamic Port Forwarding with SSH and SOCKS Tunneling** — Local forward (`ssh -L local:host:remote`) binds one local port to one service through the pivot. **Dynamic forward** (`ssh -D 9050 user@pivot`) opens a local SOCKS listener that forwards *anything* for the target network — point proxychains at it (`socks4 127.0.0.1 9050`) and your tools behave as if the internal network were local (scans, msfconsole, xfreerdp). Workflow: verify the pivot's services → open the forward → confirm the listener → proxychains → enumerate. *(2 Q — gap.)* *Use: both.*

**4. Remote/Reverse Port Forwarding with SSH** — For when the internal host has **no route back** to you, so a normal reverse shell dies: `ssh -R` makes the *pivot* listen on a port and forwards connections back through the existing SSH session to your handler. The payload must call back to the **pivot's internal address** (not your public one) — the classic mistake. Sequence: build payload (callback = pivot IP) → open the `-R` forward → deliver/run payload → handler catches the session over the SSH socket. *(2 Q — gap.)* *Use: both.*

**5. Meterpreter Tunneling & Port Forwarding** — Pivoting from inside a session: sweep the internal network (ping-sweep post module or scripted loops — retry, first pings can be dropped), then **AutoRoute** (`run autoroute -s <network>`) so session traffic routes into that network, and/or start the **SOCKS proxy** module (`auxiliary/server/socks_proxy`) for external tools via proxychains. `portfwd add` maps single services (forward or reverse with `-R`). Skipping the route step is why "proxychains finds nothing." *(2 Q — gap.)* *Use: both.*

**6. Socat Redirection with a Reverse Shell** — Socat is a plain bidirectional relay: on the pivot, `socat TCP4-LISTEN:<port>,fork TCP4:<attack-host>:<port>` — the target calls the *pivot*, the pivot forwards to your listener, and to the target's network the connection looks local to the pivot. No SSH needed. The listener must face the target side. *(1 Q — gap.)* *Use: both.*

**7. Socat Redirection with a Bind Shell** — Same relay, opposite shell direction: the target runs a **bind** payload (it listens), the pivot's socat forwards your handler's connection to it. Constraint: the bind port must be reachable from the pivot's network — socat can't cross a firewall the target itself can't. Prefer reverse shells where inbound ports are blocked. *(1 Q — gap.)* *Use: both.*

**8. SSH for Windows: plink.exe** — PuTTY's command-line client gives a Windows host the same dynamic forward (`plink -ssh -D 9050 user@host`); **Proxifier** then routes Windows apps (mstsc for RDP) through the SOCKS listener. Using a tool already on the box is a living-off-the-land move — quieter than uploading. Don't assume an SSH client exists on older Windows. *(1 optional exercise — gap.)* *Use: both.*

**9. SSH Pivoting with sshuttle** — sshuttle removes per-tool proxy config: it installs iptables rules that redirect traffic for the target network through the SSH session, so tools run *directly* against internal hosts. Costs: needs **root** (iptables) and works over SSH only. Pick it when you have root + SSH and want simplicity; proxychains when you don't. *(1 optional exercise — gap.)* *Use: both.*

**10. Web Server Pivoting with Rpivot** — A reverse SOCKS proxy (Python 2) for web-server footholds: `server.py` on your host, `client.py` on the pivot connects *back* to you, opening a local SOCKS listener into the internal network; has an HTTP-proxy mode with NTLM auth for proxied environments. Verify the proxy with one simple web request before scanning. Mind the Python 2 requirement on both ends. *(3 Q — gap.)* *Use: both.*

**11. Port Forwarding with Windows: Netsh** — No upload needed: Windows' built-in `netsh interface portproxy add v4tov4 listenport=… listenaddress=<pivot-IP> connectport=… connectaddress=<internal-IP>` forwards a port from a dual-homed Windows host; verify with `show v4tov4`, then connect (e.g. xfreerdp to pivot:port). A rule on the wrong listen address looks right but is unreachable. Remove the rule afterward. *(1 Q — gap.)* *Use: both.*

**12. DNS Tunneling with Dnscat2** — When only DNS gets out: dnscat2 hides an encrypted C2 channel inside DNS queries/TXT responses (server on your host under a domain you control; a pre-shared secret secures the session; the Windows client is a PowerShell module). Slow and voluminous — short sessions, minimal traffic. Defence: restrict outbound DNS to authorized resolvers; monitor TXT/volume anomalies. *(1 Q — gap.)* *Use: both.*

**13. SOCKS5 Tunneling with Chisel** — A single Go binary (HTTP transport secured with SSH) for restricted environments: server on the pivot + client on your host = SOCKS (default 1080) into the internal network; **reverse mode** flips it — server with `--reverse` on your host, client connects back from the pivot (`R:socks`). Practical notes: the binary is large to transfer, and build/glibc mismatches bite — check runtime deps, keep a prebuilt ready. *(1 Q — gap.)* *Use: both.*

**14. ICMP Tunneling with SOCKS** — If ping is allowed but little else: ptunnel-ng encapsulates traffic in ICMP echo request/reply pairs (server on the pivot, client locally), carrying an SSH session inside ICMP, on top of which a dynamic forward + proxychains runs. Needs root on both ends and matching builds; slow. Verify in Wireshark that traffic really is inside ICMP. Defence: restrict ICMP; watch for abnormal ICMP volume. *(1 Q — gap.)* *Use: both.*

**15. RDP and SOCKS Tunneling with SocksOverRDP** — A double pivot: RDP's Dynamic Virtual Channels (the side-channel that carries clipboard/audio) can carry a SOCKS proxy. Load the plugin DLL on the target (`regsvr32`), connect by RDP, confirm the local 1080 listener, route the next hop through Proxifier. Register the DLL *before* connecting; tune RDP experience settings on slow links. Defence: monitor RDP virtual-channel activity; limit which hosts accept RDP. *(1 Q — gap.)* *Use: both.*

**16. Skills Assessment (scenario)** — Full chain: a leftover **web shell** on an external host is the foothold; from it, enumerate, find credentials and hosts, and pivot hop by hop through internal networks to the **Domain Controller's** flag. The module's own advice: keep a network map and a next-hop plan, log credentials/routes as found, and expect each hop to need a *different* technique. *(7 Q — gap; scenario text only.)* *Use: engagement.*

**17. Detection & Prevention** — The defender's mirror: pivoting is visible only against a **baseline** — full asset inventory (hosts, locations, dual-homed machines, elevated accounts), a network diagram, DHCP/DNS records, application inventory — audited regularly. Then people (MFA, BYOD management, SOC + IR plan), process (asset/provisioning/change management), and technology (perimeter NGFW, DMZ hardening, IDS/IPS, segmentation, host IDS, logging). ATT&CK anchors: T1133 External Remote Services, T1021 Remote Services, T1571 Non-Standard Ports (attackers blend on 443/8443), T1572 Protocol Tunneling (watch beaconing in allowed protocols), T1090 Proxy, and Living off the Land (hardest — it uses what's already there). *Use: both.*

**18. Beyond this Module** — These techniques are daily pentest work and feed teammates' next steps (further exploitation through your tunnels, per-subnet persistence, C2, control bypass, exfiltration). If the networking felt shaky, the Introduction to Networking module is the named backfill; forward path: Active Directory modules (next in this path), Shells & Payloads, web-application modules, then boxes/Pro Labs practice and community write-ups. *Use: both.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account, and it is heavily hands-on: 14 interactive sections, each a different tunnel. Work them in order; §3 (SSH dynamic), §5 (Meterpreter), §11 (netsh) and §13 (Chisel) are the four you will reach for most.)*
