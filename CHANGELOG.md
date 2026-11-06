# Changelog

## 1.1.0 (2026-11-06)

- Standard device report: `templates/device-report.md` (also Appendix C in `SKILL.md` and `prompts/security-scan.md`).
- All three prompts now offer, at the end of a run, to write an anonymous device report, strip personal data, show it for review and file it with `gh issue create` or the issue form.
- Issue form rebuilt around the report: device fields, services, overall result, a report box and a required privacy check.
- New guide: `docs/device-reports.md`.

## 1.0.0 (2026-11-06)

- First public release.
- `SKILL.md`: full maintenance agent (intake, health audit, spyware and backdoor scan, backup, debloat, storage cleanup, unused-app removal, optimization, restart checks, report, lock-down) with approval gates and rollback scripts.
- `prompts/security-scan.md`: standalone spyware and threat scan.
- `prompts/quick-debloat.md`: short debloat-only prompt with tiered approval and a rollback script.
