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
| `--review` / `--review=each` / `--no-review` | Answer `Review?` yes / yes and after every commit / no | ask |
| `--depends=N[,N]` | PRs this one depends on | detected from the base branch |

## Order

```
read back -> Go? -> research -> (draft issue -> Approve?)+ -> create issue -> branch
  -> prior art -> plan -> Go? -> ( (failing test -> passing code -> tests and format)+ -> present -> Approve? -> commit )+
  -> Review? -> (findings -> commit loop)* -> (draft PR -> Approve?)+ -> open PR
```

Every question to the user is one of three words, the same word at the same moment, written as the last line of the turn. The question tool is not used: it hides what sits above it, and a dismissed prompt leaves the step with no answer. The user's next message is the answer:

| Prompt | Moment | Answers |
|---|---|---|
| `Go?` | after the read-back, before research; and after prior art, once the plan is in chat | Go / Change |
| `Approve?` | a gate, once the full draft of the issue, commit, or PR is in chat | Approve / Change |
| `Review?` | after the last commit, once the reviewer's model and effort are in chat | Yes / No |

Whatever the word gates goes in chat above it, in the same turn: the read-back, the plan, the draft, the file table, the test line. A draft is shown verbatim in a code block, never described. The last line holds the word and nothing else. On Change, redraft from the note and ask again. Nothing is created, committed, or opened before its `Approve?` returns Approve. A correction at any prompt is a convention from then on: every later draft of that kind follows it unasked, it is worth a memory note when the harness keeps one, and when the corrected form came from a skill the user owns, that skill is edited in the same pass. Bodies reach `gh` through `--body-file -` on stdin, never a file in the repo. A `create` runs once, after its gate, never as a probe.

A question from the user is answered, not acted on. It gets one of three replies: the reason, when it holds; the reason and a softer alternative, when prior art or the user's likely preference points another way; or a plain concession that the step overreached, broke something, or put something in the wrong place. Nothing changes until the user says Change or Approve. The tone of the question changes none of this, and "you're right to be frustrated" or "you're right to push back" is never the reply.

## Voice

Everything a human will read is written as the user would write it: factual, in words a 4th grader reads, as a comment is. A body's first sentence says what the whole change does, in the present tense, without much detail. The sentences after it are short and plain, in whatever tense the fact needs, and they may say what was there before and what was removed or fixed. Every noun is written where it is used: a pronoun or a possessive standing in for a noun from the sentence before is compression. The plain word wins over the one picked for effect. First person is allowed in two places and nowhere else, and neither is required: an opinionated decision, and a question to the maintainers, which reads like one person talking to another rather than a checklist item. A body with no decision to defend and no open question has no "I".

An issue or PR body is written for a reader who was not in the conversation. It carries nothing the conversation alone supplies: not the reason the user gave for wanting the change, not the thing they pointed at to motivate it, not the words they used to ask. What the change does and why it is right are stated from the code and the repo. A sentence that would puzzle a maintainer who never saw the chat or this skill is a tell, and so is a word that only means something inside this skill: "gate", "redo", "tell", "prompt", "read-back".

A markdown paragraph is one line however long; wrapping prose at a column is a tell. Commit bodies are the exception and wrap at 72. `##` is the largest heading, `###` is rare and only inside an `##`, a body under 150 words has no headings, and bold is never a heading. Paragraphs stay short. A list may follow a sentence and a colon when the items are parallel, numbered when order matters, and a comma run long enough that a reader loses their place becomes one. A wall of headings and bullets reads as generated, and so does a wall of prose where a list was the natural shape.

When the repo ships a template (`.github/ISSUE_TEMPLATE/`, `.github/PULL_REQUEST_TEMPLATE.md`, `CONTRIBUTING.md`), its sections and order win. Fill it, do not restate it.

## Steps

**0. Read back.** One line in chat with the change as understood and the flags in effect. `Go?`

**1. Research.** Read the code the change touches and its tests, `gh issue list` for overlap, `git log --oneline -20 -- <paths>` for history, and the template locations above. This is knowledge for the drafts, not a message.

**2. Issue.** Post the draft subject and body in chat. `Approve?` The subject states the problem as a fact about the code, never the fix. Without a template the body keeps up to three parts apart, and a part with nothing in it is absent:

1. The problem or the missing feature: what happens today, as fact, and what is needed. No fix here. For a bug, the evidence: a permalink to the lines at fault (`gh browse -n <path>:<line>`), the triggering input, and the output against what was expected, in a code block when the values need one.
2. Possible fixes, when there are any, as a paragraph or a short list. A reader must be able to accept the problem and reject every fix.
3. Questions to the maintainers, when there are any: how they want it done, whether it is wanted, what else to address. Written to a person, with what was looked into where that helps them answer.

