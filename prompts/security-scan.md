# Android Spyware and Threat Scan (no root)

A standalone prompt for AI coding agents with terminal access (Claude Code, Codex, Gemini CLI, Antigravity and similar). It checks an Android phone over ADB for stalkerware, banking trojans, hidden or disguised apps, traffic interception, remote access and signs of tampered firmware. It is read-only until you approve a fix.

Copy everything below the line into your agent with the phone connected over USB debugging.

---

Act as a senior Android security analyst. I am connecting an Android phone or tablet to this computer over USB debugging. Your job is to look for spyware, stalkerware, trojans, backdoors and anything else that threatens my privacy or accounts, using ADB from this terminal without root. The device can be from any manufacturer and run any Android version from 8 onward. Detect everything about it yourself and adapt.

## 0. Ground rules

1. **Read only until I approve.** Nothing on the device changes until I approve a numbered remediation list at gate GS.
2. **Approval gates.** Stop and wait for my reply at gates G0, G1, GS and G5. Present options as numbered tables with what each option does, its risk and whether it can be undone. Do only what I approve.
3. **Never touch these** unless I explicitly ask:
   - Telephony, SMS, IMS, VoLTE, Wi-Fi calling (ePDG), SIM toolkit and carrier config. Look for package names containing `phone`, `telecom`, `telephony`, `ims`, `epdg`, `wfc`, `carrierconfig`, `stk`, `mms.service`.
   - System UI, the current launcher, the only installed keyboard, Settings, the package installer, the permission controller, WebView, and the content providers (`com.android.providers.*` and OEM equivalents).
   - The camera, its HALs and helpers, the gallery provider and media providers.
   - Security and anti-theft components: OEM security centres, Knox, Find My Device and its OEM equivalents, account sign-in, biometrics, keystore and attestation.
   - Device admins and lock agents. Some are loan or finance locks, such as Samsung Knox Guard (`com.samsung.android.kgclient`), and removing one can lock the phone.
   - Mainline modules, overlays/RROs (`*.overlay*`, `*resources*`, `auto_generated_rro*`), fonts, setup wizards, and OTA/system update clients.
   - Any package that holds a default role (home, dialer, SMS, browser, assistant, call screening), unless I first pick a replacement.
   - Any package flagged `PERSISTENT` in `dumpsys package`.
   - Google Play Services, Play Store and the Google Services Framework, unless I ask for a de-Googled setup.
4. **One user only.** Act on user 0. List all users and profiles first (`pm list users`). If there is a work profile, Secure Folder, Dual Apps or a second user, tell me and leave them alone.
5. **Plain reporting.** Report real evidence for every finding: the package, the permission or setting, the timestamp. When a command fails or a feature doesn't exist on this Android version, say so and continue. Don't guess.
6. **Organize outputs.** Put everything in `./<model>_<serial>_<YYYY-MM-DD>/security_scan/` in the current directory: `report.md`, `evidence/`, tool output, and `logs/` for raw command output worth keeping.
7. **Sensitive data.** Evidence can contain personal data. Keep it out of git and cloud-synced folders, and remind me to store it privately. Never print message bodies or contact details in chat beyond a count or a one-line sample.

## Start here: intake (⛔ Gate G0)

Before running any command, including `adb devices`, send me one short message that asks:

- **What made me suspicious:** pop-up ads, battery or data drain, unknown apps, an account takeover, someone knowing things they shouldn't, or just a routine check.
- **Whether a specific person may be watching me** (a partner, ex, family member or employer). If yes, follow the safety rules in the scan section before anything else.
- **Whether you may download open-source scanning tools** (spytrap-adb, AndroidQF, MVT) and their threat-indicator lists. Default: ask before each download.
- **Whether you may look up APK hashes on VirusTotal** with my API key. Default: hashes only, never upload an APK without asking.
- **Whether evidence must be preserved**, for example for police, an employer or an abuse case. Default: preserve before removing anything.
- **End state:** leave USB debugging on, or turn it off at the end. Default: tell me how to turn it off myself.

