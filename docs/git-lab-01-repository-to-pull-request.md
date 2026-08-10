# Git Lab 1 — From Repository Creation to Pull Request

**Project Alpha · Lesson 1**

A worked lab following one real repository through its first day: created empty,
edited in the browser, moved to local development, branched, and proposed back
via a pull request. Every command, mistake and correction below is taken from the
actual history of `sarahjdevereaux.github.io`, not invented for teaching.

---

## Objectives

By the end of this lab you should be able to:

1. Explain why git stores **snapshots** rather than deltas, and what follows from that.
2. Move a file between the three states: working tree → staging area → repository.
3. Create a branch and explain what a branch actually *is*.
4. Push a branch and explain why pushing is not the same as merging.
5. Explain why a "pull request" is called that, and what it does and does not move.
6. Write a commit message that survives contact with your future self.

**Prerequisites:** a GitHub account, git installed, and a text editor. A GUI
client (GitKraken, GitHub Desktop) is optional — every step below is described in
terms of what git does, so any client works.

---

## Part 0 — The model, before any commands

Three ideas explain most of git's surprising behaviour.

### Git stores snapshots, not deltas

Many version control systems (SVN, CVS, RCS) store a file's history as a chain of
differences: the original, plus a list of edits. Git does not. **Every commit
references a complete snapshot of the project tree.**

Unchanged files aren't duplicated. Git addresses content by its hash, so a file
that didn't change between commits is simply the *same object*, referenced twice.
Identical content is stored once, ever, no matter how many commits or branches
contain it.

> **Nuance worth knowing.** Git *does* use deltas — but only in the storage layer,
> inside packfiles. Those deltas are chosen by a similarity heuristic across all
> objects, not chained along version history. Git will delta a file against an
> unrelated file on another branch if that compresses better. The history-shaped
> chain other systems depend on structurally is, in git, an incidental compression
> outcome.

### Therefore: diffs are computed, never stored

Nothing in `.git/` records "these lines changed." A diff is calculated on demand
by comparing two snapshots. This is why git can diff any two commits cheaply,
even across unrelated branches.

It also means **renames are not recorded — they are inferred.** Demonstrate it on
any commit that moved a file:

```bash
git show --stat <commit>              # shows: old/path.html => new/path.html
git show --stat --no-renames <commit> # shows: a delete and an add
```

The second is what is actually stored. The `=>` is git running similarity
detection at display time and deciding two blobs are close enough to call a
rename. `git mv` records nothing special — it is `mv` plus staging.

> **Practical consequence.** Rename detection uses a similarity threshold (~50%).
> If you move a file *and* substantially rewrite it in one commit, git loses the
> thread and the file's history appears to start fresh at the new path.
> **Habit: rename in one commit, rewrite in the next.**

### The three states

A change lives in one of three places, and moves in one direction:

```
  working tree  ──git add──▶  staging area  ──git commit──▶  repository
  (your edits)                (what's next)                  (permanent)
```

The staging area is the part people skip past. It exists so you can commit *some*
of your changes — a coherent unit — rather than everything you happen to have
touched.

---

## Part 1 — Create the repository

For a GitHub Pages site the repository name is not cosmetic. A repo named
`<username>.github.io` is published at `https://<username>.github.io`.

1. On GitHub: **New repository**, named `<your-username>.github.io`, public,
   initialised with a README.
2. That initialisation is itself the first commit. In the reference history it is
   `26c8e67 Create README.md`.

**Checkpoint:** the repository has one commit and one branch, `main`.

> **What "Pages is production" means.** GitHub Pages serves the default branch.
> There is no staging environment. Every file in the repo is publicly reachable —
> including ones nothing links to. A stray `index-old.html` is a live page.

---

## Part 2 — Edit in the browser (and feel the limits)

Editing files through GitHub's web interface commits directly to `main`. It works,
and it is how the reference repo started (`e700145`, `ac36b1d`, `737e437`).

Two things become apparent quickly:

- Every save is a commit straight to production.
- The default message — `Update index.html` — describes the file, not the change.
  Three commits with that message tell you nothing about what happened.

This is the motivation for everything that follows.

---

## Part 3 — Go local

