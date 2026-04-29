# Opus 4.6 Handoff Report — Premium Bukkake Cum Girls Dashboard HTML

## Purpose of this report

This is a full technical handoff for Opus 4.6. The user will feed you the latest HTML file separately. Your job is to understand the current state of the project, what was audited, what was patched, what was intentionally not changed, what broke along the way, and what lessons must be preserved going forward.

The project is a single self-contained HTML dashboard with embedded data, embedded images, an embedded XLSX workbook, a local `PB_Images` folder workflow, and an optional `pb_girls_images.js` sidecar for gallery thumbnails and image metadata.

The latest delivered HTML at the time this report was written was:

`pb_girls_28_04_2026_workflow_regen_label_fixed.html`

The immediately previous significant file was:

`pb_girls_28_04_2026_workflow_gallery_exact_link_fixed.html`

The latest change was only a label rename from “Regenerate sidecar” to “Regenerate pb_girls_images.js”; no data, sheets, XLSX, image blocks, or behavior changed in that last patch.

The user wants future work to preserve the working appearance, layout, colors, interaction model, and data. They strongly dislikes regressions caused by overzealous rewrites. Treat the HTML as a fragile production artifact, not as a clean codebase to redesign.

---

# 1. High-level project description

The app is a single-page HTML dashboard for a PremiumBukkake data set.

It contains:

1. Main grid of girls.
2. Modal detail cards.
3. Normal and cum image support.
4. Embedded thumbnail/gallery image support.
5. Optional full-size local folder image support through browser File System Access API.
6. Optional `pb_girls_images.js` sidecar support.
7. Excel-like sheet viewer with tabs:
   - `PB Cum Girls List`
   - `Nationality Leaderboard`
   - `Cum Leaderboard`
   - `Youngest Cum Girls`
   - `README`
   - `Change Log`
8. Embedded XLSX workbook stored as `XLSX_B64`.
9. Export actions for:
   - updated HTML
   - updated XLSX
   - updated/regenerated `pb_girls_images.js`
10. Validity/audit panel.
11. Image Manager.
12. Image picker/editor.
13. Gallery lightbox.

The data is stored mostly in embedded JavaScript globals.

Important objects:

- `GIRLS`
- `SHEETS`
- `EXPECTED`
- `TAGS`
- `TRAILERS`
- `XLSX_B64`
- `IMGS`
- `IMGS_CUM`
- `IMGS_GALLERY_STRINGS`
- `IMGS_GALLERY_CACHE`
- `PHOTO_FILES`
- `CUM_FILES`
- `GAL_FILES`
- `BADGES`

The user cares deeply about traceability. Every significant change should be recorded in the README and Change Log sheets. The latest version already added extensive backfilled entries, but future changes must keep doing this.

---

# 2. Core rule for future patches

Do not casually rewrite the HTML.

The safest patch strategy is:

1. Parse the latest HTML.
2. Identify the exact function/object/style block to patch.
3. Make the smallest possible text/data change.
4. Preserve all primary data objects unless the user explicitly asks for a data correction.
5. Regenerate dependent/mirror/export artifacts only when necessary.
6. Re-run consistency checks.
7. Return the HTML directly. Do not package a ZIP unless the user explicitly asks.

The user explicitly complained that ZIP output was redundant when the HTML is already available. Avoid ZIPs by default.

---

# 3. Data hierarchy and canonical sources

## 3.1 Canonical primary data

`GIRLS` is the canonical source for per-girl headline data:

- name
- nationality
- flag
- year
- age at first scene
- main scenes
- loads
- max scene
- bukkake scenes
- gloryhole scenes
- gangbang scenes
- blowbang scenes
- BTS scenes
- deceased status
- birthday
- current age

`SHEETS['PB Cum Girls List']` is the canonical spreadsheet-style representation, but it must match `GIRLS` for all mirrored fields.

`TRAILERS` is the canonical source for trailer URLs and trailer scene filenames.

`IMGS` and `IMGS_CUM` are embedded normal/cum photos.

`IMGS_GALLERY_STRINGS`, `IMGS_GALLERY_CACHE`, and `GAL_FILES` are gallery/sidecar data structures. `GAL_FILES` is especially important after the latest gallery fix because it provides exact filename mapping between gallery thumbnails and local full-size folder files.

## 3.2 Derived/mirror data

These should be regenerated or synced from canonical data, not manually trusted:

- `EXPECTED`
- leaderboards inside `SHEETS`
- per-girl summary block inside `PB Cum Girls List`
- stats rows
- embedded `XLSX_B64`
- trailer-code column values
- README / Change Log documentation rows when a change is made
- image manager counts
- sidecar export content

## 3.3 Persistent browser handles are not exportable

Connected folder handles cannot survive in a static exported HTML file. After reopening the HTML, the user must reconnect:

- `PB_Images` folder
- `pb_girls_images.js` sidecar if needed

Do not treat missing handles after reload as data loss. That is browser security behavior.

---

# 4. Original audit findings and baseline corrections

The initial audit discovered several stale or inconsistent artifacts.

## 4.1 Header stats were stale

Original visible header showed:

- 229 girls
- 32,118 loads
- 142.1 average

