# CLAUDE.md

> Operating manual for any agent working in this repository. Read fully at session start.
> Stack: **Next.js (App Router) · React · TypeScript · Payload CMS · PostgreSQL (Neon / Supabase)**.
> Keep this file high-signal. Push path-specific detail into `.claude/rules/`. CI policy lives in `minimum CI.md`.

---

## 0. Non-negotiable rules (read first)

1. **YOU MUST run the verification gate before any `git push`, pull request, or change to a remote branch.** No exceptions. See §6. If the gate is red, you do not push — you fix, then re-run until green. Show the command output as proof, never just claim success.
2. **NEVER commit secrets.** No `.env`, API keys, tokens, DB URLs, service-role keys in code, config, fixtures, or commit messages. `gitleaks` runs pre-commit; if it fires, stop and remove the secret.
3. **NEVER edit generated code, migrations already applied, or `payload-types.ts` by hand.** Regenerate them.
4. **If you cannot verify it, do not ship it.** "Looks done" is not a signal. A test, build, lint, or type-check passing is.
5. **Default model of the agent:** treat yourself as a senior engineer who explores and plans before writing, and as a junior engineer who must prove every claim with evidence.

---

## 1. Always-on skills (every request)

These skills are mandatory and load automatically when their trigger matches. Do not improvise an approach when a skill exists for it.

- **caveman** → active for **every response**. Telegraphic output: fragments, no filler, no "Great question!", no restating the prompt, no sign-offs. Cuts ~65-75% output tokens. *Scope:* compresses **prose output only** — never compresses reasoning/thinking, code, plans shown for approval, spec text, or debugging hypotheses surfaced to the user. Brain big, mouth small.
- **superpowers** → active for **every feature**. Drives the structured methodology: brainstorming → spec → plan → subagent-driven execution with built-in review. Enforces true red/green/refactor TDD, YAGNI, DRY.
- **ponytail** → active for **every feature, alongside `superpowers`**. "Lazy senior dev" discipline: before writing code, walk the decision ladder (does it need to exist? already in the codebase? stdlib? native platform feature?) and stop at the first rung that holds — minimal, reuse-first code. `/ponytail [lite|full|ultra|off]` sets intensity (default `full`); `/ponytail-review` flags over-engineering in a diff, `/ponytail-audit` scans the repo. Reinforces YAGNI/DRY — never a license to skip a failing test, TDD, or the §6 gate.
- **systematic-debugging** (part of superpowers) → active for **every debug / fix / "why is this broken"** request. 4-phase root-cause process: reproduce → isolate root cause → validate assumption → fix + confirm. After **3 failed fix attempts**, stop patching and trigger architectural review.
- **Documentation lookup** → active for **every request touching a library, framework, or API**. Use the **context7 MCP** for version-specific live docs; if context7 is unavailable, fall back to **`web_search`**. Never answer framework/API questions from memory — versions drift. See §4.

> If a request is mechanical (rename, mechanical refactor, batch fix), caveman + the relevant skill is enough. If it is design/architecture/learning, keep reasoning visible inside thinking; caveman still compresses the final prose.

---

## 1.5 Required local setup (provision if missing)

**YOU MUST have all three of these available in any repo that loads this ruleset.** They are mandatory, not optional. If one is missing, provision it yourself — idempotent, no human needed — reusing the machine's `gh` login. All of this requires `gh auth status` to be logged in; if it is not, tell the user to run `gh auth login` and do **not** silently skip the gate or the wiki.

