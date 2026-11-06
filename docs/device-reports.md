# Device reports

Device reports are how this project gets better: they show which packages are safe to remove on which phones, which commands fail on which Android versions, and how well the spyware scan does in real life. Every report uses the same format, [`templates/device-report.md`](../templates/device-report.md), so they can be compared.

## Generate one

You don't have to fill it in by hand. At the end of a run, the agent asks whether you'd like an anonymous device report. Say yes and it:

1. writes `device-report.md` into the run's output folder, filled from that run only;
2. removes anything personal (serial, IMEI, phone number, accounts, Wi-Fi names, file names and the names of apps you installed);
3. shows it to you to check before anything is sent.

Ran an older version of a prompt, or a different agent? Ask it: *"Fill in templates/device-report.md from this run's report and logs, following its privacy rules."*

## Send it

**With the GitHub CLI** (`gh`, signed in): the agent can file it for you after you approve, with:

```bash
gh issue create --repo GeekLord/android-maintenance-agent --title "[Device] <Brand> <Model>, Android <version>" --body-file device-report.md
```

**In the browser:** open [a new device report](https://github.com/GeekLord/android-maintenance-agent/issues/new?template=device-report.yml), fill the short fields and paste the file into the report box. To copy the file first:

Windows (PowerShell):
```powershell
Get-Content device-report.md -Raw | Set-Clipboard
```

macOS:
```bash
pbcopy < device-report.md
```

Linux:
```bash
xclip -selection clipboard < device-report.md
```

Please read the report once before sending. Issues are public.
