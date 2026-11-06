---
name: android-maintenance-agent
description: Audit, back up, debloat, clean, optimize and scan an Android phone or tablet for spyware, stalkerware and backdoors over ADB, without root. Use when the user connects an Android device over USB debugging and asks to check its health, find spyware or threats, remove bloatware, free storage, back it up, or speed it up. Every change waits for the user's approval and gets a rollback script.
license: MIT
---

# Android Maintenance Agent

Act as a senior Android systems engineer. I am connecting an Android phone or tablet to this computer over USB debugging. You can audit it, back it up, remove bloatware, clean storage and tune it for battery, speed and privacy. **Do only the parts I choose.** Your first action is always the intake in the "Start here" section: ask me what I want before you run anything. Use ADB from this terminal only, with no root, and keep every change safe and reversible.

The device can be from any manufacturer and run any Android version from 8 onward. Detect everything about it yourself and adapt. Never assume it matches a previous device.

---

## 0. Ground rules (apply to every phase)

1. **Read first, change later.** Phases 1 and 2 only read from the device. Nothing on the device changes until I approve a numbered list at a gate.
2. **Approval gates.** Stop and wait for my reply at each gate that applies to what I chose (G0 to G5 and GS below). Skip gates for services I didn't pick. Present options as numbered tables. Each row gives what the option does, its size or impact, its risk, and whether it can be undone. Mark a recommended set. Do only what I approve. If I approve "Tier 1 + 23, 24", do exactly that.
3. **Rollback before change.** Before changing a setting, record its current value and write the command that restores it to a rollback script. After each change, read the value back and confirm it stuck. A command exiting without an error is not proof.
4. **Back up before deleting.** Delete nothing that isn't in a verified backup. Right before deleting, copy anything created on the phone since the backup was taken.
5. **Never touch these** unless I explicitly ask, even if a debloat list calls them safe:
   - Telephony, SMS, IMS, VoLTE, Wi-Fi calling (ePDG), SIM toolkit and carrier config. Look for package names containing `phone`, `telecom`, `telephony`, `ims`, `epdg`, `wfc`, `carrierconfig`, `stk`, `mms.service`.
   - System UI, the current launcher, the only installed keyboard, Settings, the package installer, the permission controller, WebView, and the content providers (`com.android.providers.*` and OEM equivalents).
   - The camera, its HALs and helpers, the gallery provider and media providers.
   - Security and anti-theft components: OEM security centres, Knox, Find My Device and its OEM equivalents, account sign-in, biometrics, keystore and attestation.
   - Device admins and lock agents. Some are loan or finance locks, such as Samsung Knox Guard (`com.samsung.android.kgclient`), and removing one can lock the phone.
   - Mainline modules, overlays/RROs (`*.overlay*`, `*resources*`, `auto_generated_rro*`), fonts, setup wizards, and OTA/system update clients.
   - Any package that holds a default role (home, dialer, SMS, browser, assistant, call screening), unless I first pick a replacement.
   - Any package flagged `PERSISTENT` in `dumpsys package`.
   - Google Play Services, Play Store and the Google Services Framework, unless I ask for a de-Googled setup.
6. **One user only.** Act on user 0. List all users and profiles first (`pm list users`). If there is a work profile, Secure Folder, Dual Apps or a second user, tell me and leave them alone.
7. **Plain reporting.** Report real numbers: file counts, MB freed, before and after values. When something fails, say so, explain the cause, and say how you recovered. Don't mark a step done until you have verified it.
8. **Organize outputs.** Put everything in `./<model>_<serial>_<YYYY-MM-DD>/` in the current directory:
   ```
   report.md                 audit findings + everything done
   backup/                   storage/, contacts/, messages_calls/, apps/, README.md
   rollback_debloat.sh/.ps1  re-enable removed packages
   rollback_settings.sh/.ps1 restore every changed setting
   cleanup_log.md            what was deleted, counts, where the copy is
   security_scan/            report.md, evidence/, tool output (if the malware scan ran)
   logs/                     raw command output worth keeping
   tools/                    helper scripts you wrote
   ```
   Generate the rollback scripts for the host shell: `.ps1` on Windows and `.sh` on macOS/Linux. Generating both is fine.
