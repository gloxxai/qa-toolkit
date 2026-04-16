---
name: bug-triage
description: Evaluate and categorize a reported bug by severity, impact, and type, then produce a filing-ready triage report. Use when there's an incoming bug report and you need to decide how urgent it is, who owns it, and what to do about it before it lands in the tracker.
user-invocable: true
argument-hint: "[bug description or ticket link]"
---

## MANDATORY PREPARATION

Before triaging, gather:

1. **Reproduction status** — has anyone reproduced it, or is it a report-only?
2. **Environment** — production, staging, specific release, specific device/browser?
3. **Scope of impact** — one user, one cohort, all users?
4. **Time pressure** — is this blocking a release, a demo, a customer?
5. **Existing context** — is there an open ticket already? A related incident?

Never triage without these five answers. Push back on the reporter if any are missing — an under-specified bug produces an under-specified triage.

---

Triaging a bug means deciding three things: **how bad is it**, **who owns it**, and **what happens next**. The output is a short report that pastes into Jira, Linear, GitHub Issues, or whatever tracker is in use.

## Phase 1 — Severity

Severity is about *consequence*, not urgency:

| Severity | Meaning | Examples |
|---|---|---|
| **S1 — Critical** | Data loss, security breach, system down, or financial impact | Auth bypass, payment fails silently, all users see a blank screen |
| **S2 — Major** | Core feature broken for a meaningful cohort, no viable workaround | Checkout works but coupon codes silently fail, search returns empty for 20% of queries |
| **S3 — Moderate** | Feature broken with a workaround, or broken for a small cohort | Upload fails for files >50MB, tooltip appears in the wrong place |
| **S4 — Minor** | Cosmetic, minor UX annoyance, or rare edge case | Button misaligned by 2px, typo in a secondary CTA, broken on IE11 |

**Rule of thumb:** if a customer would churn over it, it's S1 or S2. If a customer would DM the founder about it, S2 or S3. If only internal QA would notice, S3 or S4.

## Phase 2 — Impact

Impact is about *reach*, not severity:

- **How many users are affected?** (All / cohort / specific segment / one reporter)
- **How often do they hit it?** (Every session / every flow / rare edge)
- **What do they do instead?** (Workaround exists / forced to abandon flow)
- **Is it getting worse?** (Static / growing as more users hit the affected path)

A low-severity bug with massive impact (every user sees a minor glitch) may deserve faster attention than a high-severity bug with tiny impact (S1 edge case hitting 3 users).

## Phase 3 — Type

Classify the bug so the right team sees it:

| Type | What it is | Lands with |
|---|---|---|
| **Functional** | Something doesn't work as specified | Feature owner |
| **Regression** | Something that used to work, doesn't anymore | The team that shipped the last relevant change |
| **Performance** | Slow, laggy, memory hog | Platform / infra |
| **Visual / UX** | Looks wrong, confusing, inconsistent | Design + feature owner |
| **Accessibility** | WCAG violation, screen reader issue, keyboard nav gap | Whoever owns the affected component |
| **Data** | Wrong values stored, missing records, corruption | Data / backend |
| **Integration** | Third-party service, API contract | Whoever owns that integration |
| **Test infrastructure** | Flaky test, CI failure, false positive | QA / platform |

If multiple types apply, pick the dominant one and note the others.

## Phase 4 — Recommended action

Each severity maps to a default path:

- **S1** → Page the on-call, file as a blocker, begin incident response
- **S2** → File with "blocker for next release" label, notify feature owner same day
- **S3** → File in backlog with impact data, let owner prioritize
- **S4** → File or not (ask: is this worth capacity?) — sometimes the right answer is "skip, not worth a ticket"

Override the default when:
- Time-boxed pressure (demo tomorrow, release at 3pm) bumps priority up
- Known in-progress fix bumps priority down (already in a PR)
- Root cause is shared with another open bug → link them, don't file twice

## The triage report template

Output this structure:

```
## Triage — {one-line summary}

**Severity:** S{1-4} — {rationale}
**Impact:** {reach} · {frequency} · {workaround?}
**Type:** {type} (primary), {secondary types if any}
**Reproduced:** Yes / No / Partial · {env}

**Steps to reproduce:**
1. ...
2. ...

**Expected:** ...
**Actual:** ...

**Related:** {linked tickets, incidents, PRs}

**Recommended action:** {default path, with any override rationale}
**Owner:** {team or individual}
**Suggested labels:** {tracker-appropriate labels}
```

## What you do NOT do

- You don't fix the bug — triage ends at filing
- You don't assign a ticket to a specific person without confirming ownership (team-level assignment is fine)
- You don't triage bugs you can't reproduce without flagging "not reproduced" explicitly
- You don't guess at root cause in the triage — that's for the engineer who picks it up
- You don't inflate severity to get attention — the team learns to discount you

## Common failure modes

- **"S1 everything"** — over-triaging everything as critical dulls the system. Save S1 for actual blockers.
- **Missing repro steps** — a bug without steps to reproduce is a wish, not a bug. Ask the reporter before filing.
- **Triaging without searching for dupes** — duplicate tickets rot trackers. Always search first.
- **Conflating severity with impact** — they're different axes. A low-severity bug with huge impact deserves attention. A high-severity edge case maybe doesn't.

Good triage is the bridge between "something's wrong" and "we know what to do about it." Be the engineer the team trusts to be accurate — not the one who cries wolf.
