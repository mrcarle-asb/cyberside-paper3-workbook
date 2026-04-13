# CyberSide Paper 3 Hyperbook

## What this is

A two-unit hyperbook (interactive workbook) companion for the **IB CS 2026 Paper 3 case study**, *An ethical approach to hacking*. Each unit wraps one *Cyberside Chats* podcast (LMG Security) with audio player, transcript excerpts, and scaffolded questions students answer directly in the page.

Built with [hyperbook](https://hyperbook.openpatch.org/). Content lives in plain markdown with `:::` directive syntax. Student answers persist in the browser via hyperbook's built-in `::textinput` directive (IndexedDB via Dexie, origin-scoped, never leaves the device).

Sibling of `flask-sequence-hyperbook` — same deployment stack, same `mrcarle-asb` org, same `.github.io` → Netlify migration pattern. This project exists as part of standardizing a hyperbook platform for CS students that works **on school wifi** (GitHub Pages is blocked, Netlify is not).

## Project structure

- `hyperbook.json` — config; `basePath=""` for Netlify root hosting; `allowDangerousHtml: true` (required for the turn-in bundler's inline `<script>`)
- `netlify.toml` — build command and publish dir
- `.github/workflows/deploy.yml` — legacy GH Pages workflow, kept for parity with flask-sequence; not the active deploy path
- `book/` — three pages:
  - `index.md` — landing page, explains the two units and how turn-in works
  - `01-ransomware-gangs.md` — Episode 59 (Ransomware Teamwork), 5 excerpts × ~2 questions each, no turn-in (context only)
  - `02-pentesting.md` — Episode 67 (on-site social engineering), 22 questions, ends with the turn-in bundler
- `public/` — audio files as **real copies**, not symlinks (symlinks broke on Netlify's Linux build container)

## Pedagogical framing

- Ransomware unit is the **hook** — tied to real-world news students walk in the door already knowing about (e.g. Rockstar ransomware hit the weekend before this was written). No turn-in; it's for thinking.
- Pentesting unit is the **technique reference** — almost every term on the Paper 3 *Additional terminology* list shows up in the wild. Turn-in lives here.
- Pages are single units: audio at top, excerpts + questions interleaved, textinputs under each question, saved in-browser.
- No printouts. No "listen and remember five things." A 17-year-old will actually engage with a page that holds their answers and a play button; they will not engage with a sheet of paper.

## Turn-in system

Pentesting page ends with a **Bundle my answers** button. When clicked it:

1. Reads all pentesting textinputs from `window.hyperbook.store.textinput.toArray()` (IndexedDB via Dexie; functionally equivalent to localStorage).
2. Filters to non-empty answers whose IDs appear in the `LABELS` map at the bottom of `book/02-pentesting.md`.
3. Formats them as a plain-text block with divider lines, preserving `LABELS` insertion order.
4. Dumps the block into a `<textarea>`. Student reviews, clicks **Copy to clipboard**, pastes into a Google Doc, adds their name, shares with the teacher.

**No backend.** No LMS integration. Google Docs is the delivery channel.

### Things NOT to break about the turn-in system

- **Never rename textinput IDs.** They're the keys in IndexedDB. Rename `pen-q4` → `pen-q04` and every student who already answered Q4 loses that answer.
- **The LABELS map lives in `book/02-pentesting.md`.** If you add a new question, add a new `::textinput{id="pen-qN"}` AND a matching entry in LABELS, or the bundler will silently skip it.
- **ID convention:** `pen-q{1..19}` for numbered questions, `pen-s{1..3}` for stretch, `ransom-ex{N}-q{M}` for ransomware (not bundled, but still stable).
- **The inline `<script>` only runs because `allowDangerousHtml: true` is set in hyperbook.json.** Don't flip that to false.

## Content conventions

- Excerpts from podcasts are blockquotes attributed to the speaker by name (Sherri, Matt, Tom, Derek) — these are real people from LMG Security, keep attributions accurate.
- "Case study connection" sidebar after each excerpt ties the real-world content back to the fictional MTPH scenario.
- Questions use bold labels (`**Q4.**`) followed by the prompt, then the textinput directive. Don't reformat — the bundler reads from IndexedDB, not from the DOM, so layout is flexible, but the pattern matters for consistency across the page.
- Use `:::alert{info}` for "your answers are local" reminders, `:::alert{warn}` for "empty answers won't be submitted."

## Source materials

- Original handout drafts live at `../../CS Explain/CyberSide_Chat_m26Paper3/`:
  - `ransomware_excerpts_handout.md` — source for Unit 1
  - `pentesting_listening_guide.md` — source for Unit 2
  - `ransomware_gangs.md` / `pentesting.md` — raw podcast transcripts
  - `ibcs_whitehat_casestudy.pdf` — the official IB case study booklet
- Audio files: copied from the same folder; originals are still there. If audio needs to be re-swapped, refresh both the mp3s in `public/` AND rebuild.

## Deployment

**Netlify is the active deploy target**, not GH Pages. School wifi blocks `*.github.io`.

- Push to `main` → Netlify auto-builds via `netlify.toml`
- `deploy.yml` still runs on push (legacy), but its output isn't the student-facing URL
- Live at: _[fill in Netlify URL after first deploy]_

The `mrcarle-asb` org holds the repo. Naming convention: repo is `cyberside-paper3-workbook` on GitHub (local folder is `-hyperbook`, remote is `-workbook`, matching flask-sequence).

## Dev commands

```
npx -y hyperbook dev     # local server; defaults to :8080, use --port 8090 if flask-sequence is already running
npx -y hyperbook build   # static build to .hyperbook/out/
```

## GitHub

- Org: mrcarle-asb
- Repo (recommended): `cyberside-paper3-workbook`
- Branch: `main`
- Netlify connected via Git integration (same pattern as flask-sequence)