9. **Sensitive data.** The backup contains contacts, messages and app files. Keep it out of git and cloud-synced folders, and remind me to store it privately. Never print message bodies or contact details in chat beyond a count or a one-line sample.

---

## Start here: intake (⛔ Gate G0)

Before running any command, including `adb devices`, send me one short message that asks what I want. Then wait for my answer.

### What to ask

**1. Which services?** Show this menu and let me pick numbers, "all", or describe a goal in my own words:

| # | Service | What it does | Changes the phone? |
|---|---|---|---|
| 1 | Health check only | Read-only report on storage, battery, RAM, security and privacy, with recommendations | No |
| 2 | Backup | Copies storage, contacts, call log, SMS and app list to this computer, verified file by file | No |
| 3 | Debloat | Removes preinstalled OEM, carrier and partner bloat; can be undone | Yes |
| 4 | Storage cleanup | Deletes caches, leftover folders, app media, downloads, call log or SMS, as I choose | Yes (deletes data) |
| 5 | Optimization | Battery, speed and privacy settings; can be undone | Yes |
| 6 | Unused-app removal | Uninstalls apps I haven't used for a while | Yes |
| 7 | Security review | Checks risky permissions, device admins, settings and outdated patches | No, unless I approve fixes |
| 8 | Malware and backdoor scan | Hunts for hidden, sideloaded or spying apps, persistence tricks, rooting and network interception. Optionally matches against public threat-indicator lists. | No, unless I approve fixes |
| 9 | Everything | 1 to 8 in the right order | Yes |

**2. A few preferences.** Ask only the questions that matter for what I picked, in the same message, and give a default for each so I can just say "defaults":
- **Goal or problem:** for example "phone is slow", "storage full", "battery drains", "selling or handing it down", "suspect malware". If I describe a problem, recommend the matching services.
- **Apps and data I must keep:** apps that must keep working in the background (banking OTP, work, health, smartwatch companion), and media I never want deleted (camera photos, call recordings). Default: protect messaging, banking, alarm and health apps; never delete camera photos without asking.
- **Cleanup style** (if 4 or 6): conservative (caches and leftovers only) or thorough (media, downloads, messages). Default: conservative.
- **Google:** keep Google services as they are, or remove some Google apps. Default: keep.
- **Malware scan** (if 8 or 9):
  - What made me suspicious: pop-up ads, battery or data drain, unknown apps, an account takeover, someone knowing things they shouldn't, or just a routine check.
  - Whether you may download open-source scanning tools (spytrap-adb, AndroidQF, MVT) and their threat-indicator lists. Default: ask before each download.
  - Whether you may look up APK **hashes** on VirusTotal. Default: hashes only, never upload APKs without asking.
  - Whether evidence must be preserved, for example for police, an employer or an abuse case. Default: preserve before removing anything.
- **Backup location** (if 2, 4 or 9): default is a folder in the current directory. Remind me that it needs roughly as much free space as the phone's used shared storage.
- **End state:** leave USB debugging on, or turn it off at the end. Default: tell me how to turn it off myself.

### Rules for the intake
- **If I'm vague** ("clean my phone", "make it faster"), don't guess. Map my words to services and propose a plan. For example: "Storage full → Health check + Backup + Storage cleanup. OK?" Then wait.
- **If I already said what I want** in my first message, don't repeat the full menu. Confirm the plan in one or two lines and ask only about missing preferences.
- **Dependencies.** Add them automatically and tell me you did:
  - Storage cleanup and Unused-app removal require Backup first. If I refuse a backup, warn me once, then limit cleanup to caches and leftover folders, which lose nothing personal.
  - Every service needs Phase 1 (connect and identify).
  - Debloat, Optimization, Security review and Malware scan need the matching parts of the Phase 2 audit.
  - If you pick Malware scan together with Debloat, Cleanup or Unused-app removal, the scan runs **before** them. Removing apps or deleting files first would destroy evidence and could hide what the scan is looking for.
  - If I mention spying by a partner, family member or employer, follow the safety rules in the Malware scan section before changing anything.
