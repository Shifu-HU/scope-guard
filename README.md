# scope-guard

> An agent skill that stops your assistant from doing things you never asked for.

Most agents are eager to help. You ask for a one-line fix and get a refactor. You ask for a rename and get reformatted files, a new test suite, three "while I was in there" cleanups, and a commit. `scope-guard` shuts that down: the agent does exactly what you asked, and asks before anything else.

## What it enforces

- **Visible authorization citation.** Before any write action the agent must quote the user's sentence that authorizes it — `Authorization: "…"` — in its reply. If it can't, it stops and asks. Internal self-checks get rationalized away; visible ones can't.
- **Default restate-and-confirm.** For any write or multi-step task the agent restates the plan first (which files, what action, what scope) and waits for an "ok". Only a fully itemized instruction skips this.
- **Full freeze when stopping.** Stopping to ask means doing *nothing* — not even the "certain part" or harmless-looking writes.
- **Only what you asked.** Nothing extra, ever.
- **Zero-guess red lines.** Confidence is not an exemption — "90% sure" is still a guess. Missing paths, names, and numbers may not be filled in from context or chat history. Guessing *goals* counts too: being asked to fix A doesn't license touching B. Previous approvals don't carry over to new tasks.
- **Vague commands get clarified.** "Continue / do it / go ahead" only counts as authorization when the pending task is unique and item-by-item confirmed; otherwise the agent restates its understanding or offers numbered options first.
- **Ask first, always.** Target, scope, files, action type, output format, or whether interfaces / config / dependencies / docs / style may change — any uncertainty means stop and ask, *before* acting.
- **No "let me do the certain part first."** Partial progress on an ambiguous task is still unrequested work.
- **Minimal action.** Read before write, local before global, one before batch, change nothing unmentioned.
- **No drive-by work.** No refactors, renames, formatting, dependency or config changes, deletions, comments, logs, tests, docs.
- **No assumption-filling.** "I assume…" becomes "Please confirm…".
- **Cheap questions.** Every uncertainty is asked as numbered options with a recommended default, all at once — the user answers with a single "1" or "go with the suggestions".
- **Stop mid-task on new uncertainty.** Discovering a new ambiguity means stop, not improvise.
- **External actions need consent.** Commits, pushes, deploys, publishes, messages.
- **Report, don't expand.** Close with what was done / not done / what needs confirmation. No unrequested next steps or extra options.

## Install

### User level (applies to all your projects)

```bash
git clone https://github.com/Shifu-HU/scope-guard ~/.workbuddy/skills/scope-guard
```

Or just drop the file in place:

```bash
mkdir -p ~/.workbuddy/skills/scope-guard
cp SKILL.md ~/.workbuddy/skills/scope-guard/SKILL.md
```

### Project level

```bash
mkdir -p .workbuddy/skills/scope-guard
cp SKILL.md .workbuddy/skills/scope-guard/
```

For DeepSeek Harness-style loaders, use the corresponding skills directory (e.g. `~/.dsh/skills/scope-guard/`).

### Make it apply to *every* conversation

Skills are loaded on demand — a skill on disk is not injected into every request. If you want these rules to apply unconditionally, put the essentials into the always-injected layer of your agent:

- WorkBuddy: `~/.workbuddy/MEMORY.md`
- DeepSeek Harness: the persona / system-prompt patch (e.g. `~/.dsh/profiles/<profile>/cordis.patch.yml`)

Keep it short there and let the full skill file carry the detail.

## Compatibility

Pure text. No scripts, no dependencies, no network calls, no credentials, no file access. Works with any agent that loads `SKILL.md`-style skills (Claude Code, WorkBuddy, and similar). Safe to audit in five minutes by reading one file.

## Pairs well with ponytail

[ponytail](https://github.com/DietrichGebert/ponytail) stops **over-engineering** — the laziest solution that works. `scope-guard` stops **over-stepping** — doing more than was asked. They cover different failure modes and compose cleanly:

- ponytail: "does this code need to exist?"
- scope-guard: "did the user ask for this at all?"

To have both apply together, append the `## Scope guard (hard rule — overrides laziness)` section to your ponytail family skill files.

## Example

> **User:** Fix the login error.
>
> **Agent without scope-guard:** Fixed it. I also refactored the auth module, renamed `doLogin` to `authenticate`, upgraded `jsonwebtoken` to the latest major, added unit tests, and committed. Want me to push?
>
> **Agent with scope-guard:** Before I continue, please confirm:
> 1. Only fix the error, or also refactor surrounding code? — A. fix only (suggested) / B. refactor too.
> 2. May I change public interfaces? — A. no (suggested) / B. yes.
> 3. May I add or change tests? — A. no / B. minimal regression test (suggested).
> 4. Should I commit afterwards? — A. no (suggested) / B. yes.
>
> Reply with the numbers or "go with the suggestions". I won't touch the code until you confirm.
>
> **User:** Continue.
>
> **Agent with scope-guard (several pending items):** Continue with which one? 1) push the README changes (suggested) 2) regenerate the diagram 3) clean up backups. I won't act until you reply.

## License

MIT — see [LICENSE](LICENSE).

Chinese version of this document: [README.zh-CN.md](README.zh-CN.md)
