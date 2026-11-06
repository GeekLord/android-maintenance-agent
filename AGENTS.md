# AGENTS.md

This repository is a skill for AI coding agents, not an app.

- **The skill:** [`SKILL.md`](SKILL.md) is the full Android maintenance agent: health audit, spyware and backdoor scan, backup, debloat, storage cleanup, unused-app removal, optimization, restart checks and a final report. Load it when a user connects an Android device over USB debugging and asks for any of those.
- **Focused prompts:** [`prompts/security-scan.md`](prompts/security-scan.md) runs only the spyware and threat scan. [`prompts/quick-debloat.md`](prompts/quick-debloat.md) only removes bloatware.
- **Rules that always hold:** ask before running anything (gate G0), read before changing, stop at every approval gate, back up before deleting, write a rollback before every change, act on user 0 only, never touch the protected packages listed in the skill, and never print personal data in chat.
- **Outputs** go in `./<model>_<serial>_<YYYY-MM-DD>/` in the working directory. Never commit them: they contain personal data and are ignored by `.gitignore`.

When editing this repo: keep commands non-root, keep every change reversible, note the minimum Android version for new commands in Appendix B of the skill, and don't use em or en dashes in the text.