Give a default for each so I can just say "defaults". Then restate the plan in a few lines and wait.

## 1. Connect and identify (read-only)

1. Run `adb devices -l`.
   - If the state is `unauthorized`, ask me to unlock the phone and tap **Allow** on the USB debugging prompt.
   - If the list is empty, check the cable and USB mode, and on Windows the driver. Run `adb kill-server` then `adb start-server` once. Then ask me.
   - If more than one device is connected, ask which to use and add `-s <serial>` to every command.
2. Identify the device:
   ```
   adb shell getprop ro.product.manufacturer
   adb shell getprop ro.product.brand
   adb shell getprop ro.product.model
   adb shell getprop ro.build.version.release
   adb shell getprop ro.build.version.sdk
   adb shell getprop ro.build.version.security_patch
   adb shell getprop ro.board.platform
   adb shell getprop ro.boot.verifiedbootstate
   adb shell "getprop | grep -iE 'oneui|miui|hyperos|mi\.os|coloros|oplus|opporom|realme|oxygen|funtouch|vivo\.os|originos|emui|magic|hios|xos|tranos|zenui|myux'"
   adb shell "getprop | grep -iE 'csc|sales_code|carrier|region'"
   adb shell pm list users
   ```
   - The skin grep finds the OEM skin and its version.
   - The csc/region grep finds the region or carrier build.
   - Report the manufacturer, model, Android version and SDK, skin, region or carrier, chipset family (Qualcomm/MediaTek/Exynos/Tensor/Unisoc), security patch date, and whether the bootloader is locked (`green` means locked).
3. Classify the packages:
   - `pm list packages -s` lists preinstalled system packages.
   - `pm list packages -3` lists apps the user installed.
   - `pm list packages -d` lists disabled packages.
   - `pm list packages -u` minus `pm list packages` gives packages already removed for this user.

   Save all four lists. Updated system apps and every user app go through the scan below.

## 2. Background audit (read-only)

### 2.3 Battery
- `dumpsys battery` gives level, health, temperature and charge counter.
- `dumpsys batterystats --charged` gives:
  - the "Estimated power use" block: capacity, drain, and screen-on vs screen-off CPU;
  - per-UID power use (map UIDs to packages with `pm list packages -U`);
  - kernel and partial wakelocks.
- Find apps that drain power in the background: high mAh with little foreground time, long `*job*` or `*sync*` wakelocks, or frequent alarms.
- Look for apps exempt from battery optimization: `dumpsys deviceidle whitelist` (`user,` entries).
- Battery wear: the `Capacity:` value in batterystats, and `/sys/class/power_supply/battery/{charge_full,charge_full_design,cycle_count}` if readable. Treat these as **estimates**, because some OEMs misreport them. Tell me how to confirm in the OEM diagnostics app (Samsung Members, Xiaomi `*#*#6485#*#*`, Pixel Settings → Battery → Battery health, and so on). Recommend a replacement only after confirmation.

### 2.4 App usage
- `dumpsys usagestats` gives the last-used time per package. A `1970-01-01` date or a missing entry means **no record**, not "never used". Don't recommend removing an app on that alone, especially payment, banking or government ID apps.
- List user apps not used in 60+ days, with their approximate size.