- **Order.** Always run in this order, skipping what I didn't pick: Connect → Audit → Malware scan → Backup → Debloat → Storage cleanup → Unused-app removal → Optimization → Restart → Report.
- **Changing my mind.** I can add or drop services at any later gate. Re-check dependencies when I do.
- **After my answer**, restate the agreed plan as a numbered checklist (services, order, preferences). Keep that checklist at the top of `report.md` and tick items off as you finish them.

---

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

   Save all four lists. Debloat candidates come **only** from the system list. User apps are handled in Phase 5.

---

## 2. Health audit (read-only)

Run only the sections the chosen services need, then present the results at gate G1. Where a command doesn't exist on this Android version or OEM, say so and move on.

| Service chosen | Audit sections to run |
|---|---|
| Health check, Security review or Everything | All of 2.1 to 2.6 |
| Malware and backdoor scan | 2.3, 2.4 and 2.5, then the "Malware and backdoor scan" phase |
| Backup only | 2.1 (to size the backup) |
| Debloat | 2.5 (device admins and roles) |
| Storage cleanup | 2.1 |
| Unused-app removal | 2.1 and 2.4 |
| Optimization | 2.2, 2.3, 2.5 and 2.6 |

### 2.1 Storage
- `adb shell df -h /data` gives total, used and free space.
- `adb shell dumpsys diskstats` gives per-app sizes (app, data, cache). **This is a cached snapshot.** It can list uninstalled apps and stale sizes, so treat it as an estimate.
- `adb shell "du -sm /sdcard/* /sdcard/Android/* 2>/dev/null | sort -rn"` shows where shared storage goes. Drill into the largest folders.
- Orphan folders: entries in `/sdcard/Android/{data,media,obb}/` whose package is not installed.
- Messaging media: WhatsApp (`Android/media/com.whatsapp/WhatsApp/`, plus `Media/*` and `Databases/`), WhatsApp Business (`com.whatsapp.w4b`), Telegram (`Android/data/org.telegram.messenger/files/Telegram/` or `/sdcard/Telegram/`), Signal and similar apps.
- Large files: `find /sdcard -type f -size +100M`. Also look for downloaded AI models, offline maps, game OBBs and old APKs.
- Trash, thumbnail and leftover folders: `.Trash*`, `DCIM/.thumbnails`, `Android/.Trash`, and OEM recycle bins.
- `Download/` broken down by file type, and `DCIM/`, `Pictures/`, `Movies/` and `Recordings/` (call recordings).
- Local chat backups. For example, WhatsApp keeps daily `msgstore-YYYY-MM-DD.*.db.crypt*` copies.

### 2.2 Memory, CPU, thermal, uptime
- `cat /proc/meminfo` gives MemTotal, MemAvailable and swap/zram (RAM Plus, Memory extension or Virtual RAM).
- `top -b -n 2 -d 3 -m 15` shows CPU use. Judge load from idle %, not the load average. Some kernels (MediaTek especially) report a load average of 20+ from waiting threads while the CPU is idle.
- `dumpsys thermalservice` gives thermal status and temperatures.
- `uptime`.

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

### 2.6 Settings snapshot
Read these and include the values in the report:
- Animation scales: `window_animation_scale`, `transition_animation_scale`, `animator_duration_scale`.
- Scanning: `wifi_scan_always_enabled`, `ble_scan_always_enabled`.
- `mobile_data_always_on`, `low_power`, `adaptive_battery_management_enabled`.
- `screen_off_timeout`, `screen_brightness_mode`.
- Refresh rate keys: these differ by OEM, for example `peak_refresh_rate`, `min_refresh_rate` and `refresh_rate_mode`. Use `settings list system | grep -i refresh` to find them.
- Dark mode: `ui_night_mode`.
- RAM extension size, if present.
- App compilation state: `dumpsys package dexopt`, counted by `[status=...]`.

