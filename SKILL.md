---
name: railway-post-breach-checklist
description: 'Use after Railway''s October 2026 security incident (attack window Oct 20, 13:58 UTC to Oct 22, 03:32 UTC), when a user asks what to do about the Railway breach, how to rotate secrets stored in Railway variables, or how to check their Railway projects afterwards. A short, ordered checklist with no scripts: crypto wallets first, a names-only inventory of variables, rotation in priority order (new key live before the old one is revoked; Regenerate for Railway-hosted databases), a look at deployments and activity in the incident window, an access review, and a short written record. The agent never prints or handles secret values, and the user confirms every change.'
---

# Railway post-breach checklist

A plain checklist for cleaning up after Railway's October 2026 security incident. Work through it in order with the user. Not affiliated with or endorsed by Railway.

## What Railway has said (stick to this)
- An attacker accessed encrypted environment variables between **Oct 20, 13:58 UTC and Oct 22, 03:32 UTC**, and decrypted a subset. Railway puts the affected share at no more than 1.85% of workspaces, says the attack appeared aimed at AI provider API keys, and can't tell which specific keys were decrypted.
- Railway asks users to rotate sensitive credentials stored in Railway variables, revoke the old ones, update the variables and redeploy. Its Oct 23 update says workspaces outside the higher-risk groups "are not confirmed safe from compromise." It also advises removing public network access to databases and reviewing GitHub and container registry audit logs.
- Railway says accounts were not logged into. It revoked all Railway API tokens and CLI sessions, revoked GitHub connections (they must be reconnected), and reset storage bucket credentials.
- The preliminary report is still the latest as of Oct 27, 2026. The final report isn't out, and source code access hasn't been ruled out.
- A security researcher (Taylor Monahan, reported by crypto.news and The Hacker News) linked crypto wallet drains to the incident. Railway has not confirmed those thefts.

Don't add to these facts, and don't tell the user whether their workspace was affected. Only Railway can say that. A revocation email from a vendor doesn't prove a key was taken (Railway said so), but that key may already have stopped working.

Treat anything stored on Railway before Oct 22, 03:32 UTC as in scope, unless the issuer's dashboard shows the current key was created after that. Signups stayed open until Oct 22, 19:42 UTC, so a new account isn't automatically out of scope.

## Safety rules (follow all of them)
1. **Never print, read, store or pass along a secret value**, or any part of one. Work with variable names only. Don't run `railway variables`, `railway variable list` or `railway run env`, don't open a `.env` file, and don't call a Railway MCP tool that returns values (`list-variables`, `get-bucket-credentials`) or raw log text (`get-logs`).
2. **The user handles every value.** They create new keys at the issuer and paste them into the Railway dashboard. The agent may write a `${{shared.NAME}}` reference, never a value. Never ask for a key, token, password or seed phrase in the chat. If the user pastes one anyway, don't repeat it, and add that secret to the rotation list.
3. **The user confirms each change, one at a time.** Before each one, say what will change, what might break and how to undo it. Approving one service or secret doesn't approve the next.
4. **Never call the user "safe" or "not affected".** Say what was checked, what changed, and what is still pending.
5. Don't delete projects, services, volumes or databases, and don't create or delete tokens or members for the user.

