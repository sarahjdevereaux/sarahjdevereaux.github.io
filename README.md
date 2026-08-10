# sarahjdevereaux.github.io

Personal CV and portfolio page for Sarah Devereaux. Static HTML, CSS and a small
amount of JavaScript — no build step, no dependencies.

## Preview locally

Opening `index.html` directly works, but serving it over HTTP matches how GitHub
Pages behaves (root-relative paths, fonts, caching). From this folder:

```
python -m http.server 8000
```

Then visit <http://localhost:8000>. Ctrl+C to stop.

## Layout

```
index.html              CV / landing page
projects/               one page per project case study
docs/                   work reports and the Git Lab series (Project Alpha)
css/    tokens.css      design tokens — colours, type, spacing. Edit values HERE.
        base.css        reset, element defaults, accessibility, print styles
        layout.css      page structure: header, hero, sections, footer
        components.css  repeating pieces: entries, tags, project cards
js/main.js              mobile nav toggle + scroll reveal
img/                    portrait and favicon
```

Colours and sizes are defined once in `css/tokens.css`. Change them there rather
than hardcoding values in the other stylesheets.

## Still to do

- Replace the lorem ipsum and "Placeholder Organisation" entries with real
  experience, education and project detail.
- Fill in the Contact section: real email address and LinkedIn URL, and
  uncomment the Bluesky row if that account gets created. Everything still
  needing replacement is findable with `grep -rn placeholder --include=*.html .`
- Point the Repository / Live version links in `projects/project-alpha.html` at
  real URLs.
