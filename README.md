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

## Requirements
- The remote `outsystems` MCP is connected and authorized.
- A portfolio key for your tenant (the skill explains how to find one).

## Install

### Claude Code (terminal, or the Code tab in the desktop app)

**Personal skill, available in all projects**

macOS / Linux / Git Bash:
```bash
git clone https://github.com/rohitsingh-sg/odc-build-playbook.git
mkdir -p ~/.claude/skills/odc-build-playbook
cp odc-build-playbook/SKILL.md ~/.claude/skills/odc-build-playbook/
```

Windows PowerShell:
```powershell
git clone https://github.com/rohitsingh-sg/odc-build-playbook.git
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills\odc-build-playbook"
Copy-Item odc-build-playbook\SKILL.md "$env:USERPROFILE\.claude\skills\odc-build-playbook\"
```

Restart Claude Code or start a new session. The skill then appears in the skill list.

**Project skill, shared with a team**

Commit `SKILL.md` to `.claude/skills/odc-build-playbook/` in your project repo. Everyone who clones the repo gets it.

### Claude Desktop chat / claude.ai

Zip the `odc-build-playbook` folder (with `SKILL.md` inside) and upload it under **Settings > Capabilities > Skills**. Availability depends on your plan, and an admin may need to enable skills for the organization. The menu names may differ by version.

## Usage
Ask for any ODC build task, or type `/odc-build-playbook`. Example: "Build a Task Tracker ODC app with a Task entity and list/edit screens."