1. **`prod-gates` plugin** — the production quality gate ([`Tensory-srls/prod-gates`](https://github.com/Tensory-srls/prod-gates)). Run `/prod-gates:prod-gate <category>` as the gate before any production push (see §6). Provision:
   ```bash
   claude plugin marketplace list 2>/dev/null | grep -q tensory-prod-gates || claude plugin marketplace add Tensory-srls/prod-gates
   claude plugin list 2>/dev/null | grep -q prod-gates || claude plugin install prod-gates@tensory-prod-gates
   ```

2. **`github-wiki` MCP** — read-only access to the internal wiki (full rationale and usage in §4). Provision:
   ```bash
   claude mcp get github-wiki >/dev/null 2>&1 || \
     claude mcp add-json -s user github-wiki "{\"type\":\"http\",\"url\":\"https://api.githubcopilot.com/mcp\",\"headers\":{\"Authorization\":\"Bearer $(gh auth token)\",\"X-MCP-Readonly\":\"true\",\"X-MCP-Toolsets\":\"repos,context\"}}"
   ```

3. **`ponytail` plugin** — minimal-code "lazy senior dev" discipline ([`DietrichGebert/ponytail`](https://github.com/DietrichGebert/ponytail)). Loads alongside `superpowers` on every feature (see §1). Provision:
   ```bash
   claude plugin marketplace list 2>/dev/null | grep -qi ponytail || claude plugin marketplace add DietrichGebert/ponytail
   claude plugin list 2>/dev/null | grep -qi ponytail || claude plugin install ponytail@ponytail
   ```

A plugin install or MCP change may need `/reload-plugins` or a session restart to expose its tools. Verify with `claude plugin list` and `claude mcp list`.

---

## 2. The mandatory workflow: research → plan → execute → review → ship

Never jump straight to code. The research and planning steps are where quality is decided.

**1. Research (plan mode).** Read the relevant files and the internal wiki (§4). Do not modify anything. For non-trivial work, **interview the user**: ask about technical details, UX, edge cases, trade-offs until the spec is self-sufficient. A good spec names the files and interfaces involved, states what is out of scope, and ends with an end-to-end verification step.

**2. Plan.** Produce a written plan the user can read and edit. Short enough to digest in chunks. For features, this is the `superpowers` brainstorming + writing-plans flow. Wait for explicit "go".

**3. Execute.** Implement in a clean session against the approved spec. TDD by default (§5). Small, reviewable increments. Commit per task (Conventional Commits, ≤50-char subject, "why" over "what").

**4. Review (adversarial, fresh context).** Before declaring done, dispatch a reviewer subagent with **fresh context** that sees only the diff and the acceptance criteria — not your reasoning. It judges the result on its own terms. **Constraint:** the reviewer flags **only gaps that break correctness or stated requirements** — not stylistic preferences or speculative improvements (avoids over-engineering).

**5. Ship.** Run the verification gate (§6). Green → push/PR with evidence. Red → back to execute.

---

## 3. Prompt-engineering & context discipline

Context is infrastructure. Performance degrades as the window fills.

- **`/clear` between unrelated tasks.** Never run unrelated tasks in one thread (no "kitchen-sink session").
- **Delegate investigations to subagents.** They explore in separate context and return only a summary. Keeps the main thread lean.
- **Bound exploration.** No open-ended spelunking that eats the window. Define what you're looking for first.
- **After 2 failed fix attempts, stop.** `/clear`, write a better initial prompt, restart. Do not loop forever on the same broken approach.
- **Adopt the agent's perspective.** Before deciding, ask what information is actually in context at decision time. Most surprising behavior comes from missing context, not bad judgment.
- **Reference, don't duplicate.** Point to `README`, `minimum CI.md`, `.claude/rules/` instead of restating them here.
- **Emphasis is rationed.** `IMPORTANT` / `YOU MUST` only on load-bearing rules. If everything is important, nothing is.

---

## 4. Documentation: external + internal

**External (live docs).** For any library/framework/API touched by a request:
1. Query the **context7 MCP** for version-specific docs and code examples (Next.js, React, Payload, Postgres drivers, Neon/Supabase SDKs, etc.).
2. If context7 is unavailable or the package isn't indexed, use **`web_search`** against official docs.
3. Match the version in this repo's `package.json` — do not assume latest.

**Internal knowledge base.** The team's conventions, patterns, and decisions live in the **private** repo `Tensory-srls/llm-web-dev-wiki`. Access it **on demand through the `github-wiki` MCP server** — do **not** clone it into the project and do **not** `@import` it (it is large; pull only the pages you need, never the whole wiki into context).

**Self-setup (automatic — no human action).** Before consulting the wiki, if the `github-wiki` MCP server isn't configured, configure it yourself from the machine's existing GitHub CLI login, then use it. Idempotent — run it and move on:

```bash
# requires `gh auth status` logged in; reuses the gh token, capped read-only at the MCP layer
claude mcp get github-wiki >/dev/null 2>&1 || \
  claude mcp add-json -s user github-wiki \
    "{\"type\":\"http\",\"url\":\"https://api.githubcopilot.com/mcp\",\"headers\":{\"Authorization\":\"Bearer $(gh auth token)\",\"X-MCP-Readonly\":\"true\",\"X-MCP-Toolsets\":\"repos,context\"}}"
```

`X-MCP-Readonly`/`X-MCP-Toolsets` cap the server to read-only `repos`+`context` tools, so it can only read repositories. If `gh` is not authenticated, fall back to a one-off `git clone` of the wiki into `.cache/` (gitignored) and consult it locally.

**Usage.** Through the `github-wiki` MCP: start at the categorized table of contents **`myWiki/wiki/index.md`** with `get_file_contents` on `Tensory-srls/llm-web-dev-wiki` (default branch `main`; the ~153 pages live under `myWiki/wiki/`), follow the `[[wikilinks]]` into the specific entity/concept/decision pages, and use `search_code` to locate relevant ones. **Prefer the wiki's conventions over generic defaults**: if the wiki and your instinct conflict, the wiki wins; if the wiki is silent, follow stack conventions in §7 and flag the gap.

*Curated reference indexes (optional, discovery aids).* Distinct from the wiki above: awesome-lists that help find *which* library or tool exists for a problem — **not documentation**. Once you have a name, get version-specific docs the usual way (context7 → `web_search`); never paste these lists into context. Clone into `.cache/` (gitignored), grep the README, pick a candidate. Reach for one only when its scope matches the task:

| Index | Use it when |
|---|---|
| [`enaqx/awesome-react`](https://github.com/enaqx/awesome-react) | **On-stack.** Choosing a React/Next ecosystem library (state, forms, data, testing) before committing to a dependency. |
| [`brillout/awesome-react-components`](https://github.com/brillout/awesome-react-components) | **On-stack.** Finding a specific React UI component (tables, forms, modals, charts) before building it from scratch. |
| [`sindresorhus/awesome-nodejs`](https://github.com/sindresorhus/awesome-nodejs) | **On-stack.** Picking a Node.js package for server-side / Payload / tooling work before adding a dependency. |
| [`trimstray/the-book-of-secret-knowledge`](https://github.com/trimstray/the-book-of-secret-knowledge) | Off-stack. CLI / ops / networking tool discovery — devops chores, infra troubleshooting. |
| [`Hack-with-Github/Awesome-Hacking`](https://github.com/Hack-with-Github/Awesome-Hacking) | Off-stack. Security / pentest research — around the §6 security gate (`gitleaks` / `semgrep` / `osv-scanner`), threat modeling, hardening. Not for feature work. |

```bash
# clone or update the indexes you need into .cache/ (gitignored)
for repo in enaqx/awesome-react brillout/awesome-react-components sindresorhus/awesome-nodejs trimstray/the-book-of-secret-knowledge Hack-with-Github/Awesome-Hacking; do
  dir=".cache/$(basename "$repo")"
  git clone "https://github.com/$repo.git" "$dir" 2>/dev/null || git -C "$dir" pull --ff-only
done
```

---

## 5. Test-Driven Development & quality bar

TDD is the default, not an option.

- **Red → Green → Refactor.** Write a failing test that encodes the requirement. Watch it fail. Write the minimum code to pass. Refactor with tests green. Never write implementation before a failing test exists for it.
- **Tests verify behavior, not implementation.** A test must fail if the behavior breaks. Tests that pass by construction are worthless — they are the #1 failure mode of agent-written code.
- **Coverage is a floor, not a goal.** Line coverage lies. The real gate is **mutation score (Stryker) ≥ 80%** on the touched modules (see `minimum CI.md` §3). If mutants survive, strengthen assertions.
- **Cover happy path, edge cases, and error states** — not just the obvious case.
- **Show evidence.** Paste the test command run and its result. Don't assert "tests pass."

---

## 6. CI verification gate (before every push / PR / remote change)

Full policy and rationale: **`minimum CI.md`**. This is the enforced subset. Every gate is deterministic and binary; the CI is the source of truth, not your confidence.

Run with a **single command** before going to the remote. Adapt to this repo's package manager (`pnpm` / `npm` / `yarn` / `bun`) and scripts.

```bash
# ── verify (fast: pre-commit + pre-push) — must be GREEN before any push ──
biome ci .                      # format + lint check (no write), exit 1 on violation
tsc --noEmit                    # strict type check
vitest run --coverage           # tests + coverage thresholds (blocking)
gitleaks protect --staged       # secret scan on staged changes

# ── verify:pr (before opening / updating a PR) ──
eslint . --format sarif         # react-hooks + type-aware semantic rules
knip                            # dead code / unused exports & deps
semgrep ci                      # SAST → SARIF
gitleaks detect                 # secret scan incl. git history
<build> && size-limit           # production build (next build / vite / etc.) + bundle weight
osv-scanner -r .                # dependency CVEs

# ── nightly / critical PRs ──
stryker run                     # mutation testing, MSI ≥ 80% on touched modules
```

Suggested `package.json` scripts (create if missing):

```json
{
  "scripts": {
    "verify": "biome ci . && tsc --noEmit && vitest run --coverage && gitleaks protect --staged",
    "verify:pr": "pnpm verify && eslint . --format sarif && knip && semgrep ci && pnpm build && size-limit && osv-scanner -r .",
    "verify:mutation": "stryker run"
  }
}
```

Rules:
- **`biome ci` in CI never writes.** Fix locally with `biome check --write`, then let the gate verify.
- **Fail the build on HIGH/CRITICAL** security findings; allowlist only with documented justification + expiry.
- **All security tools emit SARIF** → upload via `github/codeql-action/upload-sarif@v3` so findings land in the GitHub Security tab.
- **Output machine-readable** (`--format json` / `--reporter json` / SARIF) so failures can be parsed and fixed autonomously, not guessed from prose logs.
- **Branch protection makes these gates blocking.** No merge without green.

---

## 7. Stack conventions

**Next.js (App Router) + React + TS**
- Server Components by default; `"use client"` only when the component needs interactivity/state/effects.
- Data fetching in Server Components / Route Handlers / Server Actions — never expose secrets to the client bundle.
- TypeScript `strict: true`. No `any`; use `unknown` + narrowing. No non-null `!` to silence the compiler.
- ES modules, named exports, destructured imports.
- Co-locate tests (`*.test.ts(x)`) with the code they cover.

**Payload CMS**
- Collections/globals/fields defined in config; after any schema change **regenerate types** (`payload generate:types`) — never hand-edit `payload-types.ts`.
- Use Payload's Local API on the server instead of HTTP round-trips where possible.
- Access control is explicit per collection — default to deny, open up deliberately. Never widen access to make a test pass.
- Migrations are versioned and reviewed; never edit an already-applied migration.

**PostgreSQL (Neon / Supabase)**
- All schema changes go through migrations — no ad-hoc DDL against the DB.
- Connection strings come from env only. Pooled connection for serverless (Neon pooler / Supabase pgBouncer); direct connection for migrations.
- Parameterized queries only — never string-concatenate SQL (SQL injection is a top agent-generated vuln).
- If using Supabase RLS, write/verify the policy; an open table is a finding, not a convenience.

> Project-specific overrides live in `.claude/rules/` with path globs (e.g. a `src/payload/**` rule, a `src/app/**` rule). Path-scoped rules load only when the session touches that part of the tree — keep them there, not here.

---

## 8. Commands

Adapt to this repo's actual package manager and scripts (check `package.json` / lockfile first; do not guess).

```bash
<pm> install            # install deps
<pm> dev                # run dev server (next dev)
<pm> build              # production build
<pm> test               # vitest (prefer running a single test file while iterating)
<pm> verify             # fast gate — REQUIRED before push (see §6)
<pm> verify:pr          # full gate — before PR
payload generate:types  # regenerate Payload types after schema change
payload migrate         # apply DB migrations
```

Run **single tests** while iterating (`vitest path/to/x.test.ts`), not the whole suite — faster feedback loop.

---

## 9. Anti-patterns (do not do)

- Kitchen-sink session (unrelated tasks in one thread) → `/clear` between them.
- Infinite fixing (>2 failed attempts on the same approach) → `/clear`, rewrite the prompt.
- Pushing without the green verification gate (§0.1).
- Editing generated files, applied migrations, or `payload-types.ts` by hand.
- Writing implementation before a failing test exists.
- Claiming success without showing the command output.
- Widening security/access rules to make a test or build pass.
- Answering version-specific framework questions from memory instead of context7 / web_search.

---

## 10. Meta

These are strong defaults, not dogma. Occasionally it's right to let context accumulate, skip planning on a trivial change, or accept a vague prompt. Real skill is the judgment of when to deviate — developed by watching what works. When you deviate from a rule here, say so and why.