But `GIRLS` contained:

- 229 girls
- initially 32,350 loads
- average 141.3

Later, after a parallel audit found the Bruna Santos correction, the canonical total became:

- 229 girls
- 32,325 loads
- 141.2 average
- 113.14 L using 3.5 ml/load

The latest file should use:

- `229 girls`
- `32,325 loads swallowed`
- `141.2 avg per girl`

## 4.2 `EXPECTED` was stale/incomplete

`EXPECTED` originally had 229 keys but mismatched `GIRLS` in 33 fields.

Examples included:

- `Alice Seduce` missing multiple fields.
- `Dalila Lapiedra` missing multiple fields.
- `Marisa Leone` missing multiple fields.
- `Chloe Heart` had a direct birthday/current-age conflict.
- `Zaira` had a BTS mismatch.

Fix applied:

- Treat `GIRLS` as canonical.
- Rebuild/sync `EXPECTED` from `GIRLS`.

Important lesson:

A previous attempted fix made `EXPECTED` dynamic via a later `const EXPECTED = GIRLS.reduce(...)`, which caused a startup temporal-dead-zone failure because earlier startup code called `syncExpectedFromGirls()` before `EXPECTED` existed. Do not reintroduce that pattern.

Current expected pattern:

- Static `EXPECTED` exists and is valid.
- Runtime sync helpers must not reference a later `const` before initialization.
- If you make `EXPECTED` dynamic, it must be declared before anything uses it, or use `var`, or avoid TDZ entirely.

## 4.3 Embedded XLSX was stale

The embedded `XLSX_B64` workbook did not match `SHEETS`.

Original rows differed significantly:

`SHEETS`:
- PB Cum Girls List: 537 rows
- Nationality Leaderboard: 50 rows
- Cum Leaderboard: 234 rows
- Youngest Cum Girls: 24 rows
- README: 116 rows
- Change Log: 55 rows

Embedded workbook:
- PB Cum Girls List: 534 rows
- Nationality Leaderboard: 61 rows
- Cum Leaderboard: 251 rows
- Youngest Cum Girls: 42 rows
- README: 539 rows
- Change Log: 503 rows

Fix applied:

- Rebuilt embedded `XLSX_B64` from `SHEETS`.
- Later improved export pipeline so HTML export first syncs/rebuilds `XLSX_B64` before serialization.
- Later used the last released XLSX file as style template to prevent white unstyled Excel regions.

Critical rule now:

Before exporting HTML or XLSX, call the sync routine that makes `XLSX_B64` match `SHEETS`.

A stale mismatch must not happen again.

## 4.4 PB sheet row order was wrong

The latest new girls had been appended after `Zlata Shine` rather than sorted alphabetically.

Example wrong tail:

- Zaira
- Zlata Shine
- Marisa Leone
- Alice Seduce
- Dalila Lapiedra

Fix applied:

- Sort main PB data rows alphabetically by name.
- Renumber `#`.
- Keep derived sections below intact.

Current expected:

- 229 PB main data rows.
- First alphabetic entry: `Abby Margarita`.
- Last alphabetic entry: `Zlata Shine`.

## 4.5 Derived stats and summary blocks were stale

PB sheet stats and per-girl summaries were rebuilt from current data.

Correct current canonical totals after Bruna Santos correction:

- Girls: 229
- Total swallowed loads: 32,325
- Liters: 113.14 L
- Average: 141.2
- Total unique scenes: 1,248
- Bukkake scenes: 406
- Gloryhole scenes: 64
- Gangbang scenes: 31
- Blowbang scenes: 12
- BTS scenes: 440
- Interview scenes: 269
- Other scenes: 26

## 4.6 Nationality filter had duplicate static options

The nationality filter had hundreds of duplicated option entries.

Fix applied:

- Rebuilt the nationality filter from `GIRLS`.
- Kept pseudo-filters:
  - `_latina`
  - `_slavic`
  - `_european`
  - `_arab`
  - `_ebony`

## 4.7 `addNewGirl()` lost birthday/current-age fields

Original `addNewGirl()` read birthday/current-age from the UI but did not push them into the new `GIRLS` object.

Fix applied:

The pushed object now includes:

- `bday`
- `curAge`

This matters because future rows must not lose exact date information.

## 4.8 Hard-coded row limits were brittle

Known original hard-coded problems:

- `r <= 226`
- `k2 = 278`
- `Math.min(280, pb.length)`
- loops that wrote into PB summary/stat rows accidentally

Fix applied:

Introduced/demanded dynamic boundary helpers, conceptually:

- `_pbDataEnd(rows)`
- `_pbPerGirlStart(rows)`

Future code should always use data boundary detection instead of fixed row numbers.

## 4.9 Cache invalidation selectors were wrong

Original cache invalidation targeted wrong IDs/classes:

- `[id^="esheet_"]`
- `excelTabs`

Actual viewer used:

- `#etabs`
- `#esheets`
- `.esheet`
- `#excelPanel`

Fix applied:

- Corrected cache clearing and rebuild behavior.

## 4.10 Current age logic was crude

Original current-age logic did:

`currentYear - birthYear`

This overstates current age if birthday has not yet occurred.

