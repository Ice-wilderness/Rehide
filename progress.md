# Merge Handoff

## 2026-05-09 Upstream 3.9.2 Merge

Final useful state for future reference:

- Work was performed on branch `merge/upstream-3.9.2`.
- Upstream `0ca18fa` / version `3.9.2` was merged from `upstream/main`.
- Conflicts occurred in `index.js`, `manifest.json`, and `update.js`.
- `manifest.json` was resolved with upstream version `3.9.2` and Rehide author `uhhhh15,Ice_wilderness`.
- `update.js` was resolved by keeping Rehide's update source because upstream only changed line endings.
- `index.js` was resolved by using upstream `3.9.2` as the functional baseline, then re-applying Rehide safeguards:
  - default role/entity keep-6 behavior.
  - `hide_helper_hidden` markers on plugin-hidden messages.
  - guarded restore/unhide paths.
  - `limiter_migration_v2_complete: false`.
- Three upstream whitespace-only additions in the ST-PT interceptor area were removed to satisfy `git diff --cached --check`.

## Verification Completed

- No unresolved conflict files.
- No exact conflict markers.
- `git diff --cached --check` passed.
- Rehide safeguard `rg` checks passed.
- `index.js` passed `node --input-type=module --check`.

## Commit Note

Use a merge commit message that mentions upstream `3.9.2` and explicitly notes that Rehide safeguards and fork metadata were preserved.
