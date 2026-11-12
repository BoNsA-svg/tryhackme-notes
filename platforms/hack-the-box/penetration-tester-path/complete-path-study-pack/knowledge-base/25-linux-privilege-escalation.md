# 25. Linux Privilege Escalation

> **Status:** Deep-complete (lesson text) — read in full 2026-11-10, 28 / 28 sections
> **Outline metadata:** Easy, Tier 2, 28 sections, 23 interactive exercises, 1 assessment (as displayed), est. 8 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 28 question entries deliberately not attempted (spread across §§2–25, plus the 5-question skills assessment §28) and no target spawned. Hint buttons appeared in §§4, 10, 18, 19 and 28 (Q1/Q2/Q4) and were deliberately not clicked. Every question prompt supplies lab SSH login details — none were copied anywhere (credential material, per the standing rule).
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Raising privileges on Linux by enumeration: users, sudo rights, SUID binaries, cron jobs, capabilities, writable files and services, kernel and software versions — misconfigurations first, exploits second.

**Key concepts to own:** enumeration order; sudo -l; SUID / capabilities; cron and writable paths; kernel version checks.

**Operating principle of the whole module:** enumerate manually and understand what you're looking at; misconfigurations first, version exploits second, kernel exploits last (they can crash the box). LinPEAS comes *after* the manual pass, not instead of it.

## Deep summary — by category (original words)

### Information gathering (§§1–4)

**1. Introduction** — Root = total control: traffic capture, sensitive files, and (on domain-joined hosts) a launchpad into AD. The enumeration checklist that structures everything after: distro/kernel, services (and *who they run as*), packages/versions, users, homes, .ssh dirs, bash history, sudo rights, configs, readable shadow, hashes sitting in /etc/passwd, cron, unmounted filesystems, SUID/SGID, writable files/dirs. *(Theory.)* *Use: both.*

**2. Environment Enumeration** — Orient first: whoami/id/hostname, network position (ip a, routes, resolv.conf, arp), sudo -l (read it *fully* — enterprise output is long, and one entry can end the escalation), os-release + uname -a, $PATH and env, shells available, block devices/fstab (unmounted filesystems hide data), users/groups/homes/hidden files. Note the OS end-of-life date while you're there. *(1 Q — gap.)* *Use: both.*