Fix applied:

- Full `DD-MM-YYYY` birthdays calculate exact age.
- Approximate `~YYYY` birthdays remain approximate.
- Deceased entries are handled according to app semantics.

Important helper behavior:

- `expectedCurrentAge(bday)` must exist because the validity panel uses it.
- It was accidentally missing once and broke the “Check Validity of Data” button.

---

# 5. Parallel audit addition: Bruna Santos correction

Another audit running in parallel found the Bruna Santos misattribution issue. This was important and superseded the initial totals.

Bruna Santos correction applied:

- Loads: 81 → 56
- Main scenes: corrected to 1
- Bukkake scenes: 2 → 1
- BTS scenes: 3 → 2
- Interview scenes: 1
- Total scenes: 4
- Max scene: 46

This changed global totals to:

- 32,325 loads
- 113.14 L
- 1,248 total scenes
- 406 bukkake
- 440 BTS
- average 141.2

Rejected from parallel audit:

- It suggested Bruna Santos current age should be 23.
- With birthday `23-11-2003` and date 2026-04-28, age is 22.
- Kept 22.

---

# 6. Startup break and fix

A version broke startup and stuck at the loading overlay.

Root cause:

- `EXPECTED` was changed to a later `const EXPECTED = GIRLS.reduce(...)`.
- Earlier startup code referenced it through `syncExpectedFromGirls()`.
- Because of JavaScript temporal dead zone, even `typeof EXPECTED` threw before initialization.

Fix:

- Return to safe static/deterministic `EXPECTED` handling.
- Ensure no early runtime function touches a later `const`.

Lesson:

Do not place major data declarations after code that may execute immediately and reference them.

---

# 7. Restoring confirmed layout from 19.04.2026 baseline

The user provided a last confirmed working release:

`PB_girls_19_04_2026.zip`

The HTML inside was used as visual/layout/style baseline.

The v2 file had broken many sheet layouts because it rendered whole sheets generically or doubled certain sections.

Specific issue:

- `PREMIUM BUKKAKE — OVERALL STATISTICS` was doubled/misrendered.
- `PB Cum Girls List`, `Nationality Leaderboard`, `Cum Leaderboard`, `Youngest Cum Girls`, `README`, and `Change Log` had varying layout damage.

Fix applied:

- Restore layout/rendering style from the 19.04 HTML.
- Preserve newer data and corrections.
- Keep safe code hardening.

Important rule:

If the user complains about appearance, compare against the 19.04 baseline or the last confirmed working screenshot, not against your own regenerated layout.

---

# 8. Cum Leaderboard corrections

The Cum Leaderboard style drifted from the previous release.

Old intended style:

- rank 1 gold
- rank 2 silver
- rank 3 bronze
- ranks 4–10 red/white
- ranks 11–25 dark grey/cyan
- lower ranks muted dark rows

Issues fixed:

- Wrong medal-band styles had been repeated down the sheet.
- Volume formatting under 1 L was wrong.

Current expected volume format:

- Above or equal to 1 L: `X.XX L`
- Under 1 L: `ml`, e.g. `28 loads / 98 ml`

Also restored sortable behavior while maintaining visual consistency.

---

# 9. Youngest leaderboard corrections

The user noticed some `Youngest Cum Girls` ages did not include month precision even though full birthdate and first scene date existed.

Fixes applied:

- Full date + first-scene date now produce `18y2m`, `18y9m`, etc.
- Chloe Heart: `18` → `18y2m`
- Rebeka Brown:
  - `19` → `18y9m`
  - moved into the 18-year-old section
  - PB main age corrected from `19` to `18` because birthdate `11-03-2002` and first scene `08-01-2021` make her 18y9m.

Important:

Date age calculation is based on exact first scene date, not just year.

---

# 10. Sortable header arrows

At one stage the auxiliary sheet sortable columns lost the `⇅` marker.

Fix applied:

- Restored `⇅` marker to sortable column headers.
- Use same `font-size:8px` style as PB sheet arrows.
- Keep headers clickable/sortable.

Later, date sorting was fixed.

Important sorting behavior:

- PB date columns such as `Exact Date of first scene at PremiumBukkake` must parse `DD-MM-YYYY` and `DD.MM.YYYY` as actual dates.
- Do not parse by the first number only; that sorts by day-of-month and is wrong.

---

# 11. Validity button fix

The “⚡ Check Validity of Data” button broke.

Root cause:

- Validation called `expectedCurrentAge(g.bday)`.
- That function was missing in that version.

Fix:

- Reinsert `expectedCurrentAge`.
- Ensure validation panel opens and reports issues.

Current expected:

- Check Validity button should work.
- It should not silently fail on missing helper functions.

---

# 12. Export workflow overhaul

The original “Save” flow had two options:

- Save As New
- Overwrite Current File

The overwrite path used `showSaveFilePicker` or required user gesture behavior and failed in browser contexts:

Error shown:

`Failed to execute 'showSaveFilePicker' on 'Window': Must be handling a user gesture to show a file picker. Try "Save As New" instead.`

User requested:

1. Remove overwrite function.
2. Save icon should offer:
   - Export HTML as New File
   - Export `.XLSX` as New File