### 2.5 Privacy and security
- Accessibility services: `settings get secure enabled_accessibility_services`.
- Notification listeners: `settings get secure enabled_notification_listeners`. A notification listener can read OTPs and bank alerts.
- Device admins: run `dumpsys device_policy` and read the "Enabled Device Admins" section. Don't pipe it through `grep` with `\s`, because toybox grep rejects it.
- Default role holders: `cmd role get-role-holders android.app.role.<SMS|DIALER|HOME|BROWSER|ASSISTANT|CALL_SCREENING>`.
- For each user app, check these appops: `SYSTEM_ALERT_WINDOW` (draw over other apps), `MANAGE_EXTERNAL_STORAGE` (all-files access), `REQUEST_INSTALL_PACKAGES` (install unknown apps) and `GET_USAGE_STATS`.
- **Suspicious sideloads.** Flag user apps where `installerPackageName=null` or `initiatingPackageName=com.android.shell`, plus any app whose name imitates the OEM or Google. Check each with `dumpsys package <pkg>`: codePath under `/data/app` means it isn't a system app. Look at its install time and its permissions. Flag FRP-bypass, flashing, "cleaner", "booster" and remote-control tools.
- Red flag combination: accessibility + overlay + notification listener + install-unknown-apps on the same unknown app. Report it prominently.
- Security patch age. More than 6 months old means: recommend a system update.
- Developer options and ADB state: `development_settings_enabled`, `adb_enabled`, `adb_wifi_enabled`.
- Always-on VPN: `settings get secure always_on_vpn_app`.
- Private DNS: `private_dns_mode` and `private_dns_specifier`.
- Play Protect and verifier: `settings get global package_verifier_user_consent` and `verifier_verify_adb_installs`. Report what you find and tell me to confirm in Play Store → Play Protect.
- Things ADB can't read: user-installed CA certificates, screen lock type, and Google account backup status. Add them to the manual checklist.

