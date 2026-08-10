# Work Report — 9 August 2026

**Repository:** `sarahjdevereaux.github.io`
**Session:** Portfolio site — first full development pass
**Environment:** VS Code + Claude Code (local), GitKraken Desktop (git client)
**State at close:** branch `basic-info-additions`, 1 uncommitted change; `main` 2 commits ahead of `origin/main`, unpushed. **Nothing published.**

---

## 1. Summary

The repository was created and taken from a one-line "Hello World" placeholder to
a structured, working portfolio site in a single day. The working method changed
partway through: earlier commits came from editing in the browser and copying
generated files in by hand; from mid-session onward development moved local, with
files edited directly in the working tree.

Nine defects were found and fixed. Site content remains largely placeholder by
deliberate decision — structure first, copy later.

---

## 2. Defects found and fixed

| # | Defect | Root cause | Status |
|---|---|---|---|
| 1 | Portrait image did not load | `src` omitted the `img/` directory; path was relative to the page, not the repo root | Fixed |
| 2 | Portrait rendered in the wrong position | `<img>` written as a sibling immediately after `</h1>`, so it flowed inline and the role text wrapped around it | Fixed |
| 3 | Hardcoded `border:5px solid white` inline | Bypassed the token system; white was off-palette | Moved to CSS using `var(--c-metal-dim)` |
| 4 | Image dimensions guessed (`420×420`) | True size is `418×450`; no source image is square | Corrected, then image swapped |
| 5 | Low-resolution portrait | 418px source displayed at up to 320 CSS px — soft on high-density screens | Swapped to `sarah-portrait-4.jpg` (1264×1568, and 76 KB *smaller*) |
| 6 | All three project links 404'd | `projects/project-alpha.html` did not exist; the file sat at repo root | Fixed by moving the file |
| 7 | Project page would have rendered unstyled | Its stylesheet links use `../css/`, which resolved outside the repo from the root | Fixed by the same move |
| 8 | Text collided with the decorative margin rules | Container gutter (1rem) was narrower than the rules' fixed 2rem inset — rules landed *inside* the text column between 60–78rem viewport width | Fixed; gutter and inset now coordinated |
| 9 | Nav item `Links` pointed at `#linktree` | No element with that `id` existed anywhere in the repo; the link silently did nothing | Section built as `#contact` |

Defects 6 and 7 shared one cause: a file in the wrong directory. Worth noting as
a pattern — when a link breaks, the file being misplaced is as likely as the link
being wrong.

---

## 3. Changes by area

**Structure**
- `project-alpha.html` → `projects/project-alpha.html` (moved with `git mv`, so history follows).
- Deleted `index-old.html` and `css/css2.css` — dead scaffolding. Both were publicly reachable on a Pages site.
- Added `docs/` for this report and the lab document.

**Hero**
- Rebuilt as a two-column grid: `.hero__text` and a `<figure class="hero__portrait">`.
- Portrait now `img/sarah-portrait-4.jpg` with true aspect ratio and no cropping hacks.

**Metadata / SEO**
- `<title>`, description, and both footers corrected from "Placeholder Name".
- Added `canonical`, `og:url`, `og:image`, `twitter:card` — link previews now render a card rather than a bare URL.
- Added `img/favicon.svg` (previously a 404 on every page load).

**New component**
- `.contact-list` in `css/components.css`; `#contact` section in `index.html`. Reuses the existing section furniture (`.section`, `.reveal`, `.section__header`) so it inherits scroll-reveal and spacing without new layout code.

**Copy**
- Hero role line and statement rewritten: self-assessments ("experienced", "seasoned") removed in favour of concrete function, and the role line reordered to lead with clinical informatics.

**Documentation**
- `README.md` rewritten with local preview instructions, a file-by-file map, and a to-do list.

---

## 4. Git activity

| Commit | Message | Assessment |
|---|---|---|
| `4a3effd` | reworked indext and site contents | Typo in message; predates this pass |
| `9bb7457` | Improved layout margins and image size | Understates scope — also moved a file, deleted two, added a favicon, rewrote metadata and README |
| `d4582f8` | added contact section to navbar | Understates scope — 98 lines across 3 files; the navbar was one line of it |

`main` is **2 commits ahead of `origin/main`** and unpushed. Branch
`basic-info-additions` was created but has **no commits on it yet** — the hero
copy edit is an uncommitted working-tree change.

**Loose end:** branch `html-edits` holds two commits (`b8ec3ab`, `8562704`) that
were never merged and never pushed. PR #1 merged only the commit before them.
Decide whether anything there still matters before the context is lost.

---

## 5. Deliberately not done

- **Lorem ipsum and "Placeholder Organisation" entries** — held by decision until real career detail is supplied.
- **Contact values** — email and LinkedIn are `example.com`-style placeholders. The Bluesky row is commented out because the account does not exist; shipping an empty profile link reads worse than omitting it.
- **Header logo mismatch** — `index.html` says "Sarah J. Devereaux", `projects/project-alpha.html` still says "SarahDev." One line, awaiting a preference.

---

## 6. Known risk before publishing

The hero now describes healthcare IT, clinical imaging and patient care. The
Experience section immediately below still lists *Senior Security Engineer* with
Splunk and MITRE ATT&CK, and *Data Engineer* with Airflow and dbt — template
fiction from the original scaffold.

**The page currently contradicts itself within one screen.** This is worse than
visible placeholder text: lorem ipsum reads as unfinished, whereas a contradiction
reads as inflated. This must be resolved before `main` is pushed.

---

## 7. Next steps

1. Commit the hero copy edit to `basic-info-additions`.
2. Push `main` so `origin/main` catches up (this deploys the session's work).
3. Push the branch, open PR, merge — completing the Lab 1 exercise.
4. **Lab 2 — Hardening the New Repo.** Add `.gitignore`, `.gitattributes` and a
   Pages publishing control. Currently the repo has none of the three, which
   means these very documents will be served as public web pages the moment
   `main` is pushed — Jekyll runs by default and converts markdown to HTML.
5. Replace the Experience entries so they match the hero.
6. Fill in real contact values.

## 8. Session close

Ended deliberately at a pause point rather than a finish line. Nothing is
committed beyond `d4582f8`; the hero copy edit and both documents in `docs/` are
uncommitted in the working tree on `basic-info-additions`. No push has occurred,
so the live site remains at `4a3effd` and none of this work is public.
