# Future Merge Findings

These notes keep only durable facts that can help the next upstream sync.

## Repository References

- Fork remote: `origin` -> `https://github.com/Ice-wilderness/Rehide.git`
- Upstream remote: `upstream` -> `https://github.com/uhhhh15/hide.git`
- Rehide long-term rules live in `AI_MERGE_GUIDE.md`; one-off merge plans belong here or in `task_plan.md`.

## Upstream 3.9.2 Facts

- Local pre-merge target was Rehide `3.8.7` at `3eb27e8`.
- Upstream target was `upstream/main` at `0ca18fa`, version `3.9.2`.
- Common base was `0eecad37b4d7836fcfb03cf3294b7a520f55e2a4`.
- Upstream changed many files through delete/re-upload commits, but the real 3.9.2 delta was concentrated in:
  - `index.js`: real feature changes.
  - `manifest.json`: version bump from `3.8.7` to `3.9.2`.
  - `update.js`: CRLF-only rewrite; no logic change after ignoring CR at EOL.

## Durable Merge Lessons

- Use `git diff --ignore-cr-at-eol` or `git diff -w` when upstream re-uploads files; otherwise line-ending churn hides the useful changes.
- `update.js` should normally keep Rehide's update source unless upstream has a real logic change.
- `manifest.json` should take the upstream version but keep the Rehide author field.
- `index.js` is the risky file: prefer upstream as the feature baseline, then re-apply Rehide safeguards.

## Rehide Safeguards Upstream Still Lacks

- `limiter_migration_v2_complete` must default to `false`; migration code may set it to `true` only after running.
- Role/entity mode with no prior settings must default to `{ hideLastN: 6, lastProcessedLength: 0, userConfigured: true }`.
- Messages hidden by this plugin must get `hide_helper_hidden = true`.
- Restore/unhide paths must only restore messages where `hide_helper_hidden === true`, then delete that marker.
- `update.js` must keep `Ice-wilderness/Rehide` as the update source.
- `manifest.json` must keep `uhhhh15,Ice_wilderness` as author.

## Upstream 3.9.2 Features Worth Preserving

- `logUiOpenedAt` and `disableLogUi()` for the 60-minute log UI fuse.
- `isOurWiScan` to ignore world-info scans not triggered by this plugin's dry run.
- Recursive `chatHistory` leaf collection in `updateTokenStatsUI()`.
- Unified popup close handling through `closePopup()`.
- Token stats refresh on every switch to the chat stats tab.
- More detailed fallback truncation logging in `HideHelper_interceptGeneration()`.

## Useful Verification Commands

- `git diff --ignore-cr-at-eol --stat <base>..upstream/main`
- `git merge-tree <base> HEAD upstream/main`
- `rg "hide_helper_hidden|hideLastN: 6|limiter_migration_v2_complete" index.js`
- `rg "Ice-wilderness/Rehide|uhhhh15,Ice_wilderness|3.9.2" update.js manifest.json`
- `git diff --cached --check`
- `Get-Content -Raw index.js | node --input-type=module --check`