3. After exporting HTML, ask whether to export `pb_girls_images.js` too.
4. Include a `?` help button explaining the sidecar.
5. Also allow image sidecar export from Image Manager.

Implemented:

- Removed broken overwrite workflow.
- Save panel now has:
  - `Export HTML as New File`
  - `Export .XLSX as New File`
- HTML export syncs `XLSX_B64` before download.
- After HTML export, the user is prompted to export `pb_girls_images.js`.
- Help button explains the sidecar workflow.
- Image Manager includes sidecar export/regenerate buttons.

Important lesson:

Do not use `showSaveFilePicker` for overwrite unless you can guarantee direct user gesture and browser support. The current safer workflow is download-as-new-file.

---

# 13. Stale XLSX mismatch prevention

The user exported an HTML and the embedded workbook was stale in a few tag cells.

Fix:

- Patch `XLSX_B64`.
- More importantly, update HTML export pipeline:
  - before serializing the HTML, sync/rebuild `XLSX_B64` from `SHEETS`
  - ensure exported HTML carries a matching embedded workbook

Current invariant:

`XLSX_B64` parsed workbook must match `SHEETS` cell-for-cell.

Future check:

- Parse `XLSX_B64`.
- Compare to `SHEETS`.
- Diffs should be zero.

---

# 14. XLSX styling issue and template fix

The user exported an XLSX and found big formatting/layout issues:

- white regions
- missing dark background
- inconsistent style coverage
- particularly visible in `Youngest Cum Girls`

Cause:

- XLSX XML was being regenerated without adequate style coverage in new/shifted rows/cells.
- Some cells existed without style IDs, causing Excel to display white defaults.

Fix:

- Use the last released `PB_girls_list_as_of_19_04_2026.xlsx` as style template.
- Rebuild current workbook values into that package.
- Ensure all used cells across all six sheets have style coverage.
- Verify no unstyled used cells.

Critical rule:

When changing sheet structure, especially adding a column, style coverage in XLSX must be checked. HTML style is not enough.

---

# 15. Loading popup disappeared and fix

User noticed the loading screen no longer showed.

Cause:

- The loading-screen CSS/functions were still present.
- But the actual DOM block `loadingPopupBackdrop` was missing.
- It likely got lost because the export captured live DOM after the popup removed itself.

Fix:

- Reinsert the loading popup DOM.
- Add an export guard/sanitizer so future exports reinsert/preserve it.

Important:

The export pipeline must not simply serialize arbitrary live DOM after transient elements have removed themselves. It must sanitize/reconstruct required static DOM blocks.

---

# 16. Export panel width

The export panel was too wide.

Fix:

- Narrowed `#saveMenu`.
- Example CSS concept:
  - `width:500px`
  - `max-width:calc(100vw - 48px)`

No behavior or data changed.

---

# 17. Image Manager: concepts and status

The Image Manager supports:

1. Connecting local `PB_Images` folder.
2. Connecting `pb_girls_images.js` sidecar.
3. Exporting or regenerating `pb_girls_images.js`.
4. Exporting updated HTML.
5. Showing image stats.

Important data types:

- HTML proper normal images: `IMGS`
- HTML proper cum images: `IMGS_CUM`
- Sidecar/gallery images: `IMGS_GALLERY_STRINGS` / `IMGS_GALLERY_CACHE`
- Sidecar/gallery filenames: `GAL_FILES`
- Local folder detection: browser folder handle and file maps

The manager status now should show:

- Girls count
- HTML normal image count
- HTML cum image count
- Sidecar gallery count
- Sidecar status
- Folder name
- Expected folder matches
- Detected folder count
- Orphan image key warnings when relevant

Color meanings in Image Stats:

- cyan = images embedded in HTML proper
- purple = gallery images from `pb_girls_images.js`
- green = actual image files detected in connected `PB_Images` folder

---

# 18. Image Stats behavior

Initially, Image Stats existed but was confusing.

At one point, the exported HTML captured a live open stats table. Then clicking Image Stats removed it because the function was implemented as a toggle.

Fixes:

- Remove stale pre-rendered stats panel from export.
- Image Stats now refreshes in place rather than disappearing.
- Future export sanitization strips transient panels.
- It should list every girl, not only missing rows, when connected.
- It should show expected folder names and matched folder names.
- It should support exact/fuzzy folder matches.

Image Stats is read-only. It should not mutate primary data.

---

# 19. Image Manager sidecar regeneration

User wanted regeneration available directly in Image Manager, not only after HTML export.

Implemented:

- Image Manager has button to regenerate/export `pb_girls_images.js`.
- Latest label was changed to avoid confusion:
  - `🔄 Regenerate pb_girls_images.js`
- Separate button remains:
  - `📤 Export pb_girls_images.js`

Important distinction:

- Regenerate button creates/updates sidecar content from current folder/gallery selections.
- Export button downloads the current sidecar data.

The user said the previous label “Regenerate sidecar” was confusing.

---

# 20. Regenerate button color logic

User said the regenerate button should be orange only when photo changes were made.

Implemented:

- Button is blue normally.
- Button turns orange only after image/gallery selections are changed in the current session.
- Button resets back to blue after regenerating/exporting/loading the sidecar.

