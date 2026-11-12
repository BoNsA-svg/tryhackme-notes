# 7. File Transfers

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 10 / 10 sections
> **Outline metadata:** Medium, 10 sections, 5 interactive exercises, 0 assessments (as displayed); tier/time not displayed for this module
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 9 items deliberately not attempted (§2 ×3 and §3 ×3 — 2 questions + 1 optional exercise each; optional exercises only in §4, §5, §8) and no target spawned. This module's reader uses a "Mark Complete & Next" button instead of "Completed" — never pressed; navigation was Next-arrow only.
> **Account note:** Unlike Modules 1–6, this module was **not** already completed by me — HTB showed 10% (Section 1 done) before and after this read. The notes now run ahead of my own hands-on progress here.

Moving files to and from Windows and Linux targets using tools already present on those systems — a constant practical need once you have any level of access.

**Key concepts to own:** HTTP transfers; SMB transfers; FTP / TFTP; base64 / certutil workarounds.

## Deep summary — section by section (original words)

**1. File Transfers** — After a foothold you must move tools in and results out — but whitelisting, AV/EDR, firewalls and IDS/IPS block ports, tools and odd operations, so no single method works everywhere. Discipline: survey what the target and network allow → pick a matching technique → **verify integrity with a checksum** afterwards. *Use: both.*

**2. Windows File Transfer Methods** — Download: PowerShell `Invoke-WebRequest` / `Net.WebClient`, SMB (`impacket-smbserver` on your side + `net use`), FTP (`pyftpdlib`). Upload: base64-encode and paste, `Invoke-WebRequest`/`Invoke-RestMethod` to an upload server, WebDAV share, FTP. base64 is the universal adapter — binaries become text that survives command-line limits. Verify with a hash (md5sum / Get-FileHash). Pitfalls: the cmd.exe **8,191-character limit**, guest SMB blocked on newer Windows, untrusted certificates breaking Invoke-WebRequest. *(2 Q + 1 optional exercise — gap.)* *Use: both.*

**3. Linux File Transfer Methods** — Download: `wget`, `curl -o`, `base64 -d` after pasting, pure-Bash `/dev/tcp`, `scp`. Upload: base64, `curl` POST to an HTTPS upload server, a plain Python HTTP server, `scp`. HTTP(S) is the workhorse because it's usually allowed out; SSH is the encrypted alternative. Pitfalls: assuming outbound SSH/FTP is open; piping a download straight into bash uninspected. Verify checksums; keep the channel unremarkable. *(2 Q + 1 optional exercise — gap.)* *Use: both.*

**4. Transferring Files with Code** — When dedicated tools are missing, an interpreter almost always exists: Python (`urllib.request.urlretrieve`, `requests.post`), PHP (`file_get_contents`), Ruby (`Net::HTTP.get`), Perl (`LWP::Simple getstore`), JavaScript/VBScript on Windows (`WinHttpRequest`, `XMLHTTP` via cscript). Workflow: check what's installed → simplest one-liner that fetches → POST back to your upload server for uploads. Pitfall: one-liners that fail silently. *(Optional exercise — gap.)* *Use: both.*

**5. Miscellaneous File Transfer Methods** — netcat (raw TCP: listen one side, send from the other — no server software needed), PowerShell Remoting/WinRM (`New-PSSession` + `Copy-Item -ToSession/-FromSession` over 5985/5986), and RDP drive sharing (xfreerdp/rdesktop/mstsc folder mapping — remember the target's AV scans shared folders). Test any new channel with a harmless file first. *(Optional exercise — gap.)* *Use: both.*

**6. Protected File Transfers** — Sensitive output (user lists, credentials, enumeration results) travels **encrypted**: an encrypted channel (SSH/SFTP/HTTPS), or encrypt the file first, move it anywhere, decrypt and verify on arrival. Windows: Invoke-AESEncryption.ps1 (AES-256). Linux: `openssl enc -aes256 -iter 100000 -pbkdf2`. Rules: strong unique password per client; use dummy data when testing egress controls; never move real personal/financial data you weren't asked to. Defence side: monitor for large outbound transfers. *Use: engagement.*

**7. Catching Files over HTTP/S** — Run your own receiver: Nginx with WebDAV-style PUT (`dav_methods PUT`) on a chosen port — simpler and safer than Apache, where an uploaded file might execute as PHP. Workflow: create/own the upload dir → server block → restart → test with `curl -T` → confirm arrival → directory listing **off**. Pitfalls: the default config grabbing port 80 and colliding with other services; unwritable directories; check the error log first. Restrict who can upload and where files land. *Use: both.*

**8. Living off The Land** — Use binaries already on the box (LOLBins on Windows via the **LOLBAS** project, GTFOBins on Linux): a signed, trusted binary is rarely blocked by whitelisting and blends into normal activity. Examples: CertReq.exe `-Post`, OpenSSL `s_server`/`s_client`, BITS (`bitsadmin` / `Start-BitsTransfer`), `certutil -verifyctl -split -f`. Workflow: search LOLBAS/GTFOBins by download/upload function → confirm the binary exists on the target → invoke its transfer capability. Pitfalls: relying on one binary; version flag drift; defenders read these projects too. Defence: command-line logging + application control. *(Optional exercise — gap.)* *Use: both.*

**9. Detection** — The defender's view: every method leaves a distinctive command line and a distinctive HTTP **User-Agent** (Invoke-WebRequest, WinHttpRequest, Msxml2.XMLHTTP, Certutil, BITS…). Blacklisting bad command lines is easy to bypass (even case changes); **allow-listing known-good** command lines and user agents, baselined into a SIEM, is stronger — but only if the list is maintained and correlated with other signals. Knowing this shapes how quietly you choose methods. *Use: engagement (blue-team value).*

**10. Evading Detection** — Blend in: set a browser-like User-Agent (`Invoke-WebRequest -UserAgent`, matching what that machine would plausibly send — a mismatched or outdated UA stands out), and if the transfer tools themselves are blocked by whitelisting, switch to a LOLBin/GTFOBin channel. The principle: choose transfer methods deliberately, knowing the detection side. *Use: engagement (mindset transfers to CTF).*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is only 10% done on my account, so the §2/§3 questions and the optional exercises in §4, §5, §8 are first-time hands-on, not re-dos. Record which methods actually worked per target.)*
