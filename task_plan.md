# Upstream Merge Runbook

This file is the cleaned, reusable plan for future upstream merges. The 2026-05-09 upstream `3.9.2` merge used this flow successfully.

## Completed Phases

### Phase 1: Compare Upstream

**Status:** complete

Verify:

- `git fetch upstream --prune`
- `git log --oneline --reverse <base>..upstream/main`
- `git diff --ignore-cr-at-eol --stat <base>..upstream/main`

### Phase 2: Identify Noisy Diffs

**Status:** complete

Verify:

- Treat delete/re-upload commits as noise until `--ignore-cr-at-eol` or `-w` proves real logic changed.
- For 3.9.2, `update.js` was CRLF-only and should not override the Rehide update source.

### Phase 3: Resolve Fork Metadata

**Status:** complete

Verify:

- `manifest.json` takes the upstream version.
- `manifest.json` keeps `"author": "uhhhh15,Ice_wilderness"`.
- `update.js` keeps `Ice-wilderness/Rehide`.
- `AI_MERGE_GUIDE.md` is kept.

### Phase 4: Resolve `index.js`

**Status:** complete

Verify upstream features are preserved, then re-apply Rehide safeguards:

- `limiter_migration_v2_complete: false`
- `{ hideLastN: 6, lastProcessedLength: 0, userConfigured: true }` for missing role/entity settings only.
- `hide_helper_hidden = true` whenever this plugin hides messages.
- `hide_helper_hidden === true` guard before restoring messages.
- Delete `hide_helper_hidden` after restoring a plugin-hidden message.

### Phase 5: Verify Merge

**Status:** complete

Verify:

- `rg "hide_helper_hidden|hideLastN: 6|limiter_migration_v2_complete" index.js`
- `rg "Ice-wilderness/Rehide|uhhhh15,Ice_wilderness|3.9.2" update.js manifest.json`
- `git diff --cached --check`
- `Get-Content -Raw index.js | node --input-type=module --check`

## 3.9.2 Specific Resolution Record

- Base: `0eecad37b4d7836fcfb03cf3294b7a520f55e2a4`
- Upstream head: `0ca18fa`
- Local pre-merge head: `3eb27e8`
- Expected conflict files: `index.js`, `manifest.json`, `update.js`
- Staged merge result changed `index.js` and `manifest.json`; `update.js` stayed at Rehide logic.

## Future Merge Checklist

1. Start from a merge branch.
2. Fetch upstream and identify the common base.
3. Compare with line-ending noise ignored.
4. Merge upstream.
5. Resolve `manifest.json` and `update.js` before `index.js`.
6. Use upstream `index.js` as the feature baseline only when Rehide safeguards can be cleanly re-applied.
7. Run the verification commands above.
8. Commit only after the staged diff confirms Rehide metadata, update source, migration safety, default keep-6 behavior, and guarded unhide behavior.
