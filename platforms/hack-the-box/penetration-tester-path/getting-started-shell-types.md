# Getting Started: Types of Shells

> Source: HTB Academy - Penetration Tester Path  
> Purpose: Organized source notes for authorized labs and assessments

## Why Shell Access Matters

Remote code execution may allow only one command at a time. A shell creates an interactive channel that supports system enumeration, file transfer, privilege escalation, and further authorized testing without repeatedly triggering the original vulnerability.

Credentialed services such as SSH on Linux and WinRM on Windows are usually preferable when valid credentials are available. Otherwise, an approved command-execution path may be used to establish a shell.

## Shell Types

| Type | Communication model | Main consideration |
|---|---|---|
| Reverse shell | Target connects to the tester's listener | Commonly works when outbound traffic is allowed |
| Bind shell | Target listens and the tester connects | Requires an inbound-reachable target port |
| Web shell | Commands arrive through HTTP requests to a server-side script | Uses the existing web service and remains as a file until removed |

## Reverse Shell Workflow

1. Determine the correct callback interface, such as the VPN interface in an HTB lab.
2. Start a listener on the attack host.
3. Execute an operating-system-appropriate callback command through the authorized RCE path.
4. Confirm the resulting user and host context.
5. Stabilize the terminal before further interaction.

Listener:

```bash
nc -lvnp 1234
```

Flag meanings:

| Flag | Purpose |
|---|---|
| `-l` | Listen for an incoming connection |
| `-v` | Display connection information |
| `-n` | Disable DNS resolution |
| `-p 1234` | Use local port `1234` |

Identify the VPN or local interface:

```bash
ip a
```

The callback IP must be reachable from the target. HTB targets normally connect to the tester's VPN address rather than an unrelated physical interface.

## Bind Shell Workflow

A bind shell listens on the target. The tester connects to the exposed port:

```bash
nc TARGET_IP 1234
```

Bind shells can be reconnected to while the listener remains active, but inbound firewall rules, NAT, segmentation, or routing may prevent access.

## Stabilizing a Linux Shell

Spawn a pseudo-terminal:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Background the shell with `Ctrl+Z`, then run locally:

```bash
stty raw -echo
fg
```

Press Enter as needed, then configure the remote terminal:

```bash
export TERM=xterm-256color
```

Obtain the local terminal dimensions in another terminal:

```bash
stty size
```

Apply the reported values remotely:

```bash
stty rows ROWS columns COLUMNS
```

If the local terminal becomes unusable after disconnecting:

```bash
stty sane
reset
```

## Web Shells

A web shell is a server-side script that accepts a command through an HTTP parameter, executes it on the host, and returns the output. Its language must match a server-side runtime available on the target.

Common default webroots include:

| Server | Common default webroot |
|---|---|
| Apache | `/var/www/html/` |
| Nginx | `/usr/local/nginx/html/` |
| IIS | `C:\inetpub\wwwroot\` |
| XAMPP | `C:\xampp\htdocs\` |

The exact application configuration may use a different document root. Confirm it rather than relying on defaults.

Web shells operate through the application's existing HTTP(S) port, but they are persistent files and therefore require explicit authorization, careful evidence handling, and removal during cleanup.

## Operational Notes

- Reverse shells are fast but usually disappear when the process or connection ends.
- Bind shells remain reconnectable only while the target listener continues running.
- Web shells may survive reboots because the script remains on disk.
- Confirm current user, hostname, network path, and shell stability immediately.
- Record every file written and port opened so cleanup can be verified.

## Promoted References

- [Unified Shells and Listeners Knowledge](../../../../knowledge-base/exploitation/shells-and-listeners.md)
- [Shells and Listeners Cheat Sheet](../../../../cheatsheets/shells.md)

> Use these techniques only in systems you own or have explicit authorization to test.