Future requirement:

Do not leave the regenerate button orange by default after page load if nothing changed.

---

# 21. Image Manager help button

User said Image Manager lacked a help button.

Implemented:

- Help `?` button inside Image Manager header.
- Restored top-bar help button.
- It should explain:
  - connect folder
  - connect sidecar
  - export HTML
  - regenerate/export `pb_girls_images.js`
  - image counts
  - local folder handles do not persist

---

# 22. Picker modal nesting bug

User found that clicking a modal “Edit images” button did nothing, but then the picker randomly opened when clicking the `PB_Images Folder` button.

Root cause:

- Exported HTML had malformed modal nesting.
- `pickerBackdrop` was accidentally inside `imgMgrBackdrop`.
- So the picker opened but was hidden inside the closed Image Manager.
- Later opening Image Manager made the already-open picker appear.

Fix:

- `pickerBackdrop` moved back to direct child of `<body>`.
- `pickerBody` closing structure corrected.
- Export sanitizer guard added so this nesting bug should not recur.

Critical export lesson:

Do not export transient/malformed modal DOM. Ensure modal backdrops are siblings, not nested inside each other.

---

# 23. Image click/lightbox fixes

User reported normal pic, cum pic, and gallery clicks no longer opened images.

Fixes:

- Modal normal/cum image clicks open immediately from embedded image.
- If local `PB_Images` file exists, they upgrade to the full local file.
- Gallery image clicks open immediately.
- Folder-backed navigation works if folder is available.
- If no live folder handle exists, picker tells user to reconnect rather than silently doing nothing.

Also:

- `savePicks()` updates `IMGS_GALLERY_CACHE` and `IMGS_GALLERY_STRINGS`.
- Newly selected gallery images are included when exporting `pb_girls_images.js`.

---

# 24. Gallery exact-link bug and fix

This was a major image-fetching weakness.

User clicked one gallery thumbnail but the lightbox opened a different full-size image.

Root cause:

- When folder was connected, the lightbox upgraded thumbnail to local full-size file by array index.
- JSON thumbnail order and folder file order can differ.
- So thumbnail #3 opened folder file #3, not necessarily the same image.

Fix:

- Gallery thumbnails now carry an exact `data-file` filename from `GAL_FILES`.
- Clicking opens embedded thumbnail immediately.
- Then it upgrades to local full-size only if the exact filename exists.
- Removed unsafe folder-index fallback for gallery.
- Lightbox navigation uses filename-aligned local files, not folder order.
- `pb_girls_images.js` export includes `GAL_FILES` metadata.
- Regenerating sidecar from folder writes thumbnail data and filenames in deterministic order.
- Loading sidecar imports optional `GAL_FILES`.

Important future rule:

Never map gallery thumbnails to local folder files by array index. Always use filename/path metadata.

---

# 25. Sidecar export format

Current sidecar should include gallery thumbnail data and metadata sufficient to reconstruct exact links.

Important globals:

- `IMGS_GALLERY_STRINGS`
- `IMGS_GALLERY_CACHE`
- `GAL_FILES`

Future sidecar loading should accept older sidecars without `GAL_FILES`, but exact local upgrade will only be reliable if `GAL_FILES` exists.

When regenerating sidecar, ensure:

- filenames are deterministic
- thumbnail data and filename arrays stay aligned
- sidecar export includes enough data for exact image resolution later

---

# 26. Miss Booty orphan image key

User renamed the image/folder to `Miss_Booty_Alina_Holsbeeks`.

Issue:

- `IMGS` had 230 keys but `GIRLS` had 229.
- One stale orphan normal image key existed:
  - `Miss Booty`
- Canonical current girl key:
  - `Miss Booty (Alina Holsbeeks)`

Fix:

- Removed only stale orphan `IMGS["Miss Booty"]`.
- Kept canonical `IMGS["Miss Booty (Alina Holsbeeks)"]`.

Verification:

- `GIRLS`: 229
- `IMGS`: 229
- current-girl normal images: 229/229
- orphan `IMGS` keys: 0

Important:

Do not delete canonical image keys. Only remove orphan keys not in `GIRLS`.

---

# 27. Photo counts

The user asked how there could be 230 normal photos if there were 229 girls.

Answer:

- Raw `IMGS` had an orphan stale key.
- Count should be based on current `GIRLS` keys, not raw object keys.
- Image Manager now counts current-girl coverage separately and warns about orphan keys.

Future check:

- current normal count = number of current `GIRLS` names present in `IMGS`
- raw normal count = raw object keys
- orphan count = `set(IMGS) - set(GIRLS names)`

---

# 28. Trailer Code column

User requested a new column after `Tags`:

`Scene/Trailer Code`

Example format:

`PB_453_dalilalapiedra_bts - BTS`

Implemented:

- Added new column immediately after `Tags` in `PB Cum Girls List`.
- Filled for all girls listed in `TRAILERS`.
- Derived from trailer URL filename plus scene type.
- Multiple trailers joined with ` / ` to reduce row height.
- Added sync logic so column regenerates from `TRAILERS` before HTML/XLSX export.
- Patched `addNewGirl()` so future rows include the new column.
- Updated embedded and standalone XLSX.

