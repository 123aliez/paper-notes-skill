# Zotero 7+ Plugin Development Notes

> Based on analysis of Better Notes v3.0.6 (working on Zotero 9.0.3, unsigned)

## manifest.json — 正确格式（对齐 Better Notes）

```json
{
  "manifest_version": 2,
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "Description.",
  "author": "Author",
  "homepage_url": "",
  "icons": {
    "48": "chrome/content/icon.png",
    "96": "chrome/content/icon.png"
  },
  "applications": {
    "zotero": {
      "id": "plugin-id@zotero.org",
      "strict_min_version": "8.0-beta.21",
      "strict_max_version": "9.99.99"
    }
  }
}
```

**关键点：**
- `manifest_version` 必须是数字 `2`，不是字符串 `"2"`
- 不要加 `chrome.manifest` 字段（Zotero 7+ 不用）
- 不要加 `locales`、`content_scripts` 字段（那是 Firefox 格式）
- `strict_min_version: "8.0-beta.21"` 覆盖 Zotero 7.0+ 正式版
- `strict_max_version: "9.99.99"` 覆盖 Zotero 9.x 全系列
- `update_url` 可省略或为空字符串
- **Zotero 9 接受未签名插件**（Better Notes 从 GitHub releases 直接安装，未签名，正常工作）

## bootstrap.js — 正确结构

```js
var chromeHandle;

function install(data, reason) {}

async function startup({ id, version, resourceURI, rootURI }, reason) {
  await Zotero.initializationPromise;

  // 必须用 aomStartup.registerChrome 注册 chrome
  var aomStartup = Components.classes[
    "@mozilla.org/addons/addon-manager-startup;1"
  ].getService(Components.interfaces.amIAddonManagerStartup);
  var manifestURI = Services.io.newURI(rootURI + "manifest.json");
  chromeHandle = aomStartup.registerChrome(manifestURI, [
    ["content", "pluginref", rootURI + "chrome/content/"],
  ]);

  // 加载主脚本
  Services.scriptloader.loadSubScript(
    `${rootURI}/chrome/content/scripts/main.js`
  );
}

function onMainWindowLoad({ window: win }) {}
function onMainWindowUnload({ window: win }) {}

function shutdown({ id, version, resourceURI, rootURI }, reason) {
  if (reason === APP_SHUTDOWN) return;
  if (chromeHandle) {
    chromeHandle.destruct();
    chromeHandle = null;
  }
}

function uninstall(data, reason) {}
```

**关键点：**
- Chrome 注册在 bootstrap.js 中通过 `aomStartup.registerChrome()` 完成，不是在 manifest.json
- `await Zotero.initializationPromise` 确保 Zotero 初始化完成
- 主脚本通过 `Services.scriptloader.loadSubScript()` 加载
- shutdown 中必须清理 chromeHandle

## 目录结构

```
addon.xpi (改名 .zip 可查看内容)
├── manifest.json
├── bootstrap.js
└── chrome/
    └── content/
        ├── icon.png (48x48 或 96x96)
        └── scripts/
            └── main.js (插件逻辑)
```

## Zotero Notifier 事件

```js
Zotero.Notifier.registerObserver(handler, ["item"], "uniqueID");
// handler.notify(event, type, ids, extraData)
```

- API 创建笔记 → 触发 `add` 事件
- API 修改笔记 → 触发 `modify` 事件
- **必须同时监听 `add` 和 `modify`**

## Better Notes API

```js
// 确认 Better Notes 已加载
Zotero.BetterNotes?.api?.convert?.md2html

// Markdown → HTML
const html = await Zotero.BetterNotes.api.convert.md2html(markdownContent);
```

## 安装路径

- **不是** Zotero 数据目录（存 zotero.sqlite 和附件）
- 通过 `about:support` →「配置文件夹」获取正确路径
- Windows 默认：`%appdata%\Zotero\Zotero\Profiles\<random-string>\`
- 插件放在 `<profile>/extensions/<extension-id>.xpi`
- 数据目录通过 设置 → 高级 → 文件和文件夹 → 数据存储位置 查看

## 常见错误（从失败中总结）

| 错误 | 原因 | 解决 |
|------|------|------|
| "无法与该版本的 Zotero 兼容" | manifest.json 格式不对 | 对齐 Better Notes 格式，去掉多余字段 |
| `manifest_version` 写成字符串 | `"2"` → `2` | 用数字 |
| 加了 `chrome.manifest` | Zotero 7+ 不用 | 删除该字段 |
| 加了 `locales`/`content_scripts` | Firefox 格式，Zotero 7+ 不用 | 删除 |
| bootstrap.js 没注册 chrome | 需要 `aomStartup.registerChrome()` | 加上注册代码 |
| Telegram 传输 xpi 文件损坏 | 二进制文件被损坏 | 本地生成 xpi，不要通过聊天传输 |
| `ChromeUtils.import() has been removed` | Zotero 9 基于 Firefox 128+，已废弃此 API | 改用 `importESModule()`，或不 import 任何模块（bootstrap 插件可直接访问 `Zotero`、`Services`、`Components` 全局对象） |
| 插件在 Zotero 中显示已启用但无任何 `[plugin-name]` 日志输出 | bootstrap.js startup() 函数在执行第一行代码前就报错（通常是 `ChromeUtils.import()` 导致） | 在 startup() 第一行加 `Zotero.debug("[plugin] STARTUP CALLED")` 确认函数被调用；逐行加 debug 定位错误行 |

## Zotero 9 特有注意事项

1. **`ChromeUtils.import()` 已移除** — Zotero 9 基于 Firefox 128+，所有 `ChromeUtils.import()` 调用会报错并阻止后续代码执行。日志中表现为 `[JavaScript Error: "ChromeUtils.import() has been removed. Use importESModule()"]`。修复：不 import 任何模块，直接使用全局对象（`Zotero`、`Services`、`Components`、`Components.classes` 等），这些在 bootstrap 插件上下文中都是可用的。
2. **调试方法**：在 bootstrap.js 的 startup() 函数中，第一行就加 `Zotero.debug("[plugin-name] STARTUP CALLED")`，每一步关键操作都加 debug 输出。在 Zotero → 帮助 → 调试输出查看器中搜索插件名称，确认每一步是否执行。
3. **清理旧日志**：测试前先清空调试输出（帮助 → 调试输出 → Clear），重启 Zotero，再操作，确保日志是最新的。

## 已验证：Notifier Observer 在插件中可能不触发

**2026-05-20 实测结果：** 插件 manifest.json 格式正确（对齐 Better Notes），bootstrap.js 正确注册 chrome，插件在 Zotero 中显示为已启用（工具 → 插件 → Markdown AutoConvert → On）。但 `Zotero.Notifier.registerObserver` 注册的回调**从未触发**——无论是通过 API 创建笔记还是手动创建笔记。

**可能原因：**
1. `Services.scriptloader.loadSubScript()` 加载的脚本可能在不同的作用域，Notifier observer 注册在了错误的上下文
2. Zotero 7+ 的 Notifier 系统可能有变化，`["item"]` 类型的事件不覆盖笔记创建
3. `isNote()` 方法可能在 Zotero 7+ 中不存在或返回值不同

**结论：** 通过插件拦截 API 写入的笔记并自动转换 Markdown → HTML 的方案**当前不可行**。即使插件加载成功，Notifier 也无法可靠地捕获 API 创建的笔记事件。

**当前唯一可靠方案：** API 写入 HTML 版本 + 同时发 Markdown 版本给用户手动粘贴到 Better Notes 编辑器。
