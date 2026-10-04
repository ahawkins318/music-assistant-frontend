# Local notes

Fork of `music-assistant/frontend` (`origin` = ahawkins318/music-assistant-frontend, `upstream` = the
original). This file lives only on the `local-notes` branch, so fix branches cut from upstream `main` stay
clean for PRs.

## Discover row provider filter has no label

On Discover, the filter icon on a row (Random albums, Recently played, ...) opens a bare checklist of music
providers. A tick **hides** that provider from the row, but nothing on screen says so, which reads as a
backwards filter. Seen on frontend 2.17.297 (server 2.10.5); still present on `main` as of 2026-10-03.

### How it got that way

- [#2324](https://github.com/music-assistant/frontend/pull/2324) "Add provider filter to Discover
  recommendation rows" (opened 2026-08-07, merged 2026-08-18). Tick-to-hide is its stated design: "hiding one
  removes its items from that row". Its screenshot shows "Hide music provider" above the checkboxes. That text
  was the placeholder of the search field `FacetedFilter` drew at the top of every list. The review thread only
  discusses frontend vs. server filtering; nobody raised the checkbox semantics.
- [#2419](https://github.com/music-assistant/frontend/pull/2419) "No more keyboard popping up over short filter
  lists" (merged 2026-08-11, between #2324's screenshot and its merge). Hides the search field unless the list
  has more than 8 options (`MAX_OPTIONS_WITHOUT_SEARCH`).
- Together: `FacetedFilter` shows its `title` only on its default trigger button and as the search placeholder.
  The Discover row supplies its own icon trigger, so with 8 or fewer providers the title is left only in
  aria-labels. Users with more than 8 providers still see it.
- #2751 later reworded the string to "Hide music source" (`tooltip.hide_provider`).

No report of this found on the frontend or support trackers.

### Related behaviour from #2324 (by design, not addressed by the patch)

- Selections are stored per user and row as `discover.hiddenProviders.<row id>`: a hide list, so a provider
  added later shows by default.
- A row with no items is not drawn. A row with an active filter is always drawn (`rowHasActiveFilter`), with a
  "no matches" state, so an empty row appears when providers are ticked and vanishes when they are all cleared.
- "Recently played" lists only fully played items; "In progress" only audiobooks and podcast episodes.

### The patch

Branch `label-short-filter-lists`, one commit on `main`: when the search field is hidden and a custom trigger
is used, `FacetedFilter` renders the title as a small heading above the list. Lists opened from the default
trigger are unchanged (their button already shows the title). It also labels the media-type filter in
`MediaSearch.vue`, the other custom-trigger caller.

- Files: `src/components/FacetedFilter.vue`, `tests/components/FacetedFilter.test.ts`
- Checked: `vitest run tests/components/FacetedFilter.test.ts` (9 pass), `oxlint`, `eslint`, `prettier`,
  `vue-tsc`. Not yet looked at in a browser.
- To see it: `pnpm dev`, open http://localhost:3000, connect to `http://nastycat46.local:8095`.

### Sending it upstream

- The branch is pushed to the fork. Open the PR from
  https://github.com/music-assistant/frontend/compare/main...ahawkins318:music-assistant-frontend:label-short-filter-lists
  (PR type in their template: bugfix). Rebase onto `upstream/main` first if it has moved.
- Upstream accepts AI-assisted contributions when written up and reviewed by the submitter; write the
  description in your own words.
- Points for the description: the label shown in #2324's own screenshot disappeared because #2419 landed first;
  reproduce with 8 or fewer providers; one-component fix with a test.
- Running the fix before upstream ships it means building the frontend into the server image; the
  `custom-deploy` overlay in `~/dev/music-assistant-server` copies only the server package today.
