# CHANGES — Bookmark Fixer Pro 修复版说明

修复版文件：`index_fixed.html`（基于仓库 `main` 分支的 `index.html` 逐项修改）
原始对照：`index.html`（仓库原样下载）

## 🔴 关键 Bug 修复

1. **新书签文件支持 HTML/JSON 了（原来只认 CSV 文本）**
   - `readFile` 对 src 上传做格式识别：JSON / HTML（Netscape 书签）自动解析并转为标准 CSV 填入文本框，再走原有修复流程。
   - `parseBookmarkJson` 升级为文件夹层级感知：一级文件夹 → 大类，二级文件夹 → 小类。
   - 新增 `bookmarksToCsv()` 负责对象 → CSV 转换。

2. **回收站操作后「历史对比」视图不再过期**
   - 新增统一入口 `afterBookmarksChanged()`（重建面板 + 刷新对比 + 持久化），`moveToTrash` / `restoreFromTrash` / `batchRecoverToNew` / `manualAddToNew` / `manualAddToOld` / `commitFixSuspiciousLink` / 批量自动修复 / 死链批量删除 / 编辑弹窗等全部改走该入口，各面板状态永远一致。

3. **URL 清洗管线，杜绝脏链接**
   - 新增 `cleanUrl()`：剥离尾随全角/半角标点、不成对括号、`（官网）` 这类成对括号内容。
   - `URL_REGEX` 收紧：不再吞掉中文、全角括号、全角标点。
   - 解析、去重、匹配、测活、导出全链路复用清洗后的 URL。

4. **下载前体检：占位链接不再静默写入书签文件**
   - `downloadResult` 检测未修复的 `about:blank` / `[可疑链接已删除]` 条目，弹出三选：「排除占位链接并下载 / 全部包含下载 / 取消」。

5. **「旧备份独有」不再被子串匹配误杀**
   - 只当规范化 URL 相同或标题完全一致时才判定已存在，去掉原来的 `includes()` 双向子串判断；`GitHub` 与 `GitHub Desktop` 这类真实删除项现在能正常显示。

6. **批量自动修复阈值统一**
   - 新增常量 `SUSPICIOUS_MATCH_THRESHOLD = 0.35`，面板「可追溯」与「批量自动修复」共用，勾选了就不会再被静默跳过。

## 🟡 次要问题修复

7. 统计卡片类名笔误 `stat-lbl` → `stat-badge`（样式生效）。
8. 删除死代码 `finalHtml` 与 `generateBookmarkHtml`。
9. 表头过滤改为 `大类,小类` 精确前缀，不再误伤「分类…」开头的真实数据行。
10. 「解除绑定」后旧条目回填到「旧备份独有」、新条目回填到「当前新增」，不会再凭空消失。
11. 编辑弹窗改名「修改书签信息」：新增**网址**字段（含 http(s) 校验），且不再用新标题覆盖 `originalTitle`（保留匹配语义）。
12. CSV 全部按标准转义（引号翻倍），`parseCsvLine` 支持带引号/含逗号标题的解析，导出的 CSV/TXT 可无损回导。
13. 重复网址列表显示标题，方便确认该删哪条。
14. `manualAddMatch` 从 `prompt()` 改为自定义 Modal，并做 URL 清洗；Modal 支持 Esc / 点击遮罩取消。
15. 死链检测：
    - https 页面下 `http://` 站点不再误判死链（混合内容无法探测，单独计数提示）；
    - 去掉不可靠的 `Failed to fetch` 消息判断（统一走 no-cors 兜底重试）；
    - 修复剩余时间在刚开始时可能显示 0/Infinity 的问题。
16. 状态持久化：书签 / 回收站 / 旧档 / 白名单 / 死链记录自动存入 localStorage，刷新页面自动恢复；预览栏新增「清空本地缓存」按钮。
17. 性能：对比前预计算 `normalizeUrl` 缓存；相似度滑块输入加 150ms 防抖。

## 🔒 安全加固（新增）

18. 拒绝 `javascript:` / `data:` / `vbscript:` 链接进入最终书签：
    - `commitFixSuspiciousLink`（可疑链接修复）：显式拦截并提示；
    - `batchRecoverToNew`（批量挽回）：非 http(s) 链接跳过并计数提示；
    - `manualAddToNew`（单条挽回）：非 http(s) 链接直接拒绝。
    - 防止恶意书签内容以可执行 URL 形式进入导出的书签文件。

## 🐛 复查补充修复（v1.1）

19. **会话恢复后「待修复链接」列表为空**：`suspiciousBookmarks` 是派生数组，`loadState()` 恢复后未重建，导致恢复会话后可疑项标签页显示"没有待修复链接"。已在 `DOMContentLoaded` 恢复分支中补 `suspiciousBookmarks = processedBookmarks.filter(b=>b.isSuspicious)`。

> ⚠️ 注意：若 `index_fixed.html` 已上传到仓库，请重新上传包含第 19 条修复的最新版本（或直接使用同目录下正确命名的 `index.html`）。

## 验证

- `node --check` 语法检查通过。
- 25 项单元测试全部通过（URL 清洗、正则边界、CSV 解析/转义、JSON 层级映射、行解析链路、非 http(s) 拒绝规则、saveState/loadState 往返与字段剔除、可疑数组重建）。

## 使用

- 本地使用：直接用浏览器打开 `index.html`（修复版副本）或 `index_fixed.html`。
- 部署到仓库：**应命名为 `index.html`**（GitHub Pages 默认服务 index.html，README 也指向 index.html）。不要把修复版继续叫 `index_fixed.html` 而删掉 `index.html`。

## 仓库层面待办（截至 2026-08-14 上传后）

- ✅ LICENSE / SECURITY.md 已就位（GitHub 已识别 MIT）。
- ⚠️ 仓库里 `index.html` 被删除，只剩 `index_fixed.html` + `index_old.html`：请把修复版改名为 `index.html` 上传（或直接上传本目录的 `index.html`），并决定 `index_old.html` 去留（建议删除，避免用户混淆；要留请在 README 注明是旧版存档）。
- ⚠️ GitHub Pages 未启用，但 README 写着"或访问在线 GitHub Pages 链接"：请在 Settings → Pages 选择 Deploy from branch（main / root），或删掉 README 里这句话。
- 可选：README 中「异步多线程」措辞改为「并发嗅探」更准确。
