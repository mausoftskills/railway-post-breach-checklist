# Railway Post-Breach Checklist

A free agent skill (SKILL.md format) for Railway users cleaning up after Railway's October 2026 security incident. Your AI agent works through a short, ordered checklist with you: wallets first, then a names-only inventory of your variables, rotation in a safe order, a look at what happened during the incident window, an access review, and a short record of what was done.

The agent never prints or handles secret values. You create and paste new keys yourself, and you confirm every change before it happens. There are no scripts; it's a checklist the agent follows.

## The incident
Railway reported that an attacker accessed encrypted environment variables between Oct 20, 13:58 UTC and Oct 22, 03:32 UTC, and decrypted a subset of them. Railway put the affected share at no more than 1.85% of workspaces and asked users to rotate sensitive credentials stored in Railway variables, remove public access to databases, and review GitHub and registry audit logs. As of Oct 27, 2026 the preliminary report is still the latest; Railway hasn't published its final report.

- Railway, preliminary report (updated Oct 26): https://blog.railway.com/p/preliminary-report-october-security-incident
- Railway status page, incident updates (Oct 21-23): https://status.railway.com/incident/QH8L8B3E
- Railway changelog, Oct 23: https://railway.com/changelog/2026-10-23-october-21-security-incident
- Railway status page, recovery updates (Oct 26-27): https://status.railway.com/incident/O1VGXWU5

Outside reporting (crypto.news, The Hacker News, citing researcher Taylor Monahan) has linked crypto wallet drains to the incident. Railway has not confirmed those thefts. The checklist puts wallets first either way, because a leaked wallet key can't be rotated.

## Install
Copy the `railway-post-breach-checklist` folder (the one with `SKILL.md` directly inside) into your agent's skills folder. The folder name has to match the `name` in SKILL.md.

- Claude Code: `~/.claude/skills/` (or `.claude/skills/` in one project)
- Cursor: `~/.cursor/skills/` (or `.cursor/skills/` in one project)
- Other agents that read SKILL.md: their own skills folder, often `.agents/skills/`

For example:
```bash
git clone https://github.com/mausoftskills/railway-post-breach-checklist
mkdir -p ~/.claude/skills
cp -r railway-post-breach-checklist/railway-post-breach-checklist ~/.claude/skills/
```

Then ask your agent something like:
> Go through the Railway post-breach checklist with me. Don't change anything without asking.

The agent works best with the Railway CLI logged in (run `railway login` yourself) or a Railway MCP connector, but it can walk you through the dashboard by hand too.

## What it covers
1. Crypto wallets first: stop the service, check for a sweeper before sending any gas, new seed on a clean device, and you move the funds yourself.
2. An inventory of variable names (never values), including secrets that live outside variables.
3. Rotation in priority order: new key live before the old one is revoked, sealed and shared variables, Regenerate for Railway-hosted databases, rotation at the provider for external ones.
4. Deployments, audit log, issuer usage and buckets during the incident window.
5. Access review: Railway members and tokens, SSH keys, GitHub apps and keys, registries.
6. A short written record of what was done and what's still pending.

## What the paid version adds
This checklist is meant to be enough on its own if you're careful. The paid pack, [Railway Post-Breach Check](https://www.agensi.io/skills/railway-post-breach-check) ($12), is the same process with tooling around it, for people with many services or who want the agent to do more of the legwork safely:

- Offline Python scripts (standard library only) that read variables and print names, a secret flag, a kind, an issuer guess and an optional fingerprint, so the agent can take a full inventory without values ever reaching the chat. Text that looks like a secret in name or author fields is screened out.
- A rotation plan generator that turns the inventory into a prioritised table and spots the same key stored in several places.
- A deployment-window checker that lists deployments in the incident window without printing commit messages.
- A database exposure check for public TCP proxies.
- A list of known gotchas (staged changes, shared-variable redeploys, sealing, rollbacks, log and audit-log retention, deployments that won't redeploy after the incident).
- Detailed phase guides, including wallet rescue and per-engine database steps, and notes on specific issuers.
- A report template, and self-tests that run the scripts on fake secrets and fail if anything leaks.

## Limits
This is a checklist, not a guarantee. It can't tell you whether your keys were taken, and finishing it doesn't mean your systems are safe. It means the recommended steps were done and written down. Railway's advice may change when the final report comes out, so check their posts again.

Railway is a trademark of its owner. This project is not affiliated with or endorsed by Railway.

## License
MIT. See [LICENSE](LICENSE).
