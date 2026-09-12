# By the Book

One change, one fixed path, a gate before every artifact.

```
read back -> Go? -> research -> (draft issue -> Approve?)+ -> create issue -> branch
  -> prior art -> plan -> Go? -> ( (failing test -> passing code -> tests and format)+ -> present -> Approve? -> commit )+
  -> Review? -> (findings -> commit loop)* -> (draft PR -> Approve?)+ -> open PR
```

You describe the change and confirm the read-back with Go?. The skill researches the code, drafts an issue and waits for your approval, files it, branches from the issue number, looks at prior art, and then works in a test-first loop where every commit is presented and approved before it lands. Before the pull request it offers an adversarial review by a read-only agent that looks for untested edges, inputs chosen to corrupt state, formatting drift, and AI tells in the prose. The pull request is drafted, approved, and opened.

Every question it asks is one of three words, always the same word at the same moment: Go? before research and again before the first test is written, Approve? before anything is created, and Review? after the last commit. What a question gates goes in chat first, and the question holds nothing but the word and its two answers. Nothing is created, committed, or opened before you say so, and every issue, commit message, and pull request body is written the way you would write it.

## Usage

```
/by-the-book Add a max_length parameter to slugify that cuts at a word boundary
```

Flags go anywhere in the argument:

| Flag | Effect |
|---|---|
| `--worktree` | Work in an isolated worktree instead of switching branches |
| `--branch=kind` | Name the branch `feat/<slug>` or `fix/<slug>` instead of `issue-<N>-<slug>` |
| `--no-body` | Commit messages are subject only |
| `--review`, `--review=each`, `--no-review` | Run the review before the PR, after every commit too, or never; the default is to ask once |
| `--depends=N[,N]` | Pull requests this one depends on, for the footer |

## What the artifacts look like

An issue title states the problem, never the fix. The body keeps the problem, any possible fixes, and any questions to the maintainers in separate parts, and a part with nothing in it is absent. Questions read like one person talking to another.

A commit subject is imperative, lowercase, and free of articles, and says what the commit does to the repository rather than what the code now does, unless the repository's history shows another convention. The body says why and what it rejects.

A pull request subject is imperative in sentence case with the issue number at the end. The body opens with `Closes #N.`, then one or two sentences saying what was done, then how and why in plain paragraphs, then how it was tested. A footer with `Depends on` and `Stacked:` appears only when it applies.

When the repository ships issue or pull request templates, those win.

A correction you make at any gate becomes the convention for the rest of the session.

## Install

```
/plugin marketplace add ero-qt/by-the-book
/plugin install by-the-book@by-the-book
```

Or drop `skills/by-the-book/` into `~/.claude/skills/`. The skill is manual-only; it never triggers on its own.

It works alone. When [laconic](https://github.com/ero-qt/laconic) or the superpowers plugin is installed, the skill uses them for prose, test-driven development, worktrees, and reviewer dispatch, and every step is complete without them.

MIT.
