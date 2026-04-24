# 🎨 Rehide - 自定义功能及合并指南 (AI Merge Guide)

由于本仓库是由上游项目（`uhhhh15/hide` 等）复刻（Fork）并定制的，本地包含了部分非常强依赖且独特的“自定义功能”。随着上游架构的频繁更新，代码合并时经常产生冲突。

**⚠️ 请后续接手代码合并（Sync / Merge）的 AI 助手或开发者，务必仔细阅读本指南！** 在处理冲突时，**绝对不能盲目采用上游的代码覆盖本地**，必须将以下自定义逻辑完美地融合（移植）到上游的新代码结构中！

---

## 🌟 核心自定义功能列表

### 1. 角色模式下默认保留最新 6 楼 (Default Keep Latest 6 Messages in Role mode)
**功能目的：** 在角色对话模式下，如果没有读取到用户的历史隐藏配置（比如第一次加载，或者清理了缓存），为了保持页面清爽，功能应当默认生效，并**保留最新 6 条消息可见/可发送**（`hideLastN: 6`），更早的消息由插件自动隐藏。

**代码表现形式：**
在获取该插件当前运行设定的代码块（如 `getCurrentHideSettings()` 中），如果判定当前配置为空，必须返回带有 `hideLastN: 6` 的默认化对象：
```javascript
let settings = extension_settings[extensionName]?.settings_by_entity?.[entityId];
if (!settings) {
    // 强制赋予初始默认值
    settings = { hideLastN: 6, lastProcessedLength: 0, userConfigured: true };
}
```
**🔴 合并注意事项：**
上游如果重构了设置和缓存的管理方式（比如使用了新的 `Settings Manager` 或全局 `State`），请务必在“读取当前角色/群组设置且不存在历史配置”的地方注入该默认条件。注意：这只针对**角色/实体模式**；全局模式继续使用 `globalHideSettings`，不要强行套用 6 楼默认值。

---

### 2. 手动隐藏冲突防呆/防重复标记 (Manual Hide Conflict Prevention / `hide_helper_hidden`)
**功能目的：** 
上游原版的一个痛点是：在执行“取消隐藏”（Unhide）或者进行状态同步时，上游代码往往简单粗暴地将所有 `is_system = true` 的消息都转回 `false`。
这会**引发严重 Bug**：导致用户“手动隐藏”的消息或者由于其他插件而隐藏的消息，被本插件错误地强制显示出来！

**代码表现形式：**
为了区分“本插件自动隐藏的消息”和“其实途径隐藏的消息”，本地引入了独有的 `hide_helper_hidden` 字段标记。

*   **执行隐藏操作时**（例如 `runIncrementalHideCheck` / `runFullHideCheck` 等处）：
    无论是循环还是迭代，只要本插件在做“隐藏消息”(`is_system = true`) 的动作，**都必须绑定注入专属标记**。
    ```javascript
    chat[idx].is_system = true; 
    chat[idx].hide_helper_hidden = true; // <-- 独家自定义标记
    ```

*   **执行显示操作时**（例如 `runFullHideCheck` 里的恢复逻辑，或全局重置 `unhideAllMessages` 等处）：
    只有包含专属标记的消息，才能被本插件翻转显示。**这是防止污染全局状态的底线。** 数据层和 DOM 层都必须只恢复本插件标记过的消息，不能再使用“把所有 `is_system=true` 的消息统一改回 false”的写法。
    ```javascript
    // 必须增加对 hide_helper_hidden === true 的联合判定
    if (chat[i] && chat[i].is_system === true && chat[i].hide_helper_hidden === true) {
        chat[i].is_system = false;
        delete chat[i].hide_helper_hidden; // <-- 恢复后需清理标记
    }
    ```

**🔴 合并注意事项：**
但凡上游对“遍历处理消息显示”相关的逻辑进行了修改，你都必须立刻检查是否遗漏了对 `hide_helper_hidden === true` 的保护性验证！**绝不可以使用上游原本缺少保护机制的方法直接覆盖。**

---

### 3. 数据迁移标记防误伤 (Data Migration Safety Defaults)
**功能目的：** 
上游作者在进行重大架构更新时（例如调整设置存储结构或底层原生接口联动），通常会加入数据迁移代码（例如 `migration_v1_complete`、`limiter_migration_v2_complete` 等标志位）。
但有时上游会在 `defaultSettings` 中将这些标志位默认赋值为 `true`。这会**引发致命的数据丢失**：旧用户更新合并版插件后，会被判定为“已完成迁移”，从而直接跳过迁移步骤，导致其原本辛苦配置的限制参数、记录等丢失失效！