Done criteria go in as a short list only when the outcome is not obvious from the problem. On Approve, `gh issue create` and keep the number.

**3. Branch.** From the default branch: `git switch -c issue-<N>-<slug>` or the `--branch=kind` form. With `--worktree`: `git worktree add ../<repo>-issue-<N> -b issue-<N>-<slug>` and work there.

**4. Prior art.** How this has been solved before: a sibling in the repo, the library's docs, one or two known implementations. Three sources at most. This shapes the plan, posted in chat as a numbered list with one commit per item: one line each, with the behavior it covers and its first failing test. `Go?`

**5. Commit loop.** One commit per coherent change:

1. Write one failing test. Run it. Confirm it fails for the intended reason.
2. Write the least code that passes it. Run the suite.
3. Run the repo's formatter if one is configured. Add none.
4. Repeat until the commit's behavior is covered.
5. Stage the files. Nothing is committed yet, and nothing said in chat calls it committed.
6. Present in chat, in the shape below, with the draft message verbatim. `Approve?`
7. On Approve, commit with that message exactly.

~~~
**<what the commit does, one line>**

| File | Change |
|---|---|
| <path> | <what changed in it> |

`<test command>` → `<its last line>`

```
<draft message>
```
~~~

The test and the code are never written in the same tool call. A test that passes on its first run is deleted and rewritten to fail. A code comment may say why one approach over another when that is not obvious. A docstring says what the thing does now and never what was rejected, removed, or not done, unless the item is deliberately obsolete and marked so.

Message: imperative subject, lowercase, no articles, 50 characters or fewer. The subject says what the commit does to the repo, never what the code now does: its verb is one a diff can show, such as add, remove, move, rename, split, replace, fix, or bump, and a verb that names what the code does at run time is redone. When `git log` shows the repo does it another way, the repo wins. Body unless `--no-body`: why, and what it rejects, wrapped at 72, referencing `#<N>`. No trailers, no sign-offs.

**6. Review.** After the last commit, one line in chat naming the model and effort the reviewer will run at, then `Review?`. The flags answer it. The reviewer is one read-only Agent at that model and effort, with the issue text, the diff against the base, and the test command. It may run tests and read anything; it edits nothing. It looks for:

- each done criterion without a test that proves it
- bugs in the diff, with a reproducing input
- untested edges: empty, None, zero, negative, huge, unicode, boundaries, and combinations chosen to corrupt state, with proof each is handled or a demonstration it is not
- formatting and line length against the repo's config
- tells in comments, docstrings, messages, and drafts

It returns a verdict line and a table of severity, location, finding, and the input that shows it. Blocking findings go back through the commit loop, test first. Prose findings are fixed in the drafts.

**7. Pull request.** Post the draft subject and body in chat. `Approve?` The subject is imperative in sentence case with the issue number in parentheses at the end. Without a template the body is `Closes #<N>.` on the first line, then one paragraph with no heading whose first sentence says what the whole PR does and which may go on from there, then the smaller changes each on its own line or paragraph, then how and why as plain paragraphs (`## How` and `## Why` only when each runs past a paragraph), the one opinionated decision under why, then the test command and what the new tests cover. Lists, tables, code blocks, and horizontal rules go wherever they fit. Only when it applies, a horizontal rule and then the relations one per line: `Depends on #<M>.` and for a stack `Stacked:` with a numbered list of the PRs in order. `--depends` fills that; without it, a base branch other than the default means this PR depends on that branch's PR, and `gh pr create` gets `--base` set to it.

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
- A PR body that does not start with `Closes #N.`, whose first sentence does not say what the whole PR does, whose summary carries a heading, or that uses a word that only means something inside this skill
- A draft that ignores a template the repo ships
- A file in the diff the issue never mentioned
- A trailer, footer, or emoji in a commit message
- A commit subject with a capital or an article, a commit subject whose verb names what the code does at run time instead of what the diff does, a PR subject without its issue number, an issue title that names a fix
- A change made in reply to a question instead of an answer
- A body that leans on something said in the conversation
- A question asked through the question tool, a last line holding more than its one word, or a form the user corrected at an earlier question
- A presentation that describes the draft, the diff, or the test result instead of showing them
- A presentation in the first person, or one that calls the change committed before `Approve?` returns
- A plan given as a paragraph instead of a numbered list