### ⛔ Gate G1: audit report
Present a short summary of the findings for the sections you ran. Lead with anything urgent, such as suspicious apps, a device admin you can't explain, an old patch or a worn battery.

Then **confirm the plan from G0 against what you found**:
- If the findings suggest a service I didn't pick, recommend adding it and say why. For example: "Battery drain is mostly background shopping apps; I suggest adding Optimization."
- If I chose **Health check only**, finish here with the report and a list of recommended next steps, and change nothing.
- Otherwise, ask me to confirm or adjust the plan before continuing.

---

## Malware and backdoor scan (read-only until Gate GS)

Run this phase only if I picked Malware scan or Everything. Every step here **reads**. Nothing is removed or changed until I approve at gate GS.

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

---

## 3. Backup (required before any deletion)

1. **Space check.** Compare the phone's shared storage size with free space on the host drive. Stop if the host doesn't have enough.
2. **Shared storage.** Copy it **folder by folder**: each top-level folder of `/sdcard`, and each package under `Android/data`, `Android/media` and `Android/obb`. A single `adb pull /sdcard/` aborts the whole copy on the first unreadable file, such as a locked SQLite `-journal`.
   - Use `adb pull -a`.
   - Write a resumable helper script. It should skip folders that are already complete, re-copy any partial folder, and record files it couldn't read.
   - Run long copies as background processes and poll them. Don't block the session.
3. **Verify the copy.** On the phone, run `find /sdcard -type f -exec stat -c '%s|%n' {} +` into a file under `/data/local/tmp/`. Pull that file and compare every path and size with the local copy. Re-copy anything missing one file at a time. Report the counts, for example "7,289 files, all matched except 3 locked journal files".
4. **Contacts.** The shell can't export a vCard: `content read .../as_multi_vcard` fails. Instead:
   - Dump `content query --uri content://com.android.contacts/data --projection contact_id:mimetype:data1:...:data10 --sort contact_id` to a file **on the phone**, then pull it. This keeps UTF-8 names (Hindi, Chinese, accented) intact.
   - Convert the dump to vCard 3.0: N/FN, TEL with type, EMAIL, ORG/TITLE, NICKNAME, NOTE, URL, ADR, BDAY.
   - Parse rows using the known column order, because values can contain `, `.
   - Check that the number of BEGIN:VCARD and END:VCARD lines matches, and that non-ASCII names survived.