**当前已知安全默认值（按当前代码语义）：**
```javascript
const defaultSettings = {
    // v1 旧角色/群组设置迁移：当前默认保持 true，避免重复执行旧迁移。
    migration_v1_complete: true,

    // Limiter v2 从 limiter_messageLimit 迁移到 power_user.chat_truncation：
    // 必须保持 false，保证旧用户能进入迁移分支。
    limiter_migration_v2_complete: false,
};
```

**代码表现形式：**
```javascript
const defaultSettings = {
    // ...
    // 上游此处可能是 true，导致老用户直接跳过老数据迁移。
    // 必须确保将其修改为 false，以触发我们的强制接管。
    limiter_migration_v2_complete: false, 
};
```

**🔴 合并注意事项：**
每次拉取上游更新时，无论上游的 `defaultSettings` 对象增加了什么带有 `migration`、`complete` 等字眼的布尔值开关，**必须逐个检查它们的真实语义。** 若上游逻辑是“false 才执行从旧版提取数据的迁移”，则我们在 `defaultSettings` 中应当将其显式定为 `false`，以保证受众能够平滑继承旧版设置。不要机械地把所有 migration 标志都改成 false，也不要盲目照搬上游的 true。

---

### 4. Rehide Fork 元数据与更新源保护 (Fork Metadata / Update Source)
**功能目的：**
本仓库是面向 Rehide fork 用户发布的版本，不是纯上游镜像。合并上游时，必须保留 fork 的发布身份、更新源和合并指导文档。

**必须保留的内容：**
```json
// manifest.json
"author": "uhhhh15,Ice_wilderness"
```

```javascript
// update.js
const REPO_ROOT = "https://raw.githubusercontent.com/Ice-wilderness/Rehide/main";
```

**🔴 合并注意事项：**
上游通常会把 `manifest.json` 的 `author` 改回 `uhhhh15`，并把 `update.js` 的远程仓库地址改回 `uhhhh15/hide`。这些改动对 Rehide 用户是不安全的：会导致插件内置更新检查跳回上游版本。合并时应采纳上游版本号和正常功能改动，但保留 Rehide 的 author 和更新源。

另外，`AI_MERGE_GUIDE.md` 是本仓库维护文件，上游没有该文件或删除该文件时，**不要接受删除**。

---

### 5. 样式文件编码与上游上传式提交 (CSS Encoding / Re-upload Diffs)
**功能目的：**
上游有时会以“删除后重新上传”的方式更新文件，导致 `style.css` 出现二进制冲突、编码乱码或大量行尾空白差异。盲目接受某一侧可能会造成中文注释乱码、样式丢失或 diff 极难审查。

**🔴 合并注意事项：**
若 `style.css` 出现 binary conflict 或中文乱码，优先选择可读的 UTF-8 文本版本作为基底，再确认本地是否有 Rehide 专属样式需要补回。对于上游新增的正常样式（例如日志下载按钮 `#hide-helper-download-log:hover`），应合入；但不要为了“解决冲突”引入大面积无意义格式化。

---

## 🛠️ 给 AI 助手的合并操作流 (Standard Operating Procedure for AIs)

当你接到带有“解决合并冲突”、“更新上游代码”、“Sync Fork”等任务时，请严格按如下流程进行：

1. **识别核心逻辑变更**：首先对比冲突文件中上游重构了哪些核心方法（如 `Logger` 的分离，或是轮询改成了事件监听），理解上游的意图。
2. **移植防冲突标**：无论上游如何改变修改消息体 `is_system` 的方法，不要照搬，需要把咱们的 `hide_helper_hidden` 添加及删除逻辑镶嵌进他们的新方法中。
3. **条件联检查漏**：全文搜索 `is_system = false`，排查所有的重置指令，确保 `if` 条件语句中依然包含了 `hide_helper_hidden === true`。
4. **查验默认配置**：检查新的获取 `Settings` 逻辑部分，是否有 `hideLastN: 6` 的初始值兜底。
5. **警惕默认值背刺**：在 `defaultSettings` 中，若发现上游新增或修改了带有 `migration`、`complete` 等字眼的布尔值，务必推理其真正含义。当前 `limiter_migration_v2_complete` 必须保持 `false`，但 `migration_v1_complete` 当前保持 `true`。
6. **保护 Fork 元数据**：确认 `manifest.json` 保留 `uhhhh15,Ice_wilderness`，`update.js` 保留 `Ice-wilderness/Rehide` 更新源，`AI_MERGE_GUIDE.md` 未被删除。
7. **处理编码/格式噪音**：如果上游上传导致 CSS 或 JS 出现大面积换行、行尾空白、乱码差异，先确认真实功能变更，再做最小必要清理。提交前运行 `git diff --check`。
8. **自我审查与提交**：提交代码（Commit）前，确认核心业务流程依旧遵循以上 Rehide 自定义机制。如果不确定，**请主动向使用者提出疑问并进行二次核对，切勿贸然提交。**
