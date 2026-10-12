# odc-build-playbook

A Claude Code skill for building, publishing and deploying OutSystems Developer Cloud (ODC) apps through the remote `outsystems` MCP (Mentor, `publish_status`, `env_app`).

## What it does
- Plans the build into small, dependency-ordered chunks (data, then logic, then screens, then navigation) instead of one giant Mentor prompt.
- Avoids known failures: missing `portfolioKey`, the deprecated `mentor_publish`, wrong `app_info` argument name.
- Publishes via `mentor_prompt "Publish"`, then `publish_status`, then `env_app` for the runtime URL, and never publishes the same app concurrently.
- Asks before any tenant-changing action and reports results honestly.

## Test result
Same Task Tracker spec built twice through Mentor:

| | Without skill | With skill |
|---|---|---|
| Mentor prompts | 2 (first returned only a plan) | 3 chunked, no retries |
| Build time | ~35 min | ~10 min |
| Validation | 0 errors, 0 warnings | 0 errors, 2 warnings |

One run per condition, so treat the numbers as indicative.

## Install
Copy `SKILL.md` to `~/.claude/skills/odc-build-playbook/SKILL.md` (Windows: `%USERPROFILE%\.claude\skills\odc-build-playbook\SKILL.md`), then restart Claude Code. Requires the `outsystems` MCP to be connected and authorized.