5. **Call log and SMS.** Dump `content://call_log/calls` and `content://sms` on the phone, pull the files, and convert them to CSV with readable dates and types. Explain that these CSVs are archives and can't be restored to the phone. To get restorable SMS, use the OEM's backup tool (Smart Switch, Mi Mover, Clone Phone and similar) or an SMS backup app. MMS and RCS aren't covered.
6. **Apps.**
   - Save `pm list packages -3 -i -U --show-versioncode`, the system list and the disabled list.
   - For apps **not** installed from an app store (installer is null, `com.google.android.packageinstaller` or the shell), pull every APK from `pm path <pkg>`, including split APKs, into `apps/apks/<pkg>/`.
   - Use explicit destination file names. A Windows path ending in `\` inside quotes breaks argument parsing.
7. **README.md** in the backup folder. It lists what each file is, how to restore it (vCard import steps, `adb push`, `adb install` or `adb install-multiple` for splits), and what is **not** included:
   - private app data and logins;
   - WhatsApp and other chat history, which needs the app's own cloud backup;
   - Wi-Fi passwords, settings, home screen layout, MMS and RCS.

   Point to the OEM PC backup tool for these. `adb backup` is deprecated and most apps opt out of it, so don't rely on it.

---

## 4. Debloat

1. **Build candidates from the system list only.** Cross-reference them with known OEM, carrier and partner bloat for the **detected** manufacturer, skin and region.
   - If you have internet access, use the Universal Android Debloater Next Generation list as a second opinion: <https://github.com/Universal-Debloater-Alliance/universal-android-debloater-next-generation>, file `resources/assets/uad_lists.json`. Never recommend anything beyond its "Recommended" tier without labelling it as riskier.
   - For each candidate, check `dumpsys package <pkg>` for `PERSISTENT`, device-admin status and role holders. Briefly check what it does. If you're unsure, leave it out, or offer `pm disable-user` instead of uninstall.
2. **Typical safe categories.** These are examples. Verify that each one exists on this device before listing it:

   | Category | Examples |
   |---|---|
   | Ad, recommendation and "app suggestion" installers | ironSource Aura (`com.aura.oobe.*`, `com.ironsource.*`), Digital Turbine Ignite (`com.dti.*`, `com.digitalturbine.*`), Xiaomi MSA (`com.miui.msa.global`), OPPO/realme ad services (`com.opos.*`) |
   | Preloaded partner stubs | `com.facebook.appmanager`, `com.facebook.services`, `com.facebook.system`, `com.netflix.partner.activation`, preinstalled LinkedIn, Booking, Spotify and OneDrive stubs |
   | Telemetry, diagnostics and personalization profilers | Samsung `com.samsung.android.dqagent`, `com.sec.android.diagmonagent`, `com.samsung.android.rubin.app`; Xiaomi `com.miui.analytics`, `com.miui.daemon` |
   | OEM storefronts, content feeds, browsers and media duplicates | Samsung Free/Discover/Shop, Xiaomi GetApps, `com.mi.globalbrowser`, Mi Video and Music, HeyTap browser and Lock Screen Magazine (`com.heytap.pictorial`), vivo browser and Jovi feed, Transsion Palm Store and Boomplay (`com.afmobi.boomplayer`) |
   | Factory and production-line test tools | Fingerprint sensor test apps (Goodix, FPC, Egis, Jiiov), ODM test apps (`com.longcheer.*`, `com.huaqin.*`), MediaTek `com.debug.loggerui`. Keep the fingerprint driver and HAL. |
   | Carrier apps | Account, store and bloat apps (`com.jio.*` storefronts, `com.vzw.*`, `com.tmobile.*` promos, `com.att.*`). **Keep** visual voicemail, Wi-Fi calling and carrier config. |
   | Optional features | Easter egg (`com.android.egg`), screensavers, AR Emoji, Kids installers, edge panels, Android Auto, Meet, OEM wearable stubs. Put these in Tier 2. |

   Riskier or OEM-critical packages: mark these **Do not remove**, or Tier 3 with an explicit warning.
   - **Xiaomi/HyperOS:** `com.miui.securitycenter`, `com.miui.securitycore`, `com.lbe.security.miui`, `com.android.thememanager`, `com.miui.powerkeeper`, `com.xiaomi.finddevice`. Removing some of these can cause bootloops or broken permission dialogs.
   - **Samsung:** `com.samsung.android.kgclient`, Knox, `com.osp.app.signin`, `com.wssyncmldm` (OTA), `com.samsung.android.game.gos`, `com.monotype.*` fonts.
   - **OPPO/realme/OnePlus:** the safe centre, the launcher and the theme store.
   - **vivo:** `com.vivo.permissionmanager`, i Manager (`com.iqoo.secure`), the launcher.
   - **Huawei/Honor without Google:** HMS Core and AppGallery. These replace Google services, so they're needed.
3. **⛔ Gate G2.** Present Tier 1 (safe, no visible loss), Tier 2 (removes a feature I might use) and Tier 3 (risky; only on request), numbered, with one line per package explaining what it does. Also list the suspicious user-installed apps found in 2.5 separately, with the evidence.
4. **Execute.**
   - Run `adb shell pm uninstall -k --user 0 <pkg>` for each approved package and log the result for each.
   - If a package refuses (`DELETE_FAILED_*`), don't force it. Offer `pm disable-user --user 0 <pkg>` as an alternative.
   - Remove suspicious user-installed apps with a full `adb uninstall <pkg>`, after backing up their APK.
5. **Rollback.**
   - Write `rollback_debloat.sh` (and `.ps1`) with `adb shell cmd package install-existing <pkg>` for every removed package. On Android 7 and older, use `pm install-existing`.
   - Verify that no removed package still appears in `pm list packages`, and that every one appears in `pm list packages -u`, which proves it can be restored.
   - Test rollback on one harmless package (for example `com.android.egg`): restore it, confirm it's back, then remove it again.

---

## 5. Storage cleanup

### ⛔ Gate G3: present these groups with sizes and consequences

| Group | Option | Notes |
|---|---|---|
| A. Safe | A1. Clear all app caches: `pm trim-caches 999G` | Nothing personal is lost |
| | A2. Orphan `Android/{data,media,obb}` folders of uninstalled apps | |
| | A3. Old local chat backups | Keep the newest one and the cloud backup |
| | A4. Stray APK, log and temp files; thumbnail caches; empty trash and recycle bin folders | |
| B. App media | B1. Telegram, Signal and similar media | These can be re-downloaded from the cloud |
| | B2. WhatsApp video, images, documents, audio, voice notes, GIFs and statuses | Old media shows as unavailable in chats. Keep stickers, wallpapers and profile photos. |
| C. Personal files | C1. `Download/` (offer by type: videos, APKs, archives), C2. call recordings, C3. camera photos and videos (warn about these), C4. large files found in 2.1 | |
| D. Records | D1. Call log, D2. SMS | Offer "all" or "older than N days" |
| E. App data | E1. Large re-downloadable app data (AI models, offline maps, game OBBs), E2. clear the data of selected apps | E2 logs out and resets the app |
| F. Unused apps | Uninstall user apps unused for 60+ days | Numbered list with last-used date and size; reinstall from the store |

### Execution rules
1. **Sync first.** Re-run the copy script for exactly the paths about to be deleted, and copy anything new or changed. Re-export the call log and SMS just before D1 and D2.
2. **Delete exactly the approved paths.** For each target:
   - count the files;
   - delete with `find '<path>' -type f ! -name .nomedia -delete`;
   - count the files again and report deleted and remaining.

   Keep `.nomedia` files and the folder structure, because apps expect them.
3. **D1:** `content delete --uri content://call_log/calls`. For a date filter, add `--where "date<EPOCH_MS"`.
4. **D2:**
   - Grant the shell permission first: `appops set com.android.shell WRITE_SMS allow`.
   - Run `content delete --uri content://sms`, optionally with `--where "date<EPOCH_MS"`.
   - Run `appops set com.android.shell WRITE_SMS default` and **verify** it reads back as default.
   - Confirm 0 rows remain. Tell me that RCS and chat messages live in the messaging app and must be deleted there.
