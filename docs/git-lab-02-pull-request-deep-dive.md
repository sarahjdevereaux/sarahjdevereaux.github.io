# Git Lab 2 — Pull Requests, Branches and Rebase: A Deep Dive

**Project Alpha · Lesson 2**

Lab 1 followed a repository through one clean pass: branch, push, propose, merge.
This lab covers what happens when that pass is *not* clean — when a branch has
gone stale, when a merge conflicts, and when a half-finished operation leaves
debris behind.

Every command, conflict and stash below is taken from the actual history of
`sarahjdevereaux.github.io` on **10 August 2026**, between 10:00 and 10:36. The
reflog and stash timestamps are quoted verbatim. Nothing here is invented for
teaching.

> **Numbering note.** Lab 1 announced Lab 2 as *Hardening the New Repo*. This lab
> displaced it, because the material presented itself. Hardening becomes Lab 3.

---

## Objectives

By the end of this lab you should be able to:

1. Name the **three places** your work can live, and say which commands move it between them.
2. Explain why a branch that has already been merged must not be reused.
3. Read a merge conflict message and identify *which kind* of conflict each line describes.
4. Explain the difference between `merge` and `rebase` in terms of what history they produce.
5. Say what a stash is, why git creates them without being asked, and how to inspect one safely.
6. Recognise **detached HEAD** and explain why it is the state most likely to lose work.

**Prerequisites:** Lab 1, particularly Part 5 (a branch is a pointer) and Part 7
(a pull request moves no data).

---

## Part 0 — The three places

Most confusion in this lab traces back to one missing picture. Your work can be in
three places, and they are genuinely separate:

```
   working tree  ──commit──▶  local repository  ──push──▶  GitHub
   (files on disk)            (.git on your PC)            (the server)
                                    ◀──────pull──────
```

Lab 1's three *states* (working tree → staging → repository) all live inside the
first two boxes. This lab is about the gap between box two and box three.

Consequences that catch everyone:

- **Merging on the GitHub website changes box three only.** Your computer learns nothing about it until you `pull`.
- **`git status` reports on boxes one and two.** It says "up to date" based on the last time you contacted the server, not on what the server holds right now.
- **A branch can exist in one box and not the others.** Lab 1's field note 1 — "I did a pull request, but no PR existed" — was exactly this.

### The state at the start of this lab

```
main                 04c7096 [origin/main: behind 5]
html-edits           8562704 [origin/html-edits: behind 7]
basic-info-additions 04c7096 [origin/basic-info-additions]
```

Read `behind 5` precisely: **GitHub's `main` has 5 commits this computer does not
have.** Those 5 commits were created *by clicking Merge on the website* — two
merge commits and the work they brought in. Nothing is wrong. The local copy is
simply out of date, and will stay that way until told otherwise.

> **A counting trap from the session this lab records.** "Behind 5" was misread as
> "5 files." It is 5 *commits*. Coincidentally there were also 5 stale files
> involved elsewhere in the same repo. The two fives are unrelated. When a number
> appears in git output, check its unit before building a theory on it.

---

## Part 1 — What a pull request is, restated

Lab 1 covered the etymology. The operational summary, since this is where the
confusion recurs:

**A branch and a pull request are not alternatives. They are the first and last
step of one sequence.**

```
1. git pull                    bring local main up to date
2. git switch -c <name>        branch from that fresh main
3. edit, git add, git commit   the actual work
4. git push -u origin <name>   publish the branch to GitHub
5. open a PR on GitHub         "please merge this branch into main"
6. click Merge                 main moves; on Pages, this deploys
7. delete the branch           it is spent
```

Step 2 exists *so that* step 5 is possible. You never choose between them. There
is no such thing as a pull request without a branch — the branch is what the
request is *about*.

Steps 1–4 are git, on your computer. Steps 5–7 are GitHub, in a browser. That
split is why the sequence feels like two unrelated activities, especially when
days pass between them.

---

## Part 2 — The stale branch

This is the root cause of everything else in this lab.

### What happened

`html-edits` was created on 9 August at 11:54 and merged into `main` by PR #1 that
afternoon. Its job was done. It was left in place.

