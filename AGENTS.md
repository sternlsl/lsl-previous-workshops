# Project Summary

This project creates a static GitHub Pages-friendly replacement for the NYU Stern Learning Science Lab “Previous Workshops” page.

## Current Aim

Build a more navigable index of past Learning Science Lab workshops that preserves the original workshop content and resource links while adding functionality missing from the Stern CMS page.

The index should support:
- Searching workshop titles and descriptions
- Filtering by academic term
- Filtering by inferred topic/category
- Sorting by term or workshop name
- Preserving each workshop’s recording and slide links

## Source Content

Workshop content was extracted from:

https://www.stern.nyu.edu/portal-partners/faculty-staff/learning-science-lab/workshops/previous-workshops

The extracted dataset currently includes:
- 49 workshops
- 49 recording links
- 49 slide links
- Terms from Spring 2022 through Fall 2026
- Inferred topic categories based on workshop titles/descriptions

Three Fall 2026 sessions were added after the original extraction and are not
present in the source page.

## Current Output

Two pages live at the repository root. Each is a single static HTML file with
embedded CSS, JavaScript, and workshop data, deployable directly to GitHub Pages.

| File | Status | Purpose |
| --- | --- | --- |
| `index.html` | Live | The deployed replacement page: search, topic filter, card/list view, sort. |
| `index.tracks.html` | Exploratory | `index.html` plus a second "Learning Tracks" view. Shared with colleagues for feedback; **not** yet a replacement for `index.html`. |

`index.before-description-rewrite.html` is a retained snapshot from before the
workshop descriptions were rewritten. It is not deployed.

### Editing note

There is no generator script in this repository. Both pages are edited directly.
`index.tracks.html` was produced by copying `index.html` and injecting the tracks
CSS, markup, and JavaScript, so the two files share almost all of their code.

**A change to shared behavior (styling, workshop data, search, filtering, sorting)
must be applied to both files.** The workshop dataset is duplicated between them:
the `<script type="application/json" id="workshop-data">` block. Adding or editing
a workshop means editing that block in both places, plus the `bootcampIds` set and
the topic checkbox list in the controls band.

## Current Design Direction

The page mirrors the Stern source site broadly:
- Black top audience bar
- White institutional header
- Breadcrumb row
- Large page title
- Gray controls band
- Card-based workshop results
- Stern-like purple, black, gray, and accent styling
- Montserrat as the default font, loaded from Google Fonts

## Branding Status

The previous hand-built HTML approximation of the NYU Stern logo was removed because it would not conform to branding standards.

The header now contains a placeholder logo slot that expects a local file named:

`stern-logo.png`

Place that image next to `index.html` when deploying. If absent, the page shows a clean placeholder.

## Image Policy

Workshop thumbnail images were removed.

Reason: if this page replaces the source site, it should not depend on public image URLs from the Stern CMS. The generated page currently contains no workshop image URLs and no `<img>` tags.

## Learning Tracks View (`index.tracks.html` only)

A second, optional way to browse the same archive. It is additive: the existing
index and its search/filter/sort behavior are untouched.

### Structure

A tab bar (ARIA `tablist`, arrow-key navigable) sits between the page title and
the controls band and switches between two panels:

- **Search the Archive** (`#panelIndex`) — the original index, unchanged.
- **Learning Tracks** (`#panelTracks`) — carries a "New" badge so it is not missed.

The tracks panel has two states: an overview of four full-width track cards
(`#trackGrid`) with left-to-right workshop timelines (vertical on narrow
screens), and a detail view (`#trackDetail`) showing one track's ordered steps.

### Track data

Tracks are defined in the `learningTracks` array in the page script. Each entry:

```js
{
  id: 'ai',                      // URL-safe slug, used in ?track=
  name: 'Teaching with AI',
  blurb: 'One sentence on what the track is for.',
  steps: [
    {
      id: 'workshop-13',         // must match an id in the workshop-data JSON
      level: 'Foundation',       // Foundation | Core | Advanced
      note: 'Why this step comes here in the sequence.'
    }
  ]
}
```

Workshop titles, terms, descriptions, tags, and resource links are looked up from
the shared dataset via `workshopById`, so a track entry never duplicates workshop
content — only the ordering, the level, and the sequencing note.

### Curation rules

- Tracks run foundation to advanced. A step's `note` explains *why it comes at
  that point*, and must not restate the workshop description shown beneath it.
- No track exceeds five workshops. The four current tracks each have five.
- Older near-duplicate sessions are deliberately excluded from tracks so two
  steps never show the same title. Workshops outside these four tracks remain
  visible in the archive view.
- A workshop may appear in two tracks when it genuinely serves both.

### Current tracks

Brightspace Essentials (5), Teaching with AI (5), Participation & Motivation (5),
Visual Design & Presentations (5).

The overview design and track selection were revised, followed by an editorial
pass on the 20 track steps. Nineteen distinct workshop descriptions were updated
in both HTML files, and all track-specific step notes were revised. The workshop
order remains provisional. The two Brightspace setup sessions cover much of the
same ground at different speeds; their notes tell readers to choose one.
Visual Design & Presentations begins with the 15-minute PowerPoint boot camp
as its Foundation step, before the slide-design workshop.

### URL state

Track state composes with the existing query parameters:

- `?view=tracks` — open the Learning Tracks tab on the overview grid
- `?view=tracks&track=ai` — deep-link to one track's detail view
- `?track=ai` — `view=tracks` is implied, and the URL is normalized to include it

Unrecognized `track` slugs fall back to the overview grid, and the bad slug is
dropped from the URL. `q`, `topics`, and `sort` are preserved across both views
even though they only affect the archive.

**Ordering trap.** `currentMode` and `currentTrackId` are seeded from
`window.location.search` at declaration, next to `sortDirection` — deliberately
*not* in the tracks init block at the end of the script. The pre-existing
`update()` call runs before that block and calls `updateUrlState()`, which
rewrites the URL via `history.replaceState()`. Any track state read after that
point has already been stripped from the URL. (`q` and `topics` escape this
because `applyUrlState()` loads them into the form controls first, so
`updateUrlState()` re-emits them.) Anything new that reads query parameters must
be seeded before `update()` runs.

## Verification Status

`index.html` was checked with Playwright:
- 46 workshops render
- 0 `<img>` tags remain
- Search works
- Term filtering works
- Topic filtering works
- Sorting works
- Reset works
- Desktop and mobile layouts render cleanly

The four-track `index.tracks.html` revision was statically checked:
- Both scripts parse, and all 20 `steps[].id` values resolve against the dataset
- The workshop dataset still matches `index.html` exactly (49 records)
- The HTML has no nested buttons, and `git diff --check` passes

Selected workshop slide decks were consulted while revising descriptions,
including Brightspace setup, participation, AI, and presentation sessions. The
remaining descriptions were revised from the archive's existing summaries.

The prior ten-track version was checked in a browser at 1340px, 980px, and
375px. The new timeline layout has not yet received browser visual QA.

## Known External Dependencies

The page currently loads Montserrat from Google Fonts. If a fully offline/self-contained deployment is required, Montserrat would need to be bundled locally or replaced with a system fallback.