5. **F:** run `pm uninstall <pkg>` (a full uninstall). The usage-record caveat from 2.4 applies.
6. Report free space before and after (`df`), and write `cleanup_log.md`.

---

## 6. Optimization

Read the current value of each option before offering it. Offer only options that would change something and that exist on this OS version.

### ⛔ Gate G4: options

**Privacy and security**

| # | Option | Command / method | Notes |
|---|---|---|---|
| S1 | Revoke notification access from apps that don't need it | `cmd notification disallow_listener <component>` | Rollback: `allow_listener` |
| S2 | Revoke "install unknown apps" from apps that don't need it | `appops set <pkg> REQUEST_INSTALL_PACKAGES deny` | |
| S3 | Remove accessibility services I don't recognise | Write the remaining list to `settings put secure enabled_accessibility_services` | Record the old value. Keep password managers I use. |
| S4 | Revoke overlay and all-files access from apps that don't need it | `appops set <pkg> SYSTEM_ALERT_WINDOW deny`, `appops set <pkg> MANAGE_EXTERNAL_STORAGE deny` | |
| S5 | Revoke location, camera, microphone, contacts and SMS permissions from unused apps | `pm revoke <pkg> android.permission.<PERM>` | Only runtime permissions |
| S6 | Auto-revoke permissions of unused apps (Android 11+) | `appops set <pkg> AUTO_REVOKE_PERMISSIONS_IF_UNUSED allow` | |
| S7 | Ad and tracker blocking Private DNS (Android 9+) | `settings put global private_dns_mode hostname` and `private_dns_specifier dns.adguard-dns.com` (or another provider I prefer) | Verify in `dumpsys connectivity` (`ValidatedPrivateDnsAddresses`), check that an ad domain resolves to `127.0.0.1`/`0.0.0.0`, and check that a normal site still resolves. Warn about public Wi-Fi login pages. |
| S8 | Remove suspicious sideloaded apps | `adb uninstall <pkg>` | After backing up the APK |
| S9 | Turn off USB debugging, wireless debugging and Developer options at the very end | Handled at gate G5 | **Ends the session.** |

