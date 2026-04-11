# 🎨 Rehide - 自定义功能及合并指南 (AI Merge Guide)

由于本仓库是由上游项目（`uhhhh15/hide` 等）复刻（Fork）并定制的，本地包含了部分非常强依赖且独特的“自定义功能”。随着上游架构的频繁更新，代码合并时经常产生冲突。

**⚠️ 请后续接手代码合并（Sync / Merge）的 AI 助手或开发者，务必仔细阅读本指南！** 在处理冲突时，**绝对不能盲目采用上游的代码覆盖本地**，必须将以下自定义逻辑完美地融合（移植）到上游的新代码结构中！

---

## 🌟 核心自定义功能列表

### 1. 角色模式下默认隐藏 6 楼 (Default Hide 6-layers in Role mode)
**功能目的：** 在角色对话模式下，如果没有读取到用户的历史隐藏配置（比如第一次加载，或者清理了缓存），为了保持页面清爽，功能应当默认生效并将最顶部的 6 条消息隐藏（`hideLastN: 6`）。

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
上游如果重构了设置和缓存的管理方式（比如使用了新的 `Settings Manager` 或全局 `State`），请务必在初始化设置的地方注入该默认条件。

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
    只有包含专属标记的消息，才能被本插件翻转显示。**这是防止污染全局状态的底线。**
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
每次拉取上游更新时，无论上游的 `defaultSettings` 对象增加了什么带有 `migration`、`complete` 等字眼的布尔值开关，**必须检查它们的默认初始值是否安全。** 若上游逻辑是“false 才执行从旧版提取数据的迁移”，则我们在 `defaultSettings` 中应当将其显式定为 `false`，以保证你的受众能够平滑继承旧版设置，**切忌盲目照搬上游的 `true` 导致老用户数据暴毙。**

---

## 🛠️ 给 AI 助手的合并操作流 (Standard Operating Procedure for AIs)

当你接到带有“解决合并冲突”、“更新上游代码”、“Sync Fork”等任务时，请严格按如下流程进行：

1. **识别核心逻辑变更**：首先对比冲突文件中上游重构了哪些核心方法（如 `Logger` 的分离，或是轮询改成了事件监听），理解上游的意图。
2. **移植防冲突标**：无论上游如何改变修改消息体 `is_system` 的方法，不要照搬，需要把咱们的 `hide_helper_hidden` 添加及删除逻辑镶嵌进他们的新方法中。
3. **条件联检查漏**：全文搜索 `is_system = false`，排查所有的重置指令，确保 `if` 条件语句中依然包含了 `hide_helper_hidden === true`。
4. **查验默认配置**：检查新的获取 `Settings` 逻辑部分，是否有 `hideLastN: 6` 的初始值兜底。
5. **警惕默认值背刺**：在 `defaultSettings` 中，若发现上游新增或修改了带有 `migration`、`complete` 等字眼的布尔值，务必推理其真正含义。如果是为了让旧用户平滑过渡，这必须显式设为 `false`（防数据暴毙）。
6. **自我审查与提交**：提交代码（Commit）前，确认核心业务流程依旧遵循以上三点核心自定义机制。如果不确定，**请主动向使用者提出疑问并进行二次核对，切勿贸然提交。**
