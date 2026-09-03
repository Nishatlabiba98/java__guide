# 15 — Git and Code Review

[00 — Setup](../00-setup/README.md) covered the git mechanics you need to get started (clone/branch/commit/push, SSH, merge conflicts, rebase, stash, patches). This section is about the *process* around those mechanics — how TapRoot actually uses git day to day, and how to read and give a code review.

## Branching strategy

A typical trunk-based flow: `main` is always deployable, all work happens on short-lived feature branches cut from it, merged back via pull request:

```
git checkout main
git pull
git checkout -b your-name/short-description
```

Keep branches small and short-lived — a branch that lives for weeks accumulates conflicts and gets harder to review. If a feature is genuinely big, break it into smaller PRs that land incrementally behind a flag if needed, rather than one enormous review.

## Opening a pull request

- **Title**: describes the change, not the ticket number alone — "Fix null pointer on empty cart checkout," not "Fix bug."
- **Description**: what changed and *why* — the diff already shows *what*; reviewers need the why to judge if the approach is right.
- **Keep it reviewable**: a 40-line PR gets a careful review; a 2,000-line PR gets a rubber stamp because nobody can hold it all in their head. Split large changes where you can.

## PR etiquette

- Respond to every comment — even a quick "done" or "good catch, fixed" — so the reviewer knows you saw it. Leaving comments unaddressed and silently re-requesting review is a common friction point.
- Don't take feedback personally, and don't give it personally — review the code, not the person ("this could race under concurrent access" not "you forgot to think about concurrency").
- If you disagree with a comment, say why instead of silently ignoring it or silently complying — reviewers are often wrong too, and a short discussion is cheaper than a bad merge either direction.

## Reading a diff

A diff shows three things: what was removed (`-`), what was added (`+`), and surrounding unchanged context. When reviewing:

1. Read the PR description first — know what it's trying to do before judging how.
2. Look for correctness first (does this actually do what it claims, are there edge cases missed), then design/readability, then nitpicks (naming, formatting) last.
3. If a change is hard to review because it mixes a refactor with a behavior change, say so — those should usually be separate commits or separate PRs.

## What to actually look for in review

- **Correctness**: does it handle the edge cases (empty input, null, concurrent access — everything from sections 05 and 11)?
- **Tests**: does new behavior have a test (section 12)? Does an existing test's intent still make sense after this change?
- **Naming and clarity**: would someone unfamiliar with this change understand it from the code alone?
- **Scope**: does the diff only do what the PR claims, or does it quietly refactor unrelated things too (making the real change harder to spot)?

## Exercise: review your own PR

Before your first real PR, do a self-review: open the diff as if you were reviewing someone else's code. Check for a leftover debug `System.out.println`, an unused import, a variable name that made sense while writing but not while reading, and confirm every new method has at least one test from section 12. This catches the easy stuff before a teammate has to.

## Checklist

- [ ] Comfortable branching off `main`, committing in small logical chunks, and opening a PR
- [ ] Wrote a PR description explaining *why*, not just restating the diff
- [ ] Know the order to review in: correctness → tests → clarity → nitpicks
- [ ] Responded to every review comment on a real PR, not just the ones you agreed with
- [ ] Did a self-review pass before requesting review from someone else