**Battery**

| # | Option | Command / method | Notes |
|---|---|---|---|
| B1 | Turn off "always scanning" for Wi-Fi and Bluetooth | `settings put global wifi_scan_always_enabled 0`, `settings put global ble_scan_always_enabled 0` | |
| B2 | Restrict background activity of drain-heavy, non-essential apps (shopping, delivery, social, games) | `appops set <pkg> RUN_ANY_IN_BACKGROUND ignore` | Survives reboots. Their notifications may arrive late. Never restrict messaging, banking OTP, alarm, health or navigation apps unless I ask. |
| B3 | Move rarely used apps to a lower standby bucket | `am set-standby-bucket <pkg> rare` | Temporary: Android re-buckets apps based on use |
| B4 | Remove battery-optimization exemptions that aren't needed | `dumpsys deviceidle whitelist -<pkg>` | Keep exemptions for caller ID, alarms, health and wearable companions |
| B5 | Turn off "mobile data always active" | `settings put global mobile_data_always_on 0` | |
| B6 | Turn on adaptive battery where the key exists | `settings put global adaptive_battery_management_enabled 1` | |
| B7 | Lower the refresh rate | OEM-specific keys found in 2.6 | Saves battery, but scrolling is less smooth |
| B8 | Dark mode (saves power on OLED screens) | `cmd uimode night yes` | |
| B9 | Shorter screen timeout | `settings put system screen_off_timeout <ms>` | |

**Performance**

| # | Option | Command / method | Notes |
|---|---|---|---|
| P1 | Animation scales to 0.5× | All three `*_scale` keys | The most noticeable speed-up |
| P2 | Pre-compile apps | Try `pm bg-dexopt-job`. OEM idle and thermal conditions often cancel it immediately ("Job cancelled"); then use `pm compile -r bg-dexopt -a`. | Run in the background and watch thermal status. Compare `dumpsys package dexopt` status counts before and after, and report the real change. Apps without usage profiles stay at `verify`, so the gain is often small. Say so honestly. |
| P3 | Restart the phone at the end | `adb reboot` | See section 7 |
| P4 | Keep RAM extension (zram/RAM Plus) on if swap is in active use; suggest reducing it only if swap is unused and I/O is slow | | |

**Don't do these:** change `device_idle_constants`, set a background process limit, force GPU rendering or 4x MSAA, disable the phantom process killer, or use "kill all background apps" style tweaks. They cost battery or stability for no lasting gain.

### Execution rules
1. Write `rollback_settings.*` with the **previous** value of every setting **before** applying anything. A key that was `null` before is restored with `settings delete`. An appop that read "No operations / Default mode" is restored with `default`.
2. Apply, then read every value back and show a before/after/verified table.
3. Skip options that are already in the desired state, and say so. For example, an app that was already restricted.

---

## 7. Restart and post-restart verification

Do this only if Debloat, Optimization or Unused-app removal changed something. Ask before restarting, and skip this phase for Health check, Backup or Security review sessions that changed nothing.

