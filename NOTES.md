# Notes

## What did I put in my CLAUDE.md, and what did I deliberately leave out, and why?

**What I kept.** A one-line description, then three short sections:

- **Commands:** `dev`, `test`, how to run a single test by name, and `lint`. I also noted that CI runs lint and then the tests, so Claude knows to run both before committing.
- **Conventions:** rules Claude couldn't safely guess from one file. Use CommonJS, not ES modules. Write tests with `node:test` and `supertest`, not Jest. Go through `db/store.js` instead of touching the `users` array. Use the JSON error shape and status codes. Convert `:id` params with `Number(...)` because the store compares with `===`.
- **Architecture:** `server.js` exports `app` and only calls `listen()` when run directly. Each resource gets one router file in `routes/`. The store is in-memory state that all tests share, so tests must not assert exact lengths or ids.

I kept these because each one is either a mistake Claude would plausibly make (ESM syntax, Jest, a string id that never matches) or a fact it would otherwise have to rediscover every session.

**What I left out.**

- **Things Claude can read from the code in seconds**, such as the full list of endpoints, the dependency versions and a file-by-file tour. These go stale as the code changes, and they cost context on every session.
- **General Express or Node knowledge** Claude already has.
- **Anything sensitive.** No `.env` values, keys or URLs. Secrets are kept out of Claude's reach by a deny rule in `.claude/settings.json`, not by a note asking it to behave.
- **One-off tasks and course instructions** like "write NOTES.md" or the submit steps. They matter once, not on every session, so they belong in the README or the prompt.
- **Long pasted docs.** A shorter file means every line gets read and followed.

## Which permission rules did I add, and what could go wrong without my deny rule?

These are in `.claude/settings.json`, which is committed so the whole team shares them:

- **allow `Bash(npm test:*)`:** running tests is safe and constant, so Claude can verify its own changes without prompting me every time.
- **ask `Bash(git push:*)`:** pushing is visible to others and triggers CI, so I want to confirm each one.
- **deny `Read(./.env)`:** `.env` is where real secrets live. Without this rule, Claude could read the file while "understanding the config". The values would then sit in the conversation, where they could be echoed into output, logs, a commit or a test fixture, or sent to an external tool.
- **deny `Bash(git push --force:*)`:** a force-push can overwrite commits on a shared branch and silently erase someone else's work. That's hard to undo, so it's blocked outright rather than just asked about.

At first the two deny rules were accidentally merged into one malformed string. Claude Code skipped it with a warning, so for a while neither rule was active. That taught me to check the startup warnings and `/permissions` instead of assuming a rule works because it's in the file. The force-push rule is also a prefix match, so `git push -f` isn't denied. My `git push` ask rule still catches it, though.
