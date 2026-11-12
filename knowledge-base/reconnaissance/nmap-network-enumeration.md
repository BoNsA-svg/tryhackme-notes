# Nmap Network Enumeration

> **Methodology context:** Use this material through the [Reconnaissance Methodology](../../pentesting-methodology/reconnaissance/reconnaissance-methodology.md).

## Enumeration Mindset

Enumeration converts reachable systems into testable hypotheses. A scanner result is the start of analysis, not the conclusion. For every finding, ask:

1. What caused this result?
2. What does the protocol response prove?
3. What remains unknown?
4. How can the result be confirmed manually?
5. Does the finding expose information, functionality, or a path to another system?

Manual interaction matters because timeouts, filtering, rate controls, and unusual service behavior can cause automated tools to miss or misclassify targets.

## Progressive Nmap Workflow

```bash
export TARGET=<TARGET_IP>
mkdir -p nmap

# 1. Determine reachability; skip discovery with -Pn when justified
sudo nmap -sn --reason $TARGET -oA nmap/discovery

# 2. Find TCP exposure
sudo nmap -sS -p- $TARGET -oA nmap/tcp-all

# 3. Enumerate confirmed services
sudo nmap -sC -sV -p <OPEN_PORTS> $TARGET -oA nmap/services

# 4. Check relevant UDP services
sudo nmap -sU --top-ports 100 $TARGET -oA nmap/udp-top100
```

Do not blindly use speed flags. Select timeouts, retries, rates, and timing templates according to bandwidth, scope, stability, detection requirements, and the cost of false negatives.

## Scan Interpretation

| Observation | Supported conclusion | Required follow-up |
|---|---|---|
| SYN-ACK | TCP port is open | Identify and validate the service |
| RST | TCP port is closed but host responded | Record reachability; test other ports |
| No response | Possibly filtered or rate-limited | Change discovery/probe type and retry carefully |
| ICMP unreachable | Filtering or closed UDP/TCP state depending on code | Inspect `--reason` and packet trace |
| Banner/version string | Claimed service identity | Confirm behavior and exact build |

Use `--reason` to understand classifications and `--packet-trace` or packet capture when the result is important or confusing.

## Discovery Context

- Local-subnet discovery frequently uses ARP, even when ICMP options are supplied.
- ICMP silence is not proof that a host is offline.
- `-Pn` skips host discovery and treats targets as online; it can reveal services behind blocked discovery probes but increases scan work.
- Scan lists with `-iL` and preserve evidence with `-oA`.

## TCP, UDP, and Service Detection

- SYN (`-sS`) is fast and does not complete the handshake.
- Connect (`-sT`) uses the OS networking stack and completes the handshake.
- ACK (`-sA`) maps filtering behavior; it does not identify open services.
- UDP (`-sU`) is slower and silence is ambiguous, so combine it with version probes and protocol knowledge.
- Service detection (`-sV`) sends protocol-aware probes; validate high-value findings with `nc`, `curl`, `openssl s_client`, or the native client.

## NSE Safety

NSE extends Nmap from discovery into service interaction. Choose scripts individually when possible and review what they do before execution.

| Lower-impact starting points | Higher-impact categories requiring explicit approval |
|---|---|
| `default`, `safe`, `discovery`, `version` | `brute`, `intrusive`, `exploit`, `dos`, `fuzzer` |

A script category name is not a substitute for reading the script documentation and evaluating production impact.

## Performance Trade-offs

| Control | Benefit | Risk |
|---|---|---|
| Short RTT timeouts | Faster scans | Slow responses are missed |
| Fewer retries | Faster completion | Filtered or lossy hosts disappear |
| High `--min-rate` | Predictable throughput | Congestion, detection, inaccurate results |
| `-T4`/`-T5` | Faster timing | More noise and possible blocking |

Maintain a slower baseline when accuracy matters. If an optimized scan finds less, treat the difference as a measurement problem rather than proof that services vanished.

## Firewall and Detection Analysis

Compare scan types and their `--reason` output to infer filtering:

- SYN filtered but ACK unfiltered suggests a rule treating connection initiation differently.
- Responses that change with source port may reveal simplistic source-port rules.
- Decoys complicate attribution but do not conceal the tester or guarantee evasion.
- IDS/IPS evaluation must follow the Rules of Engagement; stealth is a testing requirement, not permission to bypass controls.

## Evidence Standard

For every important finding, retain:

- Exact command, date/time, interface, and source position
- Raw `.nmap`, `.gnmap`, and `.xml` output
- Open ports, protocols, service versions, and confidence
- `--reason` or packet evidence for ambiguous results
- Manual confirmation and any discrepancy between tools
- Rate/timing changes that could affect completeness

## Related Notes

- [Active Reconnaissance](active-recon.md)
- [Network Security](../network-security/network-security.md)
- [Service Enumeration Cheat Sheet](../../cheatsheets/services.md)
- [Recon Cheat Sheet](../../cheatsheets/recon.md)

## Source Contributions

- Hack The Box Academy — Network Enumeration with Nmap

> Use only in authorized labs, CTFs, and explicitly scoped assessments.
