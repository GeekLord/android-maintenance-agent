# Contributing

Thanks for helping. The most useful contributions are:

1. **Device reports.** Run a prompt and [open a device report](../../issues/new?template=device-report.yml): model, Android version and skin, which agent and model you used, which services you ran, and what worked or failed. Remove serial numbers, phone numbers and anything personal first.
2. **Bloat list fixes.** A package that was wrongly recommended (or missed) for your manufacturer or region. Say what broke and how you restored it.
3. **OS and OEM quirks.** A command that fails on a specific Android version or skin, and the working alternative. These go into Appendix B of `SKILL.md`.
4. **New threat checks.** Heuristics or indicator sources for spyware and stalkerware that work over ADB without root.

## Rules for changes

- Keep everything rootless and reversible. Every change the agent makes needs a recorded rollback.
- Never weaken an approval gate or the protected-packages list.
- Note the minimum Android version for any new command.
- Plain English, short sentences, no em or en dashes.
- Never commit device output folders, backups, APKs or bug reports.