Verification at the time:

- PB header has 23 columns.
- Trailer-code rows filled for 228 girls.
- Total trailer entries represented: 1,206.
- XLSX vs `SHEETS`: 0 diffs.
- New XLSX column W styled, no white cells.

Important:

One girl still has no trailer mapping unless changed later:

- `Alyssa Reece`

Do not fabricate trailer data.

---

# 29. README and Change Log backfill

User complained a lot of recent updates, fixes, and lessons were missing from README/Change Log.

Implemented:

- Added README section for 28.04–29.04 audit/export/image-manager lessons.
- Added rules covering:
  - export sync
  - `XLSX_B64` vs `SHEETS` parity
  - transient DOM cleanup
  - loading popup preservation
  - image manager counts
  - `pb_girls_images.js` regeneration
  - picker/lightbox rules
  - date sorting
  - validation/current-age logic
  - trailer-code derivation
  - mandatory changelog discipline
- Extended README known pitfalls from 7 to 15.
- Added 24 Change Log rows.

Verification then:

- README rows: 136
- Change Log rows: 80
- embedded XLSX vs `SHEETS`: 0 diffs
- README/Change Log unstyled used cells in XLSX: 0
- JS syntax PASS

Future requirement:

Every significant fix should add a Change Log row and, when it establishes a general lesson/rule, a README note.

---

# 30. Footer update

User requested footer text:

`Made with ❤️ by LPCS1`

Implemented in the same font type/size as:

`PremiumBukkake.com · Data source: PB Cum Girls List · 229 models · Export HTML · Export XLSX`

Preserve this footer text going forward.

---

# 31. Interesting Facts column width

User said Interesting Facts was too narrow and made rows unnecessarily thick.

Fix:

- Widened the HTML sheet “Interesting Facts” column.
- Also made XLSX style/layout adjustments so row heights are less likely to balloon.

Future changes:

If columns are added or widths shift, check Interesting Facts column again.

---

# 32. Current latest label fix

The latest user request before this report:

They found it confusing that one button was called:

`🔄 Regenerate sidecar`

while another was:

`📤 Export pb_girls_images.js`

They wanted:

`Regenerate pb_girls_images.js`

Fix applied:

- Button renamed to:
  - `🔄 Regenerate pb_girls_images.js`
- Runtime status refresh also preserves that label.
- No data or behavior changed.

Latest delivered file:

`pb_girls_28_04_2026_workflow_regen_label_fixed.html`

---

# 33. Important known data warnings not auto-fixed

Some logical issues were identified but not automatically changed because they may reflect legitimate domain definitions or missing source context.

## 33.1 `max_scene > total loads`

Detected examples included:

- Alyrex (Aly Vega)
- Dayana Teen
- Francesca Palma
- Kattie Hill
- Katy
- Lady Gang
- Macarena Lewis
- Mary
- Sonia (Sonia Seine)

These may be real errors, or `max_scene` may count scene total differently from swallowed total. Do not auto-fix without source review.

## 33.2 `main != bukk + gh + gang + bb`

Detected for many girls.

This may be valid if “main” includes or excludes other categories differently. Do not auto-fix without source rules.

## 33.3 Scene-detail load sums

Scene-detail load sums often do not equal total loads due to:

- BTS
- interviews
- welcome/goodbye scenes
- missing dates
- official totals
- scrubbed scenes
- different denominators

Do not auto-fix based only on regex sums.

## 33.4 Missing trailer mapping

Known missing trailer mapping at one point:

- Alyssa Reece

Do not fabricate.

---

# 34. Required validation checks for future edits

Whenever modifying the HTML, run checks conceptually equivalent to the following.

## 34.1 Parse major globals

Extract and parse:

- `GIRLS`
- `SHEETS`
- `EXPECTED`
- `TAGS`
- `TRAILERS`
- `XLSX_B64`
- `IMGS`
- `IMGS_CUM`
- `IMGS_GALLERY_STRINGS`
- `GAL_FILES`
- `BADGES`

## 34.2 Basic counts

Expected latest core counts:

- `GIRLS`: 229
- `EXPECTED`: 229
- `TAGS`: 229
- `TRAILERS`: 228
- total loads: 32,325
- average: 141.2

Images as of the Miss Booty fix:

- `IMGS`: 229 current keys
- `IMGS_CUM`: 229
- raw galleries may depend on sidecar, but status previously showed 229 galleries after connecting/regenerating sidecar.

## 34.3 No primary data loss

Compare before/after for:

- `GIRLS`
- `SHEETS`
- `EXPECTED`
- `TAGS`
- `TRAILERS`
- `IMGS`
- `IMGS_CUM`
- `IMGS_GALLERY_STRINGS`
- `GAL_FILES`
- `BADGES`
- `XLSX_B64`

If you are only doing UI label/CSS/interaction fixes, all data blocks should be unchanged.

## 34.4 `GIRLS` vs `EXPECTED`

Fields to compare:

- `nat`
- `flag`
- `loads`
- `main`
- `max_scene`
- `bukk`
- `gh`
- `gang`
- `bb`
- `bts`
- `deceased`
- `age`
- `year`
- `bday`
- `curAge`