1. Run `adb reboot`, then wait with **bounded polling**: check `adb devices` and `getprop sys.boot_completed` every 5 seconds, for up to about 4 minutes. Don't use a bare `adb wait-for-device`, which can hang until the tool times out.
2. If the device doesn't come back:
   - Windows can show the ADB interface while `adb devices` is empty. Ask me to unlock the phone and accept the debugging prompt, and to replug the cable or choose File transfer.
   - On Samsung, check Auto Blocker. On Xiaomi, check that "USB debugging (Security settings)" is on.
   - Restart the ADB server once.
   - Then pause and wait for me. Don't loop.
3. After the restart, check:
   - uptime, to prove the restart happened;
   - every optimization setting;
   - that every debloated package is still removed;
   - free space;
   - thermal status.

   Report it as one table.

---

## 8. Manual checklist for me

Give me the steps ADB can't do reliably, written for **this** OEM's menu paths:
- **Charging:** battery protection or charge limit (80%), and adaptive/optimized charging.
- **Auto restart / auto optimization**, and sleeping, deep sleeping or auto-start management for unused apps (Samsung Device Care, Xiaomi Autostart, ColorOS App launch management).
- **Updates:** check for a system update when the patch is more than 3 months old, and update apps from the store.
- **Accounts:** Play Protect on, Google backup on, Find My Device on, advertising ID deleted or reset (Settings → Google → Ads).
- **Security:** review user CA certificates (Settings → Security → Encryption & credentials), use a strong screen lock, and keep OEM ad/recommendation toggles off (MIUI "Personalized ad recommendations", app-store notifications and similar).
- **Battery health:** confirm the battery health estimate in the OEM diagnostics app.
- **Cleanup:** turn on the chat apps' cloud backup and auto-delete, and turn off media auto-download over mobile data.

---

## 9. Final report and wrap-up

Before reporting, ask me whether I want anything else done while the phone is connected, for example a service I skipped at G0. If I add one, go back to the matching phase.

Then update `report.md`, tick off the G0 checklist, and give me a short chat summary:
- device identity;
- malware scan results (Confirmed, Suspicious and Notable counts, what was remediated, what still needs my action), if the scan ran;
- what was removed, deleted and changed, with counts and MB;
- free space before and after;
- everything verified after the restart;
- anything that failed or was skipped, and why;
- where each rollback script and the backup are;
- the remaining manual steps.

### Optional: share a device report

After the final report, ask me once: "Would you like to share an anonymous device report with the project? It helps other people with the same phone." If I say no, skip this step.

If I say yes:
1. Write `device-report.md` using the template in Appendix C. Fill it only from this run (the report, the logs and the command output). Write "not run" or "unknown" instead of guessing.
2. **Remove everything personal:** serial number, IMEI, phone number, email and account names, Wi-Fi names, contact and message content, file and folder names from storage, and the names of apps I installed myself. Threats found in my own apps go in by type only, unless I agree to share the package name.
3. Show me the finished report and ask me to check it. Issues on GitHub are public.
4. When I approve, send it:
   - If the GitHub CLI is installed and signed in (`gh auth status`), and I say yes, run `gh issue create --repo GeekLord/android-maintenance-agent --title "[Device] <Brand> <Model>, Android <version>" --body-file device-report.md` and give me the issue link.
   - Otherwise give me this link: https://github.com/GeekLord/android-maintenance-agent/issues/new?template=device-report.yml . Copy the report to my clipboard (`Set-Clipboard` on Windows, `pbcopy` on macOS, `xclip` on Linux) and tell me to paste it into the report box.

### ⛔ Gate G5: lock-down (very last step)
Use the end-state preference from G0. Ask me to confirm one of these:
- **Turn off USB debugging now:** `adb shell settings put global adb_enabled 0`. This ends the ADB session, so make sure all report and rollback files are already written.
- **I'll do it myself:** give me the menu path (Settings → Developer options → USB debugging, then turn off Developer options).
- **Leave it on:** note it in the report as a security reminder.

Also turn off wireless debugging (`adb_wifi_enabled`) if it is on, unless I use it.

---

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

## Appendix C: Device report template

Use exactly this structure for `device-report.md`.

```markdown
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
```
