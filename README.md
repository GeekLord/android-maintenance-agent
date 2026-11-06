# Android Maintenance Agent

**Find spyware, remove bloatware and health-check any Android phone with an AI agent. No root, no app to install, and nothing changes without your approval.**

Plug your phone into a computer, open an AI coding agent with terminal access (Claude Code, OpenAI Codex, Gemini CLI, Antigravity or similar) and let it work through ADB like a careful Android engineer would: it reads first, explains what it found in plain language, asks before every change, and writes a rollback script for everything it touches.

## What it does

| Service | What you get | Changes the phone? |
|---|---|---|
| **Spyware and threat scan** | Hunts for stalkerware, banking trojans, hidden or disguised apps, apps abusing accessibility, notification access or device admin, traffic interception (proxy, VPN, Private DNS), remote access, root and tampered firmware. Optional checks with spytrap-adb, AndroidQF + MVT and VirusTotal hash lookups. | No, unless you approve a fix |
| Health check | Storage, battery wear and drain, RAM, thermal, app usage, privacy and security settings | No |
| Backup | Shared storage (verified file by file), contacts, call log, SMS and app list, copied to your computer | No |
| Debloat | Removes OEM, carrier and partner bloat in safe tiers, reversible | Yes |
| Storage cleanup | Caches, leftovers, app media, downloads, as you choose (after a verified backup) | Yes |
| Optimization | Battery, speed and privacy settings, each one recorded and reversible | Yes |
| Unused-app removal | Apps you haven't used for a while | Yes |

## Pick a prompt

| File | Use it when | Size |
|---|---|---|
| [`prompts/security-scan.md`](prompts/security-scan.md) | You suspect spyware, someone knows things they shouldn't, or you want a routine security check | Medium |
| [`prompts/quick-debloat.md`](prompts/quick-debloat.md) | You only want to remove bloatware, fast | Short |
| [`SKILL.md`](SKILL.md) | You want everything: audit, scan, backup, debloat, cleanup, optimization and a full report | Long |

## Quick start

**1. On the phone:** Settings > About phone > tap *Build number* 7 times. Then Settings > Developer options > turn on **USB debugging**.

**2. On the computer:** install [Android platform-tools](https://developer.android.com/tools/releases/platform-tools) and check that `adb` works.

Windows (PowerShell):
```powershell
winget install Google.PlatformTools
adb version
```

macOS:
```bash
brew install --cask android-platform-tools
adb version
```

Linux (Debian or Ubuntu):
```bash
sudo apt install android-tools-adb
adb version
```

**3. Connect the phone** with a USB cable and tap **Allow** on the USB debugging prompt.

**4. Run a prompt.** Open your agent in an empty folder (the agent saves reports, backups and rollback scripts there) and paste the prompt you picked. It starts by asking what you want, then waits for your answer.

## Install as a skill

`SKILL.md` follows the open Agent Skills format, so agents that support skills load it automatically when you ask for something it covers ("scan my phone for spyware", "debloat my Samsung").

Claude Code:
```bash
git clone https://github.com/GeekLord/android-maintenance-agent ~/.claude/skills/android-maintenance-agent
```

OpenAI Codex and Gemini CLI (shared skills folder):
```bash
git clone https://github.com/GeekLord/android-maintenance-agent ~/.agents/skills/android-maintenance-agent
```

Windows (PowerShell), Claude Code:
```powershell
git clone https://github.com/GeekLord/android-maintenance-agent "$HOME\.claude\skills\android-maintenance-agent"
```

Other agents (Antigravity, Cursor and the rest): paste the contents of `SKILL.md` or one of the prompts into the chat. See [docs/install.md](docs/install.md) for details.

## How it keeps your phone safe

- **Asks first.** It shows a menu and waits before running a single command.
- **Reads before it changes.** The audit and the spyware scan are read-only.
- **Approval gates.** Every change is a numbered list you approve, row by row if you like.
- **Rollback for everything.** Removed apps come back with `cmd package install-existing`, and every setting is recorded before it changes and restored by a script.
- **Backup before delete.** Nothing is deleted that isn't in a verified backup.
- **Protected packages.** Telephony, System UI, the camera, Play Services, device admins and finance locks (like Samsung Knox Guard), mainline modules and setup wizards are never touched unless you insist.
- **Stays on your computer.** Reports, evidence and backups are written to a local folder and kept out of git. Your agent's model provider still sees the commands and their output, so read your provider's data policy.

## What it can't do

ADB without root can see installed apps, their permissions and what they are allowed to do, settings, running processes and some logs. It **cannot** read other apps' private data, inspect the kernel or firmware, or reliably detect advanced zero-click spyware. A clean scan lowers the risk; it doesn't prove the phone is clean. AI recommendations can be wrong, so read each item before you approve it.

**If you think a partner, family member or employer is spying on you,** removing the app can alert them. Plan from a different, trusted device first, and consider contacting the [Coalition Against Stalkerware](https://stopstalkerware.org) or a local support service. The scan prompt walks you through this.

## Requirements

- An Android phone or tablet running Android 8 or newer
- A Windows, macOS or Linux computer with a USB cable
- An AI agent that can run terminal commands

## Credits

Inspired by the debloat prompt in [Remove Android Bloatware Using AI (No Root)](https://exposureee.in/remove-android-bloatware-using-ai/) by EXPOSUREEE. This project expands it into a full maintenance and security agent with intake, approval gates, backups, settings rollback and a spyware scan. Debloat recommendations draw on the community knowledge behind [Universal Android Debloater Next Generation](https://github.com/Universal-Debloater-Alliance/universal-android-debloater-next-generation). Threat checks can use [spytrap-adb](https://github.com/spytrap-org/spytrap-adb), the [Echap stalkerware indicators](https://github.com/AssoEchap/stalkerware-indicators) and Amnesty International's [MVT](https://github.com/mvt-project/mvt) with [AndroidQF](https://github.com/mvt-project/androidqf).

## Contributing

Ran it on your phone? [Open a device report](../../issues/new?template=device-report.yml) with your model, Android version, agent and what happened. Reports like that make the debloat lists and OS notes better for everyone. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Disclaimer

Provided as is, under the [MIT License](LICENSE). You are responsible for what you approve on your device. Package safety varies by manufacturer, Android version, region and firmware. Keep the rollback scripts and backups until you are sure everything works.

Made by [Shobhit Kumar Prabhakar](https://www.geeklord.com/about/) at [GeekLord.com](https://www.geeklord.com/).
