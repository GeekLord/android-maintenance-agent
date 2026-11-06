# Quick Debloat Prompt (no root)

A short prompt that removes preinstalled bloatware over ADB and writes a rollback script. Use it when you only want to debloat. For a health check, backup, cleanup, optimization or a spyware scan, use [`SKILL.md`](../SKILL.md) or [`security-scan.md`](security-scan.md).

Copy everything below the line into your agent with the phone connected over USB debugging.

---

Act as an Android systems engineer. I am connecting an Android device via USB debugging. Your objective is to safely debloat it using the terminal, without root access.

Follow this protocol step by step:

1. Run `adb devices -l` to verify the connection. If it shows `unauthorized`, ask me to tap **Allow** on the phone. If more than one device is listed, ask which one and add `-s <serial>` to every command.
2. Identify the device: `adb shell getprop ro.product.manufacturer`, `ro.product.model`, `ro.build.version.release` and the OEM skin. Run `adb shell pm list users` and act on user 0 only.
3. List preinstalled system packages with `adb shell pm list packages -s`. Debloat candidates come only from this list, never from apps I installed.
4. Cross-reference them with known safe-to-remove OEM, carrier and partner bloatware for the detected manufacturer, skin and region.
5. Present the candidates as a numbered table in three tiers: Tier 1 (safe, no visible loss), Tier 2 (removes a feature I might use) and Tier 3 (risky, only if I ask). Give one line per package explaining what it does. Then stop and wait for my approval.
6. For each package I approve, run `adb shell pm uninstall -k --user 0 <package_name>`, then confirm it is gone with `adb shell pm list packages <package_name>`.
7. Create a rollback script in the current directory with `adb shell cmd package install-existing <package_name>` for every removed package: `rollback.sh` on macOS or Linux, `rollback.ps1` on Windows.
8. Report what was removed, what failed and why, and where the rollback script is.
9. Ask me if I want to share an anonymous device report. If yes, fill in the template from https://raw.githubusercontent.com/GeekLord/android-maintenance-agent/main/templates/device-report.md using only this run's results, remove the serial, IMEI, phone number, account names and the names of apps I installed, show it to me, and after I approve either file it with `gh issue create --repo GeekLord/android-maintenance-agent --body-file device-report.md` or give me https://github.com/GeekLord/android-maintenance-agent/issues/new?template=device-report.yml to paste it.

Never remove telephony, SMS, IMS or carrier config, System UI, the current launcher or keyboard, Settings, the package installer, WebView, the camera, Google Play Services or Play Store, device admins or finance locks (such as Samsung Knox Guard), mainline modules, overlays, setup wizards or system update clients. Do not uninstall anything before I approve at step 5.