Expected mismatches: 0.

## 34.5 `GIRLS` vs PB sheet

PB sheet should have 229 main rows.

Core fields should match:

- name
- nationality
- birthday
- year
- first scene date
- age
- main scenes
- loads
- max scene average/indicator
- current age
- bukk
- gh
- gang
- bb
- bts

Expected core mismatches: 0.

## 34.6 XLSX vs `SHEETS`

Parse `XLSX_B64` workbook and compare values cell-by-cell against `SHEETS`.

Normalize:

- `[value, style]` → compare `value`
- missing/null/empty → empty string
- trim strings where appropriate

Expected diffs: 0.

## 34.7 XLSX style coverage

For every used cell in all six sheets, make sure it has style coverage or otherwise displays dark theme correctly.

The previous symptom of failure was white sections in Excel.

Expected unstyled used cells: 0, especially after adding columns or rows.

## 34.8 JS syntax

Extract inline scripts and run syntax check.

Expected: pass.

## 34.9 DOM structure

Check:

- `loadingPopupBackdrop` exists.
- `pickerBackdrop` is a direct body child, not nested inside `imgMgrBackdrop`.
- transient panels are not pre-rendered into exported HTML.
- no `blob:null/...` references persist in exported HTML.
- modal backdrops are closed by default.

## 34.10 UI workflow smoke tests

Manual/browser checks if possible:

- page loads and loading popup disappears
- grid renders 229 cards
- Check Validity opens
- Save menu opens
- Export HTML works
- Export XLSX works
- Image Manager opens
- connect folder button state displays
- connect sidecar button state displays
- Image Stats opens and refreshes
- modal opens
- edit data pencil works
- edit images button opens picker immediately
- normal image click opens lightbox
- cum image click opens lightbox
- gallery image click opens correct matching full-size image
- gallery navigation stays filename-aligned
- regenerate `pb_girls_images.js` button label is correct
- regenerate button is blue unless there are unsaved image changes

---

# 35. File export rules

The user explicitly dislikes unnecessary ZIPs.

Default response after editing should provide:

- HTML file link
- report file link, if useful

Do not produce a ZIP unless user asks.

When producing XLSX, provide it only if relevant to a sheet/XLSX/export change.

---

# 36. Recommended future patching methodology

## 36.1 For data-only fixes

1. Parse `GIRLS`.
2. Make minimal canonical data changes.
3. Rebuild affected `SHEETS` rows.
4. Rebuild `EXPECTED`.
5. Rebuild leaderboards/stats if totals changed.
6. Rebuild trailer code column if `TRAILERS` changed.
7. Rebuild `XLSX_B64`.
8. Add README/Change Log entries.
9. Verify.

## 36.2 For UI-only fixes

1. Avoid touching data globals.
2. Patch only function/CSS/DOM text.
3. Confirm data globals unchanged byte-for-byte if possible.
4. Run JS syntax.
5. Add Change Log entry only if significant.

## 36.3 For image workflow fixes

1. Do not mutate `IMGS`/`IMGS_CUM` unless explicitly intended.
2. Preserve `GAL_FILES`.
3. Never use folder index to map gallery thumbnail to local file.
4. Use exact filename/path metadata.
5. If changing picker or manager UI, test modal nesting.
6. Sanitize export DOM.

## 36.4 For XLSX changes

1. Use current `SHEETS` values.
2. Use last good XLSX as style template if possible.
3. Ensure `XLSX_B64` matches `SHEETS`.
4. Ensure styled cell coverage.
5. Export standalone XLSX if user needs it.
6. Add Change Log row.

---

# 37. Important code hotspots

Search for these identifiers in the HTML.

## 37.1 Data declarations

- `const GIRLS =`
- `const IMGS =`
- `const IMGS_CUM =`
- `var XLSX_B64 =`
- `var SHEETS =`
- `const EXPECTED =`
- `const TAGS =`
- `const TRAILERS =`
- `const BADGES =`
- `IMGS_GALLERY_STRINGS`
- `IMGS_GALLERY_CACHE`
- `GAL_FILES`

## 37.2 Sheet rendering/export

- `var SNAMES =`
- `function buildExcel()`
- `function renderS(name)`
- `function renderGeneric(name,rows)`
- `function sortPB(col)`
- `function sortGenericSheet(name,col)`
- `function exportUpdatedXLSX()`
- `function syncTagsToSheets()`
- `_pbDataEnd`
- `_pbPerGirlStart`

## 37.3 Save/export

- `showSaveMenu`
- `exportUpdatedHTML`
- `downloadUpdatedHTML`
- `exportDashboardHTML`
- `syncXLSXB64`
- `sanitizeExportDOM`
- `ensureLoadingPopupForExport`
- `exportUpdatedXLSX`

Exact names may differ; search semantically.

## 37.4 Image manager/picker/lightbox

- `openImageManager`
- `updateImgMgrStatus`
- `showImageStats`
- `connectImageFolder`
- `connectSidecar`
- `exportPbGirlsImages`
- `regeneratePbGirlsImages`
- `openImagePicker`
- `savePicks`
- `openLightbox`
- `openGalleryLightbox`
- `lbNav`
- `GAL_FILES`
- folder exact/fuzzy matching helper