On 10 August it was used again — a second PR, #3, from the same branch. But the
branch had not moved since 9 August at 14:10, while `main` had gained four
commits including a full rework of `index.html`.

### Why this is different from an ordinary out-of-date branch

A pull request proposes **the difference between your branch and `main`**. That
difference is computed from where the two histories last agreed.

When you branch from current `main`, do a day's work and open a PR, the difference
is your day's work. Clean.

When you reuse a branch whose work is already *in* `main`, the difference is no
longer "your new work." It is "everything `main` has learned since this branch
stopped moving" — pointed backwards. Every file `main` improved in the meantime
shows up as a proposed change, because from the stale branch's point of view those
improvements are edits it does not have.

**The rule:** once a branch's PR is merged, that branch is finished. Delete it.
New work gets a new branch, cut from freshly-pulled `main`.

> **Why "just delete it" is the whole fix.** A branch is a pointer (Lab 1, Part 5).
> Deleting one throws away a name, never a commit — the commits are already in
> `main`. Deleting a merged branch costs nothing and removes the only way to make
> this mistake.

---

## Part 3 — Reading a merge conflict

GitHub reported PR #3 as conflicted. Replaying that merge locally reproduces it
exactly:

```bash
git merge 8562704 d35cdc7
```

```
Auto-merging css/components.css
CONFLICT (add/add): Merge conflict in css/components.css
CONFLICT (rename/delete): css2.css renamed to css/css2.css in HEAD,
                          but deleted in d35cdc7.
CONFLICT (modify/delete): css/css2.css deleted in d35cdc7 and modified in HEAD.
                          Version HEAD of css/css2.css left in tree.
Auto-merging css/layout.css
CONFLICT (add/add): Merge conflict in css/layout.css
Auto-merging css/tokens.css
CONFLICT (add/add): Merge conflict in css/tokens.css
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

Five conflicts across four files. This was not a formality — it was resolved by
hand in GitHub's web editor, and resolved **correctly**: `main`'s `index.html`
survived intact. Verify it:

```bash
git diff --stat d35cdc7 afd1609   # index.html does not appear
```

### The conflict types, which are not interchangeable

Git names each conflict by *shape*, and the name tells you what decision it needs.

| Type | What it means | The question it asks |
|---|---|---|
| `content` | Both sides edited the same lines of a file both sides have | Which lines survive? |
| `add/add` | Both sides created a file at the same path, independently | Which file is *the* file? |
| `modify/delete` | One side edited a file the other side deleted | Is this file still wanted? |
| `rename/delete` | One side moved a file, the other deleted it | Same question, harder to see |

`add/add` on the three stylesheets is the interesting one. `main` wrote
`css/tokens.css` as part of the rework; `html-edits` had independently created a
file at that same path. Git had no shared ancestor for that path to compare
against, so it could not merge them — there was no "before" to measure either
side's change from. It presented both and asked.

> **Only `content` conflicts put `<<<<<<<` markers in a file.** The other three
> types are decisions about *whether a file exists at all*, so there is nothing to
> mark up. This is why a conflict can be reported on a file that looks completely
> normal when you open it.

### The one that got resolved the other way

Of the five, `css/css2.css` was kept from the stale side. That is why
`css/css2.css` sits on `main` today referenced by nothing — a dead file, created
by one `rename/delete` decision made in the middle of resolving four others.

This is worth sitting with. Four of five resolutions were right. The fifth was not
wrong so much as *unnoticed* — a file preserved by default because preserving
looks safer than deleting when you are five conflicts deep in a web form.

**`rename/delete` and `modify/delete` are where debris enters a repository.**
When you meet one, the useful question is not "which version?" but "does this file
still have a job?"

---

## Part 4 — Merge versus rebase

Both combine work. They produce different histories, and the difference is the
whole point.

### Merge

```
main         A───B───C───────M
                  \         /
your branch        D───E───
```

`M` is a **merge commit** — the only kind with two parents. `D` and `E` keep their
original hashes, dates and parents. The history records that a branch existed and
was joined. Nothing is rewritten.

### Rebase

```
before:  A───B───C          after:  A───B───C───D'───E'
              \
               D───E
