# Device report: <Brand> <Model>, Android <version>

<!--
Standard device report for Android Maintenance Agent.
The agent fills this in at the end of a run. Every value must come from the run itself; write "not run" or "unknown" instead of guessing.
Never include: serial number, IMEI, phone number, email or account names, Wi-Fi names, contact or message content, file or folder names from your storage, or the names of apps you installed yourself.
-->

## Device and setup

| Field | Value |
|---|---|
| Brand and model | |
| Model code (`ro.product.model`) | |
| Android version (SDK) | |
| OEM skin and version | |
| Security patch | |
| Region or CSC | |
| Root | No / Yes (which) |
| Agent and model | e.g. Claude Code, Claude Sonnet |
| Computer OS and shell | e.g. Windows 11, PowerShell 7 |
| Prompt and version | SKILL.md / security-scan.md / quick-debloat.md, version from CHANGELOG |
| Date of run | YYYY-MM-DD |

## Services run

- [ ] Health check
- [ ] Spyware and threat scan
- [ ] Backup
- [ ] Debloat
- [ ] Storage cleanup
- [ ] Unused-app removal
- [ ] Optimization
- [ ] Restart and verification

## Results

| Metric | Before | After |
|---|---|---|
| Free internal storage | | |
| System packages (enabled) | | |
| User apps | | |
| Packages removed | | total (Tier 1 / Tier 2 / Tier 3) |
| Settings changed | | count |
| Battery health estimate | | |

## System packages removed

| Package | Tier | What it was | Any problem afterwards? |
|---|---|---|---|
| | | | |

## Spyware and threat scan

| Group | Count |
|---|---|
| Confirmed | |
| Suspicious | |
| Notable | |

- Threat types found (for example stalkerware, banking trojan, interception, remote access), without app names unless the user agreed:
- Level 2 tools run (spytrap-adb, AndroidQF + MVT, VirusTotal hashes, Play Protect):
- Remediation applied (R1 to R9):
- Possible false positives (system or well-known packages wrongly flagged):

## Rollback

- Rollback scripts tested: Yes / No
- Result:

## What went wrong

| Step | Command or recommendation | What happened | How it was fixed |
|---|---|---|---|
| | | | |

## OS or OEM quirks worth adding to Appendix B

-

## Time and cost

- Total time with the phone connected:
- Tokens or cost reported by the agent (if shown):

## Privacy check

- [ ] No serial number, IMEI or phone number
- [ ] No email or account names
- [ ] No Wi-Fi names, contacts, messages, file or folder names
- [ ] No names of apps I installed myself (unless I chose to share a threat)
