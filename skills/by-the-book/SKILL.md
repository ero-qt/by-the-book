---
name: by-the-book
description: Use when the user asks to take a change from issue to pull request with approval gates.
disable-model-invocation: true
---

# By the Book

One change, one fixed path, a gate before every artifact. Companions are used when the skill list shows them and every step below is complete without them: laconic, superpowers:test-driven-development, superpowers:using-git-worktrees, superpowers:requesting-code-review. Never call a skill that is not listed.

`$ARGUMENTS` is the change, in the user's words, plus any of these flags:

| Flag | Effect | Default |
|---|---|---|
| `--worktree` | Work in an isolated worktree | plain branch |
| `--branch=kind` | Name the branch `feat/<slug>` or `fix/<slug>` | `issue-<N>-<slug>` |
| `--no-body` | Commit messages are subject only | subject and body |
| `--review` / `--review=each` / `--no-review` | Run the review before the PR / after every commit too / never | ask once |

## Order

```
research -> (draft issue -> gate)+ -> create issue -> branch -> prior art
  -> ( (failing test -> passing code -> tests and format)+ -> present -> gate -> commit )+
  -> review? -> (findings -> commit loop)* -> (draft PR -> gate)+ -> open PR
```

A gate is one AskUserQuestion holding the full draft, with the options Approve and Change. On Change, redraft from the note and gate again. Nothing is created, committed, or opened before its gate returns Approve. Bodies reach `gh` through `--body-file -` on stdin, never a file in the repo. A `create` runs once, after its gate, never as a probe.

## Voice

Everything a human will read is written as the user would write it. The default is factual and impersonal: "Added", "Fixed", "Today X returns Y." First person is allowed in two places and nowhere else, and neither is required: an opinionated decision ("I chose to return an empty slug because") and a question to the maintainers, which reads like one person talking to another ("Should this be done at all? I looked into it and the URL layer already truncates."). A body with no decision to defend and no open question has no "I", and that is the common case.

Markdown stays light. `##` is the largest heading, `###` is rare and only inside an `##`, a body under 150 words has no headings, and bold is never a heading. Paragraphs stay short. A list may follow a sentence and a colon when the items are parallel, numbered when order matters, and a comma run long enough that a reader loses their place becomes one. A wall of headings and bullets reads as generated, and so does a wall of prose where a list was the natural shape.

When the repo ships a template (`.github/ISSUE_TEMPLATE/`, `.github/PULL_REQUEST_TEMPLATE.md`, `CONTRIBUTING.md`), its sections and order win. Fill it, do not restate it.

## Steps

**1. Research.** Read the code the change touches and its tests, `gh issue list` for overlap, `git log --oneline -20 -- <paths>` for history, and the template locations above. This is knowledge for the drafts, not a message.

**2. Issue.** Draft subject and body, then gate. Without a template the body keeps up to three parts apart, and a part with nothing in it is absent:

1. The problem or the missing feature: what happens today, as fact, and what is needed. No fix here. For a bug, the evidence: a permalink to the lines at fault (`gh browse -n <path>:<line>`), the triggering input, and the output against what was expected, in a code block when the values need one.
2. Possible fixes, when there are any, as a paragraph or a short list. A reader must be able to accept the problem and reject every fix.
3. Questions to the maintainers, when there are any: how they want it done, whether it is wanted, what else to address. Written to a person, with what was looked into where that helps them answer.

Done criteria go in as a short list only when the outcome is not obvious from the problem. On Approve, `gh issue create` and keep the number.

**3. Branch.** From the default branch: `git switch -c issue-<N>-<slug>` or the `--branch=kind` form. With `--worktree`: `git worktree add ../<repo>-issue-<N> -b issue-<N>-<slug>` and work there.

**4. Prior art.** How this has been solved before: a sibling in the repo, the library's docs, one or two known implementations. Three sources at most. This shapes the tests and is reported only if it changed the design.

**5. Commit loop.** One commit per coherent change:

1. Write one failing test. Run it. Confirm it fails for the intended reason.
2. Write the least code that passes it. Run the suite.
3. Run the repo's formatter if one is configured. Add none.
4. Repeat until the commit's behavior is covered.
5. Present: an opening line saying what the commit does, a table of files and what changed, the test command and its last line, and the draft message. Gate.
6. On Approve, commit with that message exactly.

The test and the code are never written in the same tool call. A test that passes on its first run is deleted and rewritten to fail. A code comment may say why one approach over another when that is not obvious. A docstring says what the thing does now and never what was rejected, removed, or not done, unless the item is deliberately obsolete and marked so.

Message: imperative subject, 50 characters or fewer, in the repo's convention from `git log`. Body unless `--no-body`: why, and what it rejects, wrapped at 72, referencing `#<N>`. No trailers, no sign-offs.

**6. Review.** After the last commit, ask once: "Run an adversarial review before the PR?" Yes / No. The flags answer it. The reviewer is one read-only Agent with the issue text, the diff against the base, and the test command. It may run tests and read anything; it edits nothing. It looks for:

- each done criterion without a test that proves it
- bugs in the diff, with a reproducing input
- untested edges: empty, None, zero, negative, huge, unicode, boundaries, and combinations chosen to corrupt state, with proof each is handled or a demonstration it is not
- formatting and line length against the repo's config
- tells in comments, docstrings, messages, and drafts

It returns a verdict line ("Ship" or "N findings block") and a table of severity, location, finding, and the input that shows it. Blocking findings go back through the commit loop, test first. Prose findings are fixed in the drafts.

**7. Pull request.** Draft subject and body, then gate. Without a template the body is `Closes #<N>.` on the first line, then one or two sentences saying what was done with no heading, then how and why as plain paragraphs (`## How` and `## Why` only when each runs past a paragraph), the one opinionated decision under why, then the test command and what the new tests cover.

**8. Report.** The end-of-turn message opens with the PR link and what shipped, then a table of commits, the state, and one Next line.

## Scope

The diff holds only what the issue needs. No `.gitignore`, formatter config, README touch-ups, or drive-by refactors unless the issue says so. Anything else worth fixing becomes a second issue draft at the end, unfiled.

## Stop and redo

- Test and code in one tool call, or the suite run once at the end
- A branch before the issue number exists
- A `gh ... create` with a placeholder body, or a body file left in the tree
- A fix in the same paragraph as the problem
- An "I" or a question added because the shape had a slot for one
- A docstring that says what the code used to do or what was decided against
- A PR body that does not start with `Closes #N.`, or whose summary carries a heading
- A draft that ignores a template the repo ships
- A file in the diff the issue never mentioned
- A trailer, footer, or emoji in a commit message