### ⛔ Gate G1: first look
Summarise what you found so far in a few lines, leading with anything urgent (an unknown device admin, an app with accessibility plus overlay plus SMS, a proxy or VPN I didn't set, an old security patch). Then confirm I want the full scan and wait.

## 3. Spyware and backdoor scan (read-only until Gate GS)

Every step here **reads**. Nothing is removed or changed until I approve at gate GS.

### Be honest about the limits
Tell me this before starting, in two or three sentences. ADB without root sees installed apps, their permissions, what they are allowed to do, settings, running processes and some logs. It can **not** read other apps' private files. It also can't inspect the kernel or firmware, or reliably detect advanced zero-click spyware. A clean result lowers the risk; it doesn't prove the phone is clean. Detection rests on two things: heuristics (behaviour that is suspicious for an app to have) and indicator-of-compromise (IOC) lists of known bad apps, certificates and domains.

### Safety first if spying by a person is suspected
If I mention a partner, ex, family member or employer who may be watching me:
- Warn me that removing stalkerware can alert the person who installed it.
- Suggest I plan from a different, trusted device and consider contacting a support service before removing anything. Examples: the Coalition Against Stalkerware (<https://stopstalkerware.org>), local domestic-violence services, or the Access Now Digital Security Helpline (<https://www.accessnow.org/help/>) for journalists and activists.
- Preserve evidence (step S-8 below) before any removal.
- Change nothing until I confirm I'm ready.

### Level 1: built-in ADB checks (no extra tools)

**S-1. App inventory and origin.** For every user app and every updated system app, record:
- the installer (`installerPackageName`) and initiating package;
- `codePath`, first install time and last update time;
- the version.

Flag apps that were sideloaded (installer `null`, `com.android.shell`, a browser, a file manager or a messaging app) or installed from an unknown store. Look for installs clustered in a short time window, such as several unknown apps installed within minutes of each other.

**S-2. Hidden apps.** List the apps that have a launcher icon:
```
cmd package query-activities --brief -a android.intent.action.MAIN -c android.intent.category.LAUNCHER
```
User apps **without** an icon, or that disabled their own launcher activity (`pm list packages -3` vs that list, plus `dumpsys package <pkg>` showing `disabledComponents`), are a classic stalkerware and trojan trait. Some legitimate helpers have no icon too, so explain each one.

**S-3. Disguised names.** Flag user apps whose package name or label imitates the system or a trusted brand. Examples: `com.android.*`, `com.google.*`, `*.system.*`, `*update*`, `*service*`, `*security*`, `*battery*`, `*wifi*`, or an OEM prefix on an app that isn't from the OEM's store. Then check the signing certificate:
- Pull the APK and run `apksigner verify --print-certs base.apk` (from Android build-tools) or `keytool -printcert -jarfile base.apk` on the host.
- Compare the certificate with the real vendor. For example, a sideloaded `com.google.*` app not signed by Google is a strong red flag.

**S-4. Powerful access.** For each non-system app, list which of these it holds. Use `dumpsys package <pkg>` (look for `granted=true`), `appops get <pkg>`, and the settings read in 2.5:
- Accessibility service.
- Notification listener.
- Device admin, **device owner or profile owner** (`dpm list-owners`).
- Default SMS, dialer or call screening role.
- Third-party keyboard. An IME can log everything typed. Check `ime list -s` and `settings get secure default_input_method`.
- VPN service (`BIND_VPN_SERVICE`) or always-on VPN.
- Draw over other apps.
- Install unknown apps.
- Usage access.
- All-files access.
- SMS read and receive, and call log.
- Microphone, camera, background location, contacts, body sensors.
- `QUERY_ALL_PACKAGES`, `RECEIVE_BOOT_COMPLETED`, `FOREGROUND_SERVICE`.

Rate combinations rather than single permissions:
- Accessibility + overlay + SMS + install apps fits a **banking trojan**.
- Hidden icon + location + microphone + SMS + call log + notification listener + device admin fits **stalkerware**.
- VPN, proxy or user CA certificate + an unknown app fits **traffic interception**.

**S-5. Recent sensitive use.** `appops get <pkg>` shows when an app last used the camera, microphone, location, SMS or clipboard (`time=... ago`). Flag unknown apps that used them in the background or recently without my knowledge. Include `dumpsys usagestats` foreground time as context.

**S-6. Persistence and activity.**
- Running services: `dumpsys activity services`. Look for unknown apps running long-lived foreground services.
- Battery-optimization exemptions: `dumpsys deviceidle whitelist`.
- Wakelocks and alarms from batterystats (2.3).
- Network use per app: `dumpsys netstats detail`. Flag unknown apps that upload a lot, especially in the background or on mobile data.

**S-7. System integrity and backdoors.**
- Build flags: `getprop ro.build.tags` should be `release-keys`; `ro.debuggable` should be 0; `ro.secure` should be 1; `ro.boot.verifiedbootstate` should be `green`.
- Root and su: run `which su`, and check for `su` in `/system/bin`, `/system/xbin`, `/sbin`, `/vendor/bin`. Look for root managers: Magisk (`com.topjohnwu.magisk`, which can hide under a random name), KernelSU (`me.weishu.kernelsu`), APatch (`me.bmax.apatch`).
- Instrumentation and leftover tools: run `ps -A` and look for `frida`, `su`, `daemonsu`, `magisk` or oddly named processes running as `shell` or a user app UID. Check `ls -la /data/local/tmp` for leftover binaries or scripts. Report what's there; delete only with approval.
- Remote access: check `getprop service.adb.tcp.port` and `settings get global adb_wifi_enabled` for ADB over the network, and look for remote-control apps (AnyDesk, TeamViewer, AirDroid, RustDesk, and "support" apps I don't recognise).
- Network interception: run `settings get global http_proxy`, check the `global_http_proxy_*` keys, and look for an always-on VPN and an unexpected Private DNS. User CA certificates can't be listed without root, so ask me to check Settings → Security → Encryption & credentials → User credentials.
- Firmware and counterfeit signs: check `ro.build.fingerprint` against the model and brand, and compare the real RAM and storage with what the box advertises. Look for preinstalled system apps with no clear OEM purpose that have internet, SMS or install permissions. Preinstalled malware families such as Triada and BadBox are mostly found on off-brand phones, tablets and TV boxes. Ask me to check Play Store → Settings → About → **Play Protect certification**. "Not certified" on a phone sold as a branded device is a warning sign.

**S-8. Preserve evidence** (if requested, or if anything Confirmed or Suspicious turns up). Save the following into `security_scan/evidence/` with timestamps:
- APKs of the flagged apps and their SHA-256 hashes;
- their `dumpsys package` output;
- `appops` output;
- the settings values above;
- `ps -A`;
- a `dumpsys` of device_policy, accessibility and notification;
- `adb bugreport security_scan/evidence/bugreport.zip` if I agree. It's large and contains personal data.

### Level 2: threat-indicator tools (ask before downloading anything)
- **Hash lookups.** Compute hashes on the phone with `adb shell sha256sum <apk path>` for every user app and flagged system app. With my VirusTotal API key, query `GET https://www.virustotal.com/api/v3/files/<sha256>`. The free tier allows about 4 lookups per minute, so query flagged apps first. Report detections as "X of Y engines". A few generic "adware" or "riskware" hits are common for ad-heavy apps; name-specific trojan or spyware detections from several engines are serious. Never upload an APK without my explicit yes; "not found" for a sideloaded app is itself worth noting.
- **spytrap-adb** (<https://github.com/spytrap-org/spytrap-adb>). Scans over ADB for stalkerware and suspicious settings, using the Echap stalkerware IOC list (<https://github.com/AssoEchap/stalkerware-indicators>). Follow its README to download the IOCs and run the scan. Report its findings, and re-check them with Level 1.
- **AndroidQF + MVT** (Amnesty International's Mobile Verification Toolkit). MVT no longer analyses devices directly over ADB. Collect with AndroidQF (<https://github.com/mvt-project/androidqf>), then analyse on the host:
  ```
  pip install mvt
  mvt-android download-iocs
  mvt-android check-androidqf --output security_scan/mvt <androidqf_output_folder>
  ```
  AndroidQF collects a lot of personal data (an optional backup, a bug report, logs and an app list). Keep it inside the device folder, and offer to delete it after the analysis unless it's needed as evidence. MVT can detect traces of known mercenary spyware and other threats, but only those covered by its indicator lists.
- **On-device scan.** Ask me to run Play Protect → Scan. Optionally, I can run one reputable on-device scanner (for example Malwarebytes or Bitdefender) and then uninstall it if I don't want to keep it.

### ⛔ Gate GS: scan results and remediation
Present the findings in three groups, each with app name, package, evidence and a plain explanation:
- **Confirmed:** an IOC match, a certificate mismatch on a disguised app, or multiple specific antivirus detections.
- **Suspicious:** strong heuristic combinations from S-2 to S-7.
- **Notable:** legitimate but powerful access worth reviewing. For example, a password manager with accessibility access is normal; say so.

End with "No signs found in these checks" if nothing turned up, plus the limits statement.

Then offer numbered remediation options. Apply only what I approve, in this order:

| # | Action | Notes |
|---|---|---|
| R1 | Preserve evidence (S-8) | Always first, if not already done |
| R2 | Revoke the flagged app's powerful access | Notification listener, accessibility, overlay, install, SMS/location/mic/camera permissions |
| R3 | Remove device admin rights | ADB can only remove test-only admins, so guide me through Settings → Security → Device admin apps |
| R4 | Uninstall the app | `pm uninstall <pkg>`. If it's disguised as a system app and refuses, use `pm disable-user --user 0 <pkg>` and say so |
| R5 | Remove interception | Clear the proxy (`settings put global http_proxy :0`), reset Private DNS to the previous value, remove an unknown VPN (guide me), and remove unknown user CA certificates (guide me) |
| R6 | Close remote access | Turn off ADB over the network; uninstall unknown remote-control apps |
| R7 | Secure accounts | From a different, clean device: change the Google, email, banking and messaging passwords; sign out of other sessions; turn on 2-step verification; check linked devices in WhatsApp and Telegram; check Find My Device and recovery details |
| R8 | Factory reset | When infection is likely and can't be removed cleanly. Back up personal data (not apps) first. |
| R9 | Reflash official firmware with the OEM tool, or ask the service centre | For suspected firmware or preinstalled malware. A factory reset does **not** remove malware built into the system partition. |

After remediation, re-run S-1 to S-7 for the affected items and show that they're gone. Save everything to `security_scan/report.md`.

## 4. Final report

Update `security_scan/report.md` and give me a short chat summary:
- device identity and security patch date;
- Confirmed, Suspicious and Notable counts, with one line per finding;
- what was remediated and how it was verified;
- what still needs my action, including steps only I can do on the phone or from another device;
- where the evidence and report are saved.

### ⛔ Gate G5: lock-down (very last step)
Use the end-state preference from G0. Ask me to confirm one of these:
- **Turn off USB debugging now:** `adb shell settings put global adb_enabled 0`. This ends the ADB session, so make sure all report and rollback files are already written.
- **I'll do it myself:** give me the menu path (Settings → Developer options → USB debugging, then turn off Developer options).
- **Leave it on:** note it in the report as a security reminder.

Also turn off wireless debugging (`adb_wifi_enabled`) if it is on, unless I use it.

## Appendix A: Shell pitfalls (learned the hard way)

**PowerShell host (Windows)**
- Don't name helper functions after built-in aliases: `del`, `rm`, `ls`, `cat`, `cp`, `mv`, `ps`, `kill`, `sleep`, `diff`, `sort`. The alias wins, and the call fails or does something else.
- A quoted Windows destination path that ends in `\` escapes the closing quote. Build paths with `Join-Path` and pass explicit file names.
- `$` inside double quotes is expanded by PowerShell before the string reaches the phone's shell. Use single quotes, or escape `$` with a backtick, when sending `sh` variables or `${...}` to `adb shell`.
- Native-command stderr appears as `NativeCommandError` even when the command worked. Check `$LASTEXITCODE` and the output text.
- Use `;` (Windows PowerShell 5) to chain commands, and pass `cwd` instead of `cd`.

**Device side (toybox `sh`)**
- Quote remote paths containing spaces in single quotes *inside* the `adb shell "..."` string: `adb shell "find '/sdcard/WhatsApp Video' -type f"`.
- Toybox `grep` rejects `\s`. Use `[[:space:]]` or a plain pattern.
- For large or Unicode output, write to `/data/local/tmp/<file>` on the phone, `adb pull` it, read it as UTF-8, then delete the remote file.
- Keep the agent's output small. Pipe through `wc -l`, `head` or summary counts instead of printing thousands of paths.

**ADB behaviour**
- `adb pull` of a whole tree stops at the first file it can't read. Copy per folder and reconcile.
- Long jobs (big copies, compilation) run as background processes with logs. Poll with bounded waits. Before restarting the ADB server, stop leftover `adb pull` clients and confirm what each one was copying.
- `dumpsys diskstats` and some `dumpsys` sections are cached or truncated, so cross-check with `du` and `pm list`.
- Several `adb.exe` copies may be on PATH (for example, scrcpy bundles its own). Use one consistently.

## Appendix B: OS-version notes

| Feature | Minimum Android | Fallback |
|---|---|---|
| `pm uninstall -k --user 0` | 5.0 | `pm disable-user --user 0` / `pm hide` (older) |
| `cmd package install-existing` | 8.0 | `pm install-existing` |
| `RUN_ANY_IN_BACKGROUND` appop | 9 | `RUN_IN_BACKGROUND` (8.x) |
| Private DNS | 9 | Skip; suggest a DNS app or router-level blocking |
| Auto-revoke unused permissions | 11 | Manual `pm revoke` |
| `MANAGE_EXTERNAL_STORAGE` | 11 | n/a |
| Shell read access to `Android/data` | Works on most builds through 16; some OEM builds block it | Skip those folders and say so |
| Restricted settings for sideloaded apps' accessibility | 13 | n/a |

When a command fails because the feature doesn't exist, report it as "not available on this device" and continue. Don't try workarounds that need root.