```bash
git clone https://github.com/<username>/<username>.github.io.git
cd <username>.github.io
```

Preview over HTTP rather than double-clicking the file, so paths behave the way
they will in production:

```bash
python -m http.server 8000
```

**Checkpoint:** `git status` reports a clean tree on `main`.

---

## Part 4 — The commit cycle

Make a change, then inspect before committing:

```bash
git status          # which files are in which state
git diff            # unstaged changes, line by line
git add <file>      # move to staging
git diff --staged   # verify exactly what is about to be committed
git commit -m "..."
```

`git diff --staged` is the habit worth building. It answers "what am I actually
about to record?" — the question a bad commit message usually fails to.

### Writing the message

The rule of thumb: **if you cannot describe the commit in one line without lying
by omission, it should have been more than one commit.**

Two real messages from the reference history, and why they fall short:

| Message | What it actually contained |
|---|---|
| `Improved layout margins and image size` | Also moved a file into `projects/`, deleted two files, added a favicon, and rewrote all page metadata and the README |
| `added contact section to navbar` | 98 lines across 3 files — an entire section plus a new stylesheet component. The navbar was one line of it |

Neither is *wrong* about what it names. Both are radically incomplete.

> **A useful framing:** a pull request shows the diff *and* the commit history.
> If the PR is the patient's chart, commit messages are the chart notes. A note
> saying "navbar tweak" against a record showing three files rewritten is a
> documentation failure — it misleads the next reader, who is usually you.

---

## Part 5 — Branch

```bash
git switch -c basic-info-additions
```

A branch is not a copy of anything. **It is a movable pointer to a commit.**
Creating one is nearly free — it writes a file containing a hash.

Naming: lowercase, hyphenated, describing intent. Spaces are not permitted.
`basic-info-additions` is good; `info update` is not a valid branch name.

> **The mistake to avoid — branch *before* you work, not after.**
> In the reference history, `d4582f8` was committed straight to `main`, and the
> branch was created afterwards. This is the single easiest habit to get wrong,
> because you only think of branching once you are already editing.

> **The other trap.** Uncommitted changes do not belong to a branch at all. They
> float above whichever branch you are on and follow you when you switch. A branch
> only begins protecting work once you **commit** to it. Creating a branch and
> then editing is not enough — verify with `git log --oneline -1` on both branches:
> if they show the same hash, your branch contains nothing yet.

Commit your work to the branch before continuing.

---

## Part 6 — Push

```bash
git push -u origin basic-info-additions
```

This is the step that first creates the branch **on GitHub**. Until now it existed
only on your machine. `-u` sets the upstream so future pushes need no arguments.

Verify what is and isn't published:

```bash
git branch -avv
```

A branch with no `[origin/...]` marker exists locally only. This single command
answers "has GitHub seen this?" — and it is how you check whether a pull request
is even possible yet.

**Critically: `main` has not moved.** You have published a proposal. The live site
is unchanged.

---

## Part 7 — The pull request

On GitHub: open a pull request from your branch into `main`.

### Why it is called that

The name confuses everyone, and the confusion is worth resolving properly rather
than memorising.

**`push` and `pull request` are not opposites.** They belong to different
categories:

- **`push` is plumbing.** It moves bytes. Commits that existed only on your disk now also exist on a server.
- **A pull request is paperwork.** It moves *nothing*. Opening one transfers zero commits — GitHub already has them.

The verb describes **the action you are asking someone else to perform**. Git was
built for Linux kernel development, where contributors had no write access to the
maintainer's repository. The workflow was: publish your commits somewhere, then
email the maintainer — *"I've done work at this URL, please pull it."* You were
literally requesting they run `git pull`.

That command still ships with git today:

```bash
git request-pull -h
```

It predates GitHub, which built a web page around it. The name is a fossil of a
world of separate machines and email. GitLab's **merge request** is the honest
modern name.

It is shaped like a *friend request*: named for what you want the other party to
do, sent by you, completed by them.

### On a solo repository

You are requesting this from yourself, which feels absurd — and the workflow was
indeed designed for a maintainer/contributor split you don't have. Its value here
is different: **it is a diff you are forcing yourself to read before it goes live.**

Nothing technically prevents you pushing straight to `main`. The pull request is
voluntary discipline, not a gate — unless you enable branch protection.

