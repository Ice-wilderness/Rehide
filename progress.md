# Merge Handoff

## 2026-10-30 同步上游 4.2.4

- 合并前本地提交：`3d6d41a`；上次上游基点：`ebfbb46`（4.2.3）。
- 本次上游提交：`fe87619`（4.2.4）；合并分支：`merge/upstream-4.2.4`。
- 合入移动端软键盘动画期间的弹窗宽度与居中修复、视口滚动重排、补帧合并和关闭清理。
- 仅版本信息发生冲突，保留 Rehide 作者并采用上游版本号；隐藏、恢复和设置迁移函数与合并前一致，更新源保持 Rehide。
- 验证通过：JavaScript 模块语法、JSON、差异空白、冲突标记及编码检查。
- 使用 Node VM 验证实际弹窗函数在移动端、键盘弹出、视口误报、桌面和自定义最大宽度下的定位，以及重排补帧合并和关闭时取消补帧。
- 尚未在真实移动端 SillyTavern 宿主中联调。

## 2026-10-27 同步上游 4.2.3

- 合并前本地提交：`d6fb4f8`；上次上游基点：`0ca18fa`（3.9.2）。
- 本次上游提交：`ebfbb46`（4.2.3）；合并分支：`merge/upstream-4.2.3`。
- 合入上游聊天统计重构、上下文查看器、移动端适配、功能开关及生成拦截器声明。
- 解决 `index.js`、`manifest.json`、`style.css` 的冲突，采用上游新统计流程和主题选择器，保留 Rehide 定制。
- 已核对角色默认保留最新 6 楼、隐藏标记与恢复保护、Limiter 迁移默认值，以及 Rehide 作者和更新源。
- 验证通过：三个 JavaScript 文件的模块语法检查、JSON 解析、差异空白检查、冲突标记与编码检查。
- 使用 Node VM 运行实际函数，验证角色/全局默认配置、全量和增量隐藏、数据与 DOM 的手动隐藏保护、取消隐藏以及旧 Limiter 设置迁移和防重复迁移。
- 当前仓库不含 SillyTavern 宿主及完整运行环境，尚未进行宿主内界面和真实生成联调。

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