```

Rebase takes your commits, sets them aside, moves your branch to the tip of
`main`, and **replays them one at a time** on the new base. `D'` and `E'` are new
commits with new hashes. Same changes, different parents. `D` and `E` still exist
in the reflog but nothing points at them.

History comes out linear, as though you had branched from current `main` all
along.

### Which to use

| | Merge | Rebase |
|---|---|---|
| History | Records what happened | Records a tidy fiction |
| Hashes | Preserved | Rewritten |
| Conflicts | Once, at the join | Potentially once **per commit** replayed |
| Safe on a pushed branch? | Yes | **No** |

That last row is the rule that matters. Rebasing rewrites commits; if anyone —
including GitHub — already has the originals, you have created two divergent
versions of the same work. On a solo repository the damage is limited to
confusing yourself, which is quite enough.

**Default: merge.** Reach for rebase when you understand specifically why you want
linear history and the branch has not been pushed.

---

## Part 5 — Stashes, and why four appeared

### What a stash is

A shelf. `git stash` sweeps uncommitted changes off your working tree and stores
them, leaving a clean tree behind. `git stash pop` puts them back.

It exists because several git operations *require* a clean working tree. Merge,
rebase and branch-switching all need to rewrite files on disk, and git refuses to
do that on top of edits you have not committed — refusing is how it avoids
destroying them.

### Why they appeared uninvited

```
stash@{0}  2026-08-10 10:31:40  On (no branch): Auto stash before rebase of "HEAD" onto "origin/main"
stash@{1}  2026-08-10 10:03:06  On (no branch): Auto stash before merge of "HEAD" and "origin/main"
stash@{2}  2026-08-10 10:02:39  On (no branch): Auto stash before merge of "HEAD" and "origin/main"
stash@{3}  2026-08-10 10:02:17  On (no branch): Auto stash before rebase of "HEAD" onto "origin/main"
```

Four stashes in a 29-minute window, none created deliberately.

**`Auto stash`** means git did it for you. Modern git, rather than refusing an
operation outright, shelves your changes, runs the operation, and restores them
afterwards. When the operation does not complete cleanly, the restore does not
happen — and the shelf keeps its contents. Four attempts, four abandoned shelves.

They are inert. Nothing is broken. But each holds a snapshot of a working tree
from a moment of uncertainty, and they will sit there indefinitely.

### The detail that matters most: `On (no branch)`

Every one of these stashes was taken while **not on any branch**. That is detached
HEAD, and it is Part 6.

### Inspecting one safely

Read before you restore. All three of these are read-only:

```bash
git stash list                  # what shelves exist
git stash show --stat stash@{0} # which files, how many lines
git stash show -p   stash@{0}   # the full diff
```

To compare a stash against what you already have — this is what "diffing the
stash" means, and it is the question you actually care about, *is there anything
in here I do not already have?*:

```bash
git diff stash@{0} main
```

Only then:

```bash
git stash pop stash@{0}   # restore and remove from the shelf
git stash apply stash@{0} # restore but keep the shelf copy
git stash drop  stash@{0} # discard
```

> **`apply` before `pop`.** `apply` leaves the stash in place, so a bad restore
> costs nothing. `pop` deletes on success, and a popped stash that conflicts is
> recoverable only through the reflog.

---

## Part 6 — Detached HEAD

`HEAD` normally points at a branch name, which points at a commit. Committing
moves the branch forward, and `HEAD` follows.

**Detached HEAD** is when `HEAD` points straight at a commit, with no branch in
between. You get there by checking out a raw commit hash, which the reflog shows
happening at 10:31:46:

```
d4582f8 HEAD@{2026-08-10 10:31:46}: checkout: moving from 8562704f45e3... to main
```

Moving *from a 40-character hash* — that is the signature of detached HEAD.

Everything works normally in this state, which is the trap. You can edit, stage
and commit. But new commits have nothing pointing at them, so the moment you
check out any branch, they become unreachable — no name, no branch, invisible to
`git log`. Git prints a warning about "leaving commits behind" that is easy to
scroll past.

Detached HEAD is also why the stashes read `On (no branch)`: git had no branch
name to record.

**Recognising it:** `git status` opens with `HEAD detached at <hash>` instead of
`On branch <name>`.

**Getting out, keeping the work:**

```bash
git switch -c rescue-branch   # name the commits, then they are safe
```

**Getting out, discarding it:**

```bash
git switch main
```

> **Reachability is the whole model.** Git deletes nothing you can still reach from
> a branch, a tag, or `HEAD`. Detached HEAD is the one everyday state that
> manufactures unreachable commits. The reflog is the safety net beneath it — it
> records every position `HEAD` held, so work "lost" this way is usually
> recoverable for about 30 days via `git reflog`.

---

## Part 7 — Reconstructing the session

The reflog reads as a narrative. This is the day's second half, in order:

| Time | Event | Reading |
|---|---|---|
| 09:48 | `commit: wrote up git-lab01 and a daily work report` | Last real work. `basic-info-additions` complete |
| 10:00 | `checkout: basic-info-additions → main → basic-info-additions` | Orientation. Looking for something |
| 10:02:17 | `Auto stash before rebase`, on no branch | First attempt. Detached, uncommitted changes present |
| 10:02:39 | `Auto stash before merge` | Switched strategy in 22 seconds |
| 10:03:06 | `Auto stash before merge` | Again |
| 10:31:40 | `Auto stash before rebase` | 28 minutes later, back to rebase |
| 10:31:46 | `checkout: moving from 8562704... to main` | Left detached HEAD. Recovered |
| 10:32:48 | `rebase (finish): returning to basic-info-additions` / `rebase: fast-forward` | Completed, and did nothing |
| 10:36:01 | `Merge branch 'main' into html-edits` | GitHub's "Update branch". The five conflicts, resolved by hand |
| — | PR #3 merged | `afd1609` |

Two observations worth more than the sequence itself.

**The 10:32 rebase was a no-op.** `rebase: fast-forward` means git found nothing to
replay — the branch was already based where it was being moved to. Alternating
merge and rebase against a branch that needed neither produced four stashes and no
change to the repository. *When an operation reports doing nothing, that is
information: the problem is elsewhere.*

**The work was never at risk.** Every commit from 09:48 survived, because it had
been committed. The flailing happened entirely in the layer above — detached HEAD,
stashes, uncommitted state — which is precisely the layer where flailing is
recoverable. Lab 1's Part 5 note holds: *a branch only begins protecting work once
you commit to it.* The corollary is the reassuring half. **Committed work is very
hard to lose.**

---

## Part 8 — The cleanup, as an exercise

The repository is currently correct and messy. The live site is fine; nothing
below is urgent. Each step is here because it teaches something.

### 8.1 — Sync

```bash
git switch main
git pull
```

Brings down the 5 commits made by clicking Merge. Confirm with `git log --oneline -3`
— the two merge commits should now be local.

### 8.2 — Inspect, then clear the stashes

Per Part 5: `show --stat`, then `diff` against `main`, then `drop`. Expect to find
they contain only work that already landed.

```bash
git stash drop stash@{0}   # repeat; indices renumber after each drop
```

> Dropping renumbers what remains — `stash@{1}` becomes `stash@{0}`. Dropping
> `{0}` four times is correct; dropping `{0}`,`{1}`,`{2}`,`{3}` is not.

### 8.3 — Delete the spent branches

Both had their work merged. Verify first:

```bash
git branch --merged main
```

Anything listed there is fully contained in `main` and safe to delete.

```bash
git branch -d basic-info-additions          # -d refuses if not merged. Use it, not -D
git push origin --delete basic-info-additions
```

Same for `html-edits` — the branch this lab is largely about.

> **`-d` versus `-D`.** `-d` deletes only if merged. `-D` deletes regardless. Use
> `-d` and let it stop you; if it objects, it has found something you did not know.

### 8.4 — A real PR, done deliberately

Five files reached `main` through PR #3 and are referenced by nothing:

| File | Why it is dead |
|---|---|
| `index-old.html` | The "Hello, World!" page from the first commit |
| `project-alpha.html` (root) | Superseded by `projects/project-alpha.html` |
| `css/css2.css` | The `rename/delete` resolution from Part 3 |
| `images/sarah-portrait-crop.jpg` | Wrong directory — the site uses `img/` |
| `images/sarah-portrait-new.png` | 1.1 MB, referenced nowhere |

Confirm for yourself rather than taking the table's word for it:

```bash
grep -oE '(href|src)="[^"]*"' index.html | sort -u
```

Then run the full Part 1 sequence on it: branch from fresh `main`, delete the
files, commit, push, open the PR, merge, delete the branch. Small, reversible,
zero-stakes — which is what makes it good practice rather than good housekeeping.

> **On `images/` versus `img/`.** Two directories for one purpose is how a
> repository starts rotting. Every future portrait now has two plausible homes and
> nobody, including you in a month, will remember which one is real. Pick `img/`,
> delete the other, keep it that way. Lab 1's field note 6 applies too: Windows
> will not warn you when case or directory is wrong, and Pages will.

---

## Field notes — real errors from this session

1. **A merged branch reused for new work.** The cause of PR #3's five conflicts. *Delete branches at merge time; the mistake becomes unavailable.*

2. **Merge and rebase alternated against a branch needing neither.** Four auto-stashes in 29 minutes, no change to the repository. *`rebase: fast-forward` means "nothing to do" — stop and diagnose instead of switching tools.*

3. **Detached HEAD, unnoticed.** All four stashes read `On (no branch)`. *`git status` line 1 tells you; it is the line most worth reading and least often read.*

4. **A `rename/delete` conflict resolved by keeping the file.** `css/css2.css` is dead weight on `main` today because of one decision made mid-way through resolving four others. *Deletion conflicts ask "is this wanted?", not "which version?"*

5. **"Behind 5" read as five files.** *Check the unit before theorising.*

6. **Merging on the website, then reasoning from a local `git status`.** The local clone knew nothing of either merge. *After merging in a browser, `git pull` before anything else.*

---

## Checkpoint questions

1. You merged a PR on github.com. Is your local `main` up to date? What tells you?
2. Why does reusing a merged branch produce conflicts in files you never touched?
3. `git status` says `HEAD detached at 8562704`. You have made two commits. What happens if you run `git switch main` now?
4. What is the difference between `git stash apply` and `git stash pop`, and which is safer?
5. A conflict is reported in `css/tokens.css`, but the file contains no `<<<<<<<` markers. Why?
6. You rebase a branch you pushed yesterday. What have you done to the copy on GitHub?
7. `rebase: fast-forward` appears in your reflog. What did the rebase change?
8. Which of these can lose committed work: merge, rebase, stash, detached HEAD?

---

## Glossary

Additions to Lab 1's.

| Term | Meaning |
|---|---|
| **rebase** | Replay commits onto a new base. Creates new commits with new hashes |
| **merge commit** | A commit with two parents, recording a join |
| **fast-forward** | Moving a pointer forward when no divergence exists. No new commit |
| **stash** | A shelf for uncommitted changes; lets git clean the tree without discarding work |
| **auto stash** | A stash git created on your behalf before a merge or rebase |
| **detached HEAD** | `HEAD` pointing at a commit rather than a branch. New commits here are unreachable |
| **reachable** | Findable by walking back from a branch, tag or `HEAD`. Unreachable objects are eventually deleted |
| **reflog** | A local log of every position `HEAD` and branches have held. The recovery tool of last resort |
| **behind / ahead** | Commit counts between a local branch and its upstream, as of your last fetch |
| **add/add conflict** | Both sides independently created a file at one path |
| **rename/delete conflict** | One side moved a file, the other deleted it |

---

## Next lab

**Git Lab 3 — Hardening the New Repo** (displaced from Lab 2).

`.gitignore`, `.gitattributes`, `_config.yml` / `.nojekyll`, and branch protection
— the difference between what git tracks, how it treats what it tracks, and what
Pages actually serves. Also: why `.gitignore` is not a privacy control, and why
removing a secret properly means rewriting history.

The `images/` versus `img/` split from Part 8.4 is the natural opening: the
shopfront and the workshop have started to blur, and hardening is how they get
separated again.
