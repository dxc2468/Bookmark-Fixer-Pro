# 安全策略 (Security Policy)

## 项目定位

Bookmark Fixer Pro 是一个 **100% 纯前端、零后端** 的本地浏览器工具：所有解析、去重、对比、测活、导出均在浏览器内存中完成，**没有任何数据上传到服务器**。用户数据只可能存在于：

- 浏览器内存 / `localStorage`（本机浏览器，未加密，请勿在多用户共享设备上使用）；
- 用户主动点击下载导出的文件。

## 支持的版本 (Supported Versions)

本项目以单文件 `index.html` 形式发布。安全维护策略：

| 版本 | 支持状态 |
| ---- | -------- |
| 最新 main 分支 / 最新 Release | ✅ 受支持 |
| 更早的历史版本 | ❌ 不再维护，请升级 |

请始终使用 GitHub Releases 或 `main` 分支最新的 `index.html`。

## 报告漏洞 (Reporting a Vulnerability)

**首选方式：GitHub 私有安全通告（Security Advisories）**

1. 打开仓库页面 → **Security** 选项卡 → **Report a vulnerability**；
2. 填写影响描述（组件、触发条件、复现步骤、影响评估）。

**备选方式：GitHub Issues**（公开可见，请勿在其中粘贴含敏感信息的书签数据；仅描述漏洞性质）。

**响应承诺**

- 3 个工作日内确认收到报告；
- 7 个工作日内给出初步评估与修复计划；
- 严重问题（如 XSS 执行、数据泄露）将在修复后尽快发布，并同步更新本文件。

## 安全模型与已知考虑

本工具把「用户上传的书签内容」视为**不可信输入**（AI 生成的 CSV、从社区复制的文本、历史备份 HTML/JSON 均可能被恶意构造）：

- ✅ **已加固**：所有插入 DOM 的文本均经 `escapeHtml()` 转义，防止书签内容触发 HTML 注入/XSS；
- ✅ **已加固**：`javascript:` / `data:` / `vbscript:` 等非 http(s) 链接在「可疑链接修复」「历史挽回」「手动新增」等入口被拒绝，不会进入最终导出的书签文件（见 `index_fixed.html`）；
- ✅ **已加固**：下载书签文件前会体检未修复的 `about:blank` 占位链接，避免脏数据写入；
- ⚠️ **外部依赖**：页面通过 CDN 加载 Tailwind CSS 与 Font Awesome。CDN 被攻陷或不可用时可能影响样式与交互（不涉及用户数据）。对安全要求严格的部署，建议自托管这两个静态资源；
- ⚠️ **数据隐私**：工具本身不联网上报，但请勿把含 `token=`、`session_id=`、内网/密码链接的原始书签发给第三方 AI——相关提示已写入 README；
- ⚠️ **localStorage**：会话数据（书签/回收站/旧档/白名单）保存在浏览器 `localStorage`，同一浏览器其他页面/扩展程序可读取，敏感书签请在共享设备上使用后点击「清空本地缓存」。

## 漏洞面参考（供开发者自查）

- `parseBookmarkHtml` / `parseBookmarkJson` / `doProcess` 对不可信文本的解析路径；
- 所有 `innerHTML` 拼接点（必须经 `escapeHtml`）；
- 死链检测的 `fetch` 与「打开选中链接」的 `window.open`（仅允许 http/https）；
- localStorage 读写与「清空本地缓存」行为。
