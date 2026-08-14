# mistakes.md

A running log of mistakes made while building/PR'ing templates for this repo —
schema errors, failed PRs, misread requirements, bad assumptions about an
upstream project, anything kaywoz had to correct. Read this file in full
before starting new work. Add a new entry any time a PR fails, gets rejected,
or turns out to have been based on a misunderstanding.

## Entry format

```markdown
### YYYY-MM-DD — <slug or template name>

- **Trigger:** PR rejected / CI failed / merged-but-wrong / misunderstood request
- **What happened:** one or two sentences, factual, no editorializing
- **Root cause:** why it happened — bad assumption, missed doc, wrong field, etc.
- **Fix applied:** what was changed to correct it
- **Rule going forward:** one concrete, checkable rule to add to CLAUDE.md
  (or a pointer to the CLAUDE.md section it was added to)
```

Keep entries short and specific. The goal is a checklist that prevents repeat
mistakes, not a diary.

---

### 2026-08-14 — mirror-workflow-keepalive

- **Trigger:** misunderstood request
- **What happened:** Recommended an external scheduler (cron-job.org) calling
  `workflow_dispatch` via the GitHub API to stop the scheduled workflow from
  being auto-disabled.
- **Root cause:** Assumed the 60-day auto-disable was tied to the `schedule:`
  trigger specifically and could be bypassed by an externally-triggered
  `workflow_dispatch` call. Actual mechanism: disable is based on repo
  commit/push inactivity, and a disabled workflow doesn't run from any
  trigger, including `workflow_dispatch`.
- **Fix applied:** Dropped the external-scheduler suggestion. Added a step
  that commits a heartbeat file to the repo on every run instead.
- **Rule going forward:** Verify what actually resets a platform auto-disable
  timer before proposing a fix for it — don't infer from trigger-type names.

### 2026-08-14 — mirror-workflow-repo-access

- **Trigger:** misread requirements
- **What happened:** Attempted `gh auth status`, raw `curl` to the GitHub
  API, and an MCP connector search to get access to
  `kaywoz/github-mirror-controller`. All failed: invalid token, blocked API
  path, session scoped to PR-review operations only, no GitHub connector
  installed.
- **Root cause:** Didn't check available access scope before attempting
  multiple tool paths in sequence.
- **Fix applied:** Asked kaywoz to paste file contents / apply changes
  manually instead.
- **Rule going forward:** Check GitHub access scope once at task start; if
  the repo isn't reachable, ask for file contents immediately rather than
  cycling through CLI/API/connector attempts.

### 2026-08-14 — mirror-workflow-refs-pull

- **Trigger:** CI failed
- **What happened:** `git push --mirror` to Codeberg failed with
  `deny updating a hidden ref` on `refs/pull/N/head` refs.
- **Root cause:** `git clone --mirror` from GitHub pulls in
  `refs/pull/*` (GitHub's own read-only PR refs); Codeberg rejects direct
  writes to that ref namespace, and `push --mirror` pushes all refs in one
  command.
- **Fix applied:** Replaced with explicit refspecs —
  `git push --prune ... '+refs/heads/*:refs/heads/*' '+refs/tags/*:refs/tags/*'`
  — mirroring branches/tags only.
- **Rule going forward:** Never use `git push --mirror` against a host that
  restricts `refs/pull/*` or similar reserved namespaces; push explicit
  `refs/heads/*` and `refs/tags/*` refspecs instead.
