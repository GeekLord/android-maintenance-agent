# Installing and running

## 1. Prepare the phone

1. Settings > About phone > tap **Build number** 7 times (on Xiaomi it's *MIUI/HyperOS version*, on Samsung it's under *Software information*).
2. Settings > Developer options > turn on **USB debugging**. On Xiaomi also turn on *USB debugging (Security settings)* if you want the agent to change settings.
3. Connect the USB cable, choose *File transfer* if asked, and tap **Allow** on the debugging prompt. Tick *Always allow from this computer* only on your own computer.

## 2. Prepare the computer

Install Android platform-tools from [developer.android.com](https://developer.android.com/tools/releases/platform-tools) or with a package manager.

Windows (PowerShell):
```powershell
winget install Google.PlatformTools
```

macOS:
```bash
brew install --cask android-platform-tools
```

Linux (Debian or Ubuntu):
```bash
sudo apt install android-tools-adb
```

Check the connection:
```bash
adb devices -l
```

If the phone shows as `unauthorized`, unlock it and tap Allow. If nothing shows, try another cable or USB port, and on Windows install your phone maker's USB driver.

## 3. Install the skill (optional)

| Agent | Personal skills folder | Project skills folder |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| OpenAI Codex | `~/.agents/skills/` | `.agents/skills/` |
| Gemini CLI | `~/.gemini/skills/` or `~/.agents/skills/` | `.gemini/skills/` or `.agents/skills/` |

Clone the repo into one of those folders so the path ends in `android-maintenance-agent/SKILL.md`. Restart the agent, then ask for what you want ("check my phone for spyware").

Agents without skill support: paste `SKILL.md` or a file from `prompts/` into the chat.

## 4. Run it

- Start the agent in an **empty folder**. It writes `./<model>_<serial>_<YYYY-MM-DD>/` there with the report, backups, evidence and rollback scripts.
- Don't use a folder synced to the cloud (OneDrive, Google Drive, Dropbox, iCloud). The backup and evidence contain personal data.
- Keep the phone unlocked and connected until the agent says it's done.

## 5. Undo

- **Removed apps:** run `rollback_debloat.sh` (macOS or Linux) or `rollback_debloat.ps1` (Windows) from the output folder, or for one app:
  ```bash
  adb shell cmd package install-existing <package_name>
  ```
- **Changed settings:** run `rollback_settings.sh` or `rollback_settings.ps1`.
- A removed app comes back only while its APK is still on the system partition. A factory reset restores all preinstalled apps.