## 37.5 Validity

- `runValidityCheck`
- `expectedCurrentAge`
- `parseBirthInfo`

## 37.6 Loading popup

- `loadingPopupBackdrop`
- `hideLoadingPopup`
- loading guard/sanitizer

---

# 38. Specific mistakes to avoid repeating

1. Do not make `EXPECTED` a late `const` referenced before initialization.
2. Do not serialize live DOM with transient panels open.
3. Do not let loading popup DOM be removed from exported HTML.
4. Do not nest `pickerBackdrop` inside `imgMgrBackdrop`.
5. Do not leave `blob:null/...` URLs in exported HTML.
6. Do not map gallery thumbnails to local files by index.
7. Do not generate XLSX rows without style IDs.
8. Do not change sheet layout/style unless explicitly asked.
9. Do not reintroduce fixed row numbers like 226/278.
10. Do not break the 19.04 visual style baseline.
11. Do not forget Change Log/README trace for significant changes.
12. Do not output ZIPs unless requested.
13. Do not delete orphan-looking data without confirming canonical key mapping.
14. Do not fabricate missing trailer URLs.
15. Do not let sidecar regeneration button stay orange without current-session changes.

---

# 39. Current expected user-facing behavior

## Main grid

- 229 girls
- 32,325 loads
- 141.2 average
- normal/cum images show
- connected folder upgrades images when available
- Show Cum Pics toggle works

## Modal

- Normal/cum image clicks open lightbox.
- Gallery thumbnails open correct images.
- Edit data pencil works.
- Edit images button opens picker immediately.
- Tags visible.
- Trailer scene details visible.

## Image Manager

- Connect `PB_Images` folder.
- Connect `pb_girls_images.js`.
- `🔄 Regenerate pb_girls_images.js` label.
- `📤 Export pb_girls_images.js` label.
- Regenerate button blue unless image changes made.
- Help button exists.
- Image Stats shows detailed counts.
- Counts distinguish HTML proper, sidecar/gallery, and folder detections.
- Expected folder name visible.

## Sheets

- `PB Cum Girls List` layout matches old style.
- `Scene/Trailer Code` column after Tags.
- Date sorting works for first scene date.
- Age column colors are consistent.
- Interesting Facts wider.
- Cum Leaderboard old medal-band styling.
- Youngest age precision where exact date available.
- README and Change Log contain late-April fixes.

## Export

- Save panel is not too wide.
- No overwrite path.
- Export HTML as new file.
- Export XLSX as new file.
- HTML export syncs XLSX first.
- Optional prompt to export `pb_girls_images.js`.
- Exported HTML includes loading popup.
- Exported HTML sanitizes transient UI.
- Exported HTML does not include stale blob URLs.
- Exported XLSX dark styling preserved.

---

# 40. Suggested report format after future work

When returning a patch to the user, be direct.

Include:

- what was fixed
- what was not changed
- validation checks
- file link(s)

Do not include a ZIP unless asked.

Example:

“Fixed. I changed only the gallery filename resolver and sidecar export metadata. `GIRLS`, `SHEETS`, `XLSX_B64`, `IMGS`, and `IMGS_CUM` are unchanged. JS syntax passes. Gallery local upgrade now uses exact filename from `GAL_FILES`, not folder index.”

---

# 41. Current likely file chain

The project went through many versions. Important chain:

1. Original uploaded ZIP around `pb_girls (5).zip`.
2. First audited fixed HTML.
3. v2 with Bruna correction.
4. Repaired v2 startup.
5. Restored layout from 19.04 baseline.
6. Cum/Youngest/style corrections.
7. Validity button fix.
8. Sort arrows fix.
9. User exported 28.04 version.
10. XLSX sync fix.
11. Workflow export fix.
12. Loading popup fix.
13. Export panel width fix.
14. Image stats fix.
15. Image stats v2/full counts.
16. Image clicks/picker fix.
17. Picker modal nesting/order fix.
18. Sort/images/help fix.
19. Miss Booty orphan fix.
20. Trailer-code column.
21. README/Change Log backfill.
22. Gallery exact-link fix.
23. Regenerate label fix.

The latest file to use is the latest HTML the user will feed you, not earlier files.

---

# 42. Final advice to Opus 4.6

This is not a clean greenfield project. It is a large self-contained HTML artifact with embedded data, stateful browser APIs, and a lot of fragile historical layout expectations.

The user’s priority order is:

1. Do not break working UI.
2. Do not lose data.
3. Keep the old visual style.
4. Fix the specific bug.
5. Preserve export integrity.
6. Add traceability to README/Change Log for significant changes.
7. Avoid unnecessary packaging and extra files.

Treat every change like a surgical patch.

If you are not sure whether something is source data or derived data, do not alter it. Report it.

If you change export behavior, test exported HTML structure.

If you change image behavior, test both embedded images and connected-folder upgrades.

If you change sheets, test both HTML sheet rendering and XLSX styling.

If you add columns/rows, check XLSX style coverage.

If you touch gallery logic, never rely on index order again. Use exact filename metadata.

End of handoff.