### While the PR is open

- Pushing more commits to the branch **updates the open PR automatically**. You don't open a new one to address feedback.
- If `main` moves after you branched, your changes may no longer apply cleanly. That is a **merge conflict** — not corruption, just git declining to guess which version wins when the same lines were edited twice.

---

## Part 8 — Merge

Merging is the moment the change enters `main`. On a Pages repository, **that is
the deploy.** Expect 30–60 seconds before it is live, and hard-refresh
(`Ctrl+F5`) if the browser serves you a cached copy.

Then bring your local copy back in line:

```bash
git switch main
git pull
```

**Checkpoint:** `git log --oneline --graph --all` shows your branch merged into
`main`, and the change is visible at your live URL.

---

## Field notes — real errors from this repository

Errors are the lab. These all actually happened.

1. **"I did a pull request" — no PR existed.** A branch had been created locally and never pushed. Branch ≠ pull request; the branch is git, the PR is GitHub. *Diagnose with `git branch -avv` and look for the `[origin/...]` marker.*

2. **A branch with no commits.** The branch and `main` pointed at the same hash while the actual edit sat uncommitted in the working tree.

3. **A PR larger than expected.** Because `main` had unpushed commits, a branch descended from it carried those too — so the PR contained three commits, not one. *Push `main` first if you want a PR that reviews exactly one idea.*

4. **A stranded branch.** `html-edits` kept two commits that were never merged and never pushed; PR #1 merged only the commit before them. Branches outlive their usefulness quietly.

5. **A file in the wrong directory broke two things at once.** `project-alpha.html` sat at the repo root but linked its stylesheets as `../css/`. Every link to it 404'd, and it would have rendered unstyled. *When a link breaks, suspect the file's location as readily as the link's text.*

6. **Case-sensitivity trap.** Windows filesystems are case-insensitive; the Linux servers behind GitHub Pages are not. A path that is merely mis-cased works locally and 404s in production.

---

## Checkpoint questions

1. You ran `git push`. Is your change live on the site? Why or why not?
2. Your branch and `main` show the same commit hash. What does that tell you?
3. Why does `git show --stat` display a rename that isn't stored anywhere?
4. You open a PR and then notice a typo. Do you close it and open a new one?
5. Why is `Update index.html` a poor commit message even though it is accurate?
6. Your colleague uses GitLab and says "merge request". Are they doing something different?

---

## Glossary

| Term | Meaning |
|---|---|
| **working tree** | The files as they currently exist on disk |
| **staging area** (index) | The set of changes selected for the next commit |
| **commit** | A permanent snapshot of the whole tree, plus author, message and parent |
| **branch** | A movable pointer to a commit — not a copy |
| **upstream** | The remote branch a local branch tracks |
| **push** | Send commits to a remote. Moves data |
| **pull request** | A request that someone merge a published branch. Moves no data |
| **merge** | Combine a branch's history into another. On Pages, this is the deploy |
| **merge conflict** | Git declining to guess when the same lines changed twice |
| **packfile** | Compressed object storage where git *does* use deltas |

---

## Next lab

**Git Lab 2 — Hardening the New Repo.**

A GitHub Pages repository is two things at once: the workshop where the work
happens, and the shopfront the internet sees. By default there is no separation —
every file committed is publicly reachable, and with Jekyll running by default,
markdown files are even converted into web pages. Lab 1's own work report would be
published as a page on the live site.

Lab 2 covers keeping the shopfront presenting finished work while the workshop
stays private, across the three layers that are easy to conflate:

| Layer | File | Question it answers |
|---|---|---|
| Tracking | `.gitignore` | What does git record at all? |
| Handling | `.gitattributes` | How does git treat what it tracks (line endings, binaries, diffs)? |
| Publishing | `_config.yml` / `.nojekyll` | What does GitHub Pages actually serve? |

Also covered: why `.gitignore` is **not** a privacy control (anything ever
committed stays in history permanently), why removing a secret properly means
rewriting history, and branch protection as a technical gate rather than
voluntary discipline.

**Git Lab 3** — merge conflicts on purpose: create one, read it, resolve it, and
learn why `git rebase` produces a different history than `git merge`.
