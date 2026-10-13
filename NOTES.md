# Notes

## What's in CLAUDE.md, what's left out, and why

CLAUDE.md holds only what Claude can't quickly work out from the code and would otherwise get wrong. That covers the exact commands (including how to run a single test), the CI gate (lint then test), and the project conventions. The conventions are: CommonJS only; `node:test` + supertest with no real server; all data access goes through `db/store.js`; errors use the `{ error }` JSON shape; don't touch `.env`. It also gives a short architecture overview. That overview explains the `require.main === module` trick that makes `app` testable, and warns that the in-memory store is shared across tests.

I left out anything Claude can read directly or that goes stale fast. That means no file-by-file listing, no dependency versions or full `package.json` scripts, no copy of the README, and no generic advice like "write clean code" or "add tests". Every line in CLAUDE.md is loaded into context on every session, so filler costs tokens and dilutes the rules that matter.

## Permission rules, and what the deny rules prevent

`.claude/settings.json` auto-allows the safe, frequent, read-only or local-only commands: `npm test`, `npm run lint`, `node --test`, `git status`, `git diff`, `git log`, `git add`, `git commit` and `git checkout -b`. That way routine work doesn't stall on prompts. `git push` and `npm install` are set to **ask**, because they reach outside the machine (the remote, the npm registry) and can't be fully undone.

The **deny** rules block reading or editing `.env`, `git push --force`/`-f`, and `git reset --hard`. Without them, Claude could read `.env` while exploring and pull secrets into its context. Once there, they're sent to the model, and they could end up in a log, a test fixture, a commit or a pasted snippet. A force push could overwrite teammates' commits on the remote. `reset --hard` silently throws away uncommitted work. All of these are fast, easy to trigger by accident, and hard or impossible to reverse, so they should never run without a human deciding to.

One caveat: `Read(./.env)` covers Claude's file tools. It isn't a hard sandbox against every shell command (e.g. `cat .env`), so `.gitignore` is still the real guard against committing it.