**3. Services & Internals Enumeration** — Second pass: what's installed and running *as root*, sockets in use, who's logged in (w/who/lastlog), password policy, AD-joined or not, history/log/backup files, recently modified files, /etc/hosts, tools present on the box. Cross-check every binary against **GTFObins** (there's a curl/jq lookup for binaries present) and every version against known issues. If ifconfig is missing, ip is the tool. *(1 Q — gap.)* *Use: both.*

**4. Credential Hunting** — Credentials hide in boring files: configs across the filesystem, web roots under /var (the wp-config pattern), scripts, bash history, backups, spool/mail, and **~/.ssh** (private keys + known_hosts, which maps where the user connects). Found password → try it for *all* users, not just its owner. *(1 Q — gap.)* *Use: both.*

### Environment-based (§§5–7)

**5. Path Abuse** — If a writable directory precedes the system dirs in PATH — or `.` is in PATH — a planted script named like a real command runs when a privileged user (or their cron job) calls the bare command name. Check PATH on every foothold; plant, chmod +x, wait. Defence: no `.` or user-writable dirs in privileged PATHs; absolute paths in privileged scripts. *(1 Q — gap.)* *Use: both.*

**6. Wildcard Abuse** — The shell expands wildcards *before* the command runs, so filenames in a directory a privileged command globs over become its **options**: the classic is a root cron `tar` over a writable home dir, with crafted filenames carrying tar's checkpoint options that execute a script. Fix: explicit file lists, never wildcards over user-writable dirs. *(Theory.)* *Use: both.*

**7. Escaping Restricted Shells** — rbash/rksh/rzsh each restrict differently; escapes recycle shell features: backtick substitution inside an argument, command substitution, chaining (`;`/`|`), environment variables, shell functions. Try them in turn — and expect per-account differences. Defence: restricted shells are one layer, never the boundary. *(1 Q — gap.)* *Use: both.*

### Permissions-based (§§8–11)

**8. Special Permissions** — SUID runs with the *owner's* rights (root-owned = root), SGID with the group's. Enumerate both (`find / -perm -4000` / `-6000`), ignore the standard set, interrogate the odd ones, and check GTFObins per binary/version (entries change). Audit and strip the bit where unneeded. *(2 Q — gap.)* *Use: both.*

**9. Sudo Rights Abuse** — `sudo -l` is the highest-value single command in the module. A permitted binary with an option that runs a program is root: the lesson's case is tcpdump's postrotate-command (note: modern AppArmor profiles may close exactly that route — version/context matters). Sudoers hygiene: absolute paths only (else PATH abuse applies), grant sparingly. *(1 Q — gap.)* *Use: both.*

**10. Privileged Groups** — Membership ≈ root for several groups: **lxd/lxc** (privileged container + host root mounted), **docker** (host root mounted in a container), **disk** (debugfs reads raw devices), **adm** (reads all logs — a credential source). `id` early; audit membership like you'd audit sudoers. *(1 Q — gap.)* *Use: both.*

**11. Capabilities** — Fine-grained privileges set with setcap; getcap enumerates them. A dangerous capability on an abusable binary (the lesson: an editor with cap_dac_override) writes root-owned files — here, blanking root's password field in /etc/passwd, then `su`. Treat capability-bearing binaries exactly like SUID binaries in audits. *(1 Q — gap.)* *Use: both.*

### Service-based (§§12–18)

**12. Vulnerable Services** — A root-running service with a known flaw: Screen 4.5.0's missing permission check on log-file opening → root-owned file creation → setuid shell via the preload mechanism. Workflow is version discipline: confirm version → matched public exploit → compile → run. Patch and run services unprivileged. *(1 Q — gap.)* *Use: both.*

**13. Cron Job Abuse** — Root cron + a **world-writable script** it runs = root on a timer. Find writable files; if the crontab is unreadable, **pspy** watches processes to prove which script root runs and when. Always back up the script before appending your payload, and restore it after. Scheduling typos (every 3 minutes vs 3 hours) are themselves findings. *(1 Q — gap.)* *Use: both.*

**14. LXD** — Privileged containers have **no UID mapping**: root inside = root on the host, with the host filesystem mounted in. Import image → init with security.privileged=true → mount host / → exec. Group membership is the whole vulnerability. *(1 Q — gap.)* *Use: both.*

**15. Docker** — Same shape: mount host root into a container and chroot, or abuse a writable **/var/run/docker.sock** to do it. Mounted host filesystems leak credentials (the lesson pulls an SSH private key from a mounted home dir — key material never reproduced here). Docker group = root; never expose the socket. *(1 Q — gap.)* *Use: both.*

**16. Kubernetes** — Kubelet API (10250; read-only 10255) allows **anonymous access by default**: list pods, exec into containers, lift the service-account token/cert, check its rights, then create a pod mounting the host root → host files incl. root's SSH key. API server on 6443. Fixes: anonymous access off, RBAC least privilege, restrict pod creation/host mounts, rotate tokens. *(Theory.)* *Use: engagement.*

**17. Logrotate** — The logrotten flaw needs a specific trio: a **writable log file**, logrotate **running as root**, and a **vulnerable version** (3.8.6 / 3.11.0 / 3.15.0 / 3.18.0) with the create option in play. Check all three before spending time; compile the exploit on a similar kernel and transfer it. *(1 Q — gap.)* *Use: both.*

**18. Miscellaneous Techniques** — Three classics: **tcpdump present** → unprivileged capture → cleartext credentials/hashes from the wire (net-creds/PCredz); **NFS no_root_squash** → plant a setuid binary on the share as local root, execute it on the host (root_squash is the fix and the failure mode); **tmux** → a privileged session on a shared socket your group can reach is a shell as that user (`tmux -S`). *(1 Q — gap.)* *Use: both.*

### Internals-based (§§19–22)

**19. Kernel Exploits** — Match the exploit to `uname -a` exactly (Dirty COW CVE-2016-5195 is the archetype). Legacy/unpatched kernels persist because compatibility freezes patching. Discipline: understand the exploit before running it, lab-test when possible, prefer config fixes on real engagements — a crashed production box is your finding too. *(1 Q — gap.)* *Use: both.*

**20. Shared Libraries — LD_PRELOAD** — If sudo **preserves LD_PRELOAD** (env_keep), any permitted sudo command becomes root: compile a small .so whose initializer sets uid/gid 0 and execs a shell, then `sudo LD_PRELOAD=… <permitted command>`. An absolute path in sudoers does **not** stop this. Fix: never env_keep LD_PRELOAD. *(1 Q — gap.)* *Use: both.*

**21. Shared Object Hijacking** — A setuid binary loading a library from a **writable RUNPATH** directory: ldd shows the dependency, readelf confirms the path, drop in a replacement .so exporting the *exact* function the binary expects (learn it from the undefined-symbol error) → runs as root. Wrong symbol name = the binary just fails. *(1 Q — gap.)* *Use: both.*

**22. Python Library Hijacking** — Three routes when a root-run (via sudo) Python script imports modules: write access to an **installed module** (inject at the top of the called function), a **higher-priority writable directory** in sys.path (same-named module shadows the real one), or sudo **SETENV** letting you set PYTHONPATH. Filename must match exactly; a setuid bit on a .py file itself does nothing — sudo is the vehicle. *(1 Q — gap.)* *Use: both.*

### Recent 0-days (§§23–26)

**23. Sudo** — **Baron Samedit CVE-2021-3156** (heap overflow, a decade hidden — match the PoC to the distro build) and **CVE-2019-14287** (a permitted "run as any user except root" is bypassed with `-u#-1`, which sudo resolves to UID 0). sudo -V is the check. *(1 Q — gap.)* *Use: both.*

**24. Polkit — PwnKit CVE-2021-4034** — pkexec memory corruption, unprivileged → root; compile the PoC for the target environment. Know the layout: actions in /usr/share/polkit-1/actions, rules.d, local authority overrides. *(1 Q — gap.)* *Use: both.*

**25. Dirty Pipe CVE-2022-0847** — Kernels 5.8–5.17: a pipe write lands in a **read-only file's page cache** — write to files you can only read. Two endings: blank root's password in /etc/passwd (restore the file after) or hijack a SUID binary into a setuid shell (**clean the shell up** — leaving one behind is a critical own-goal). Also affects Android. *(1 Q — gap.)* *Use: both.*

**26. Netfilter** — A run of kernel flaws in the packet-filtering subsystem: CVE-2021-22555 (2.6–5.11), CVE-2022-25636 (5.4–5.6.10, can corrupt the kernel → reboot), CVE-2023-32233 (≤6.3.1, use-after-free, unstable exploits). Containers/VMs don't hide the host kernel. Version match first; stability risk is real. *(Theory.)* *Use: both.*

**27. Linux Hardening** — The defence mirror of the whole module: patch automatically (unattended-upgrades / yum-cron); audit writable files, SUID binaries, cron and sudo entries (absolute paths everywhere); no cleartext credentials in world-readable files; clean homes and histories; user management (few admins, no password reuse via PAM/opasswd, privileged-group discipline); baseline audits (STIG/ISO/PCI) with **Lynis** (hardening index + findings like GRUB passwords and cron file permissions). An audit complements a pentest — it never replaces one. *(Theory.)* *Use: engagement.*

**28. Skills Assessment (scenario)** — A client-provided low-privilege user on a public web server; **five flags at ascending privilege levels**, student → root, chaining the module's techniques in whatever order the host offers. Methodical enumeration and per-level documentation are the graded skills. *(5 Q — gap; scenario text only; its SSH login details were not copied.)* *Use: both.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. Work §§2–4 until the manual enumeration checklist is muscle memory before leaning on LinPEAS; the assessment (§28) is five chained escalations, so keep a running log of every candidate weakness from the first command — most chains are won in the enumeration notes, not at the exploit step.)*