## Before you start
- Confirm scope: which workspaces, projects and environments. Include staging, PR environments and idle services.
- Ask where the same keys are also used outside Railway (CI, other hosts, laptops). Rotating a key at the issuer breaks those too.
- Service logs age out fast (Railway's docs: Free 3 days, Trial and Hobby 7, Pro 30, Enterprise up to 90). Ask the user to look at their logs for Oct 20-22 now, in the dashboard: unknown outbound hosts, error bursts, auth failures. The agent doesn't fetch logs.
- If the CLI says Unauthorized, the user runs `railway logout`, then `railway login`, in their own terminal.

## 1. Crypto wallets first
Ask straight away: does any project hold a wallet seed phrase or private key, under any variable name? If yes, this comes before everything else. A leaked wallet key can't be rotated: whoever copied it can move the funds at any time. The user does every step in their own wallet software. The agent explains the order and never asks for, sees, signs with or sends anything.
- [ ] Stop the service that uses the key (with approval) and pause anything that pays into the old address.
- [ ] Check the address on a block explorer for a sweeper: outgoing transactions the user didn't make, or a balance emptied right after each deposit. If either shows, **don't send gas to the old address**. Bots will take it in seconds. Moving tokens out then needs a private bundle where someone else pays the gas (Flashbots on Ethereum, a separate fee payer or a Jito bundle on Solana) or professional help. Be wary of unsolicited "recovery" offers; they are a common scam.
- [ ] Create a new wallet with a **new seed** on a clean device. A new account under the old seed isn't enough, since every address derived from a leaked seed is exposed.
- [ ] The user moves the funds and NFTs from every exposed address on every chain. On each chain, move tokens first and the native coin (ETH, SOL) last, so gas is there until the end. Move contract roles (owner, admin, minter) and replace the leaked signer on any multisig.
- [ ] Treat the old wallet as burned. Never fund it again. Only then put a new, low-balance hot key into Railway like any other secret (step 3).

## 2. Inventory: variable names only
Goal: one list of every variable **name** per project, environment and service, plus shared variables, with a guess at which ones are secrets and who issued them.
- With a Railway MCP connector, use tools that return names only (`describe-service`, `describe-environment`). If one shows a value, stop using it and don't repeat what it showed.
- Otherwise the user reads the names from each service's **Variables** tab (not the Raw Editor, which shows values) and types them in.
- Also list secrets that aren't variables: private registry credentials in service settings, webhook URLs and headers, Railway tokens used in CI, keys committed to the repo.
- Note which databases have a public TCP proxy (public networking turned on). Railway advises removing public access.

## 3. Rotate in priority order
Suggested order (the user decides): wallets (step 1), then AI provider keys, payment keys, cloud keys, GitHub and registry tokens, and database credentials, then everything else.

For each ordinary key, always in this order:
- [ ] The user creates a **new key at the issuer** and leaves the old one active.
- [ ] The user pastes it into Railway and **seals** it (the variable's 3-dot menu, then Seal). Sealed values can't be read back by anyone, which is what you want for a fresh key. For a key used by several services, add a **new** shared variable (for example `OPENAI_API_KEY_NEW`) instead of editing the old one: editing a shared variable redeploys every service that references it at once. Then switch one service at a time, with the agent writing `OPENAI_API_KEY=${{shared.OPENAI_API_KEY_NEW}}` once the user approves (single quotes in the shell).
- [ ] Check for staged (pending) changes first. Deploying an environment ships everyone's staged changes, so if someone else has some, stop and ask.
- [ ] Redeploy one service, then verify: the new deployment succeeded and stays up, a real call that uses the key works, and the issuer shows the new key in use and the old key's "last used" has stopped moving.
- [ ] Before redeploying, check the date of the current deployment. Railway says redeploying code from before the incident fails; if it predates Oct 22, 03:32 UTC, ask the user whether to deploy the latest commit instead.
- [ ] Update consumers outside Railway (CI, other hosts).
- [ ] **Only then** does the user revoke the old key. Say clearly that this can't be undone. Until this point, every step can be rolled back.

Special cases:
- **Databases hosted on Railway** (host ends in `.railway.internal` or `.rlwy.net`): the user clicks **Regenerate** in the database service (Database > Config > Connection), per Railway's per-engine guides. Don't edit the password variables by hand and don't change the password with SQL. Postgres and MySQL restart and drop every session; Redis and MongoDB clients using the old password start failing. There's no undo, so pick a quiet time, then redeploy each app that uses it and update anything outside Railway that uses the public URL.
- **Databases hosted elsewhere** (Neon, Supabase, RDS and so on): there's no Regenerate. Rotate the password at that provider, then update the Railway variable like any other key.
- **Public database**: once nothing outside Railway needs it, offer to remove the TCP proxy (separate approval; it can't simply be put back at the same address). Rotating the password and closing public access are two separate fixes, and you need both.
- **Single-key issuers** (making a new key kills the old one): plan a short outage in a quiet window and update every consumer right away.
- **Signing secrets** (sessions, JWTs, cookies) sign users out when rotated. **Data-encryption keys**: don't swap them here; note "needs a re-encryption plan".
- **Railway tokens** were all revoked by Railway. Where one is still needed, the user creates a new, narrowly scoped one and puts it straight into the CI secret store.
- **GitHub tokens used as registry credentials**: Railway removed them from services, but they still work on GitHub until the user deletes them there.
- If an OpenAI or Anthropic key created after the incident gets revoked, Railway's Oct 24 notice says that may be precautionary and doesn't mean it was compromised. Create another key and redeploy.

## 4. Deployments and activity in the window
Window: 2026-10-20 13:58 UTC to 2026-10-22 03:32 UTC, plus the days after, up to when each key was rotated.
- [ ] Per service, list deployments in that window: ID, status, time, who triggered it. Don't repeat commit messages or branch names; they're free text and can hold secrets. Builds were paused for part of the window, so few or no deployments is normal.
- [ ] The user checks the workspace audit log for members added, tokens created, variable reads or changes, new TCP proxies or domains, and source changes. Retention: 48 hours on Free, Trial and Hobby, 30 days on Pro, 18 months on Enterprise, so on Pro the window ages out around Nov 19-21. "Railway Staff" entries for bucket credential resets and GitHub token removals are expected.
- [ ] The user checks issuer usage pages from Oct 20 on: AI token spend spikes, calls from unknown IPs, unexpected charges. Railway says it can't refund fraudulent AI spend; contact the provider first.
- [ ] The user opens each storage bucket and looks for files they didn't upload or deletions they didn't make. Buckets have no automatic backups.

## 5. Access review (read-only; the user removes anything)
- [ ] Railway: workspace and project members, pending invites, account, workspace and project tokens, SSH keys (railway.com/account/ssh-keys), webhook destinations.
- [ ] GitHub: export the security log (and the org audit log) first, before revoking anything. Then review installed GitHub Apps (keep Railway's app on selected repos only), authorized OAuth apps, SSH and GPG keys, and personal access tokens. Per repo deployed on Railway: deploy keys, webhooks, collaborators, Actions secrets, and pushes in the window. Reconnect Railway's GitHub integration if deploys fail.
- [ ] Container registries (Docker Hub, GHCR, others): tokens and image pushes in and after the window.

Report anything odd as "needs follow-up", not as proof of an attack.

## 6. Keep a short record
Write a short report the user keeps: scope, what was rotated (name, issuer, time, how it was verified, when the old key was revoked), what's pending and who owns it, what was skipped and why, what couldn't be checked (for example, logs that had aged out), and one line per wallet (funds moved, old address burned). Names, dates and counts only, no values or addresses. Give the incident window in UTC and change times in the user's own zone, labelled. Remind the user to check Railway's report again, since the final version isn't out yet.
