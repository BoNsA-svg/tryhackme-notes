# CTF Ready
Use this part in competitions and practice boxes, where the goal is flags under a clock. Same skills as Part A, different discipline: speed, loops, and not getting stuck.

## B1. First 30 minutes on any box

1. Start full port scan in the background; work from the first results instead of waiting for it to finish.
1. While it runs: note OS hints, hostnames, and any web service — open the site, view source, check robots / common files.
1. Enumerate every open service in order, not just the web port. Write down versions exactly.
1. Start a running notes file: ports, creds found (and where), things tried that failed. Failed tries are data.

## B2. The stuck reset (use after 20 minutes with no progress)


☐  Re-read your enumeration — is there a port, vhost, share, or user you have not actually tried?

☐  Enumerate deeper, not wider: another wordlist, another tool, recursive, different extensions.

☐  Are you assuming a version is patched? Check the exact version against known issues.

☐  Check simple things: default creds, reused creds from elsewhere on the box, readable backup / config files.

☐  Switch surface: if web is dead, go back to services; if services are dead, go back to web.

## B3. CTF notes template (copy per box)

- Box / IP / date:   |   Ports & versions:
- Initial foothold — how, in one line:
- User flag path (your steps, your words):
- Privilege escalation path (your steps, your words):
- Creds & where each one came from:
- Dead ends worth remembering:
- One technique to add to the Knowledge Base:

## B4. Flag discipline

- Do not copy walkthroughs or other players' solutions into this note — your own path only, or it will not help you next time.
- Record the *class* of trick (e.g. 'writable service config'), not just the one box it worked on.
