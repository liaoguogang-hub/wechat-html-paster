# wechat-html-paster

> HTML → 微信公众号 · 内联样式粘贴 + 可视化编辑器（单文件，零依赖，浏览器直接打开即用）

把任意 HTML 文章（含 `<style>`、CSS 变量、`@media`）转换成**全内联样式**的公众号兼容 HTML，并提供 Word 式可视化编辑工具条，改完一键复制进 `mp.weixin.qq.com` 编辑器。

**当前版本：v2.0.0**（版本历史见 [CHANGELOG.md](./CHANGELOG.md)，原始版本存档于 `versions/v1.0.0.html`）

---

## 快速使用

1. 浏览器打开 `index.html`（Chrome / Edge 推荐）。
2. 左栏「① 输入 HTML」：粘贴 HTML / 点「载入样例」/「选择 .html 文件」。
3. 点「**转换并适配公众号**」：在 578px 沙箱里用浏览器原生 CSS 引擎跑一遍层叠计算，展开 `var()`、走通 `@media`，再按公众号白名单拍平成内联样式。
4. 右侧预览区直接编辑：选中文字 → 顶部**吸顶功能区**（Word 式布局，滚动不消失）改字体 / 字号 / 颜色 / 行距 / 段距 / 段落背景 / 边框 / 卡片 / 主题色。
5. 点「**复制（带格式）**」→ 去公众号后台 Ctrl+V。或「下载 .html」。

---

## 功能区布局（v2 复刻 Word）

```
┌─ QAT 快捷栏 ──────────────────────────────────────────────┐
│ 复制(带格式)  下载  │ ↶撤销 ↷重做 │      预览375/578  ⌃收起 │
├─ 功能区 ──────────────────────────────────────────────────┤
│ [字体] [段落] [插入] [卡片] [主题]  （分组名在组底部，Word 式）│
├─ 状态栏（固定高度，不随载入/选区变化） ─────────────────────┤
│ 操作状态 ······························ 当前选区信息        │
└───────────────────────────────────────────────────────────┘
```

| 分组 | 控件 |
|---|---|
| **字体** | 字体、字号、B / I / U、清除格式、字色（含色板）、文字背景（含「无」） |
| **段落** | 左/中/右/两端对齐、首行缩进、行距、段距、**段落背景（含「无」）**、段落边框（无/左侧强调线/全边框 + 边框色） |
| **插入** | 分割线、引用卡片 |
| **卡片** | 卡片背景（含色板 + 「无」）、卡片边框（色 + 2/3/4px + 「无」） |
| **主题** | **主题色**（11 预设 + 自定义 + 自动识别 + 当前色 chip）、移除全部字体、整篇底色、主体底色 |

---

## 主题色（一键全局换色）

复刻 [md.nihaotk.com](https://md.nihaotk.com/) 的「主题色」设定：

1. **转换后自动识别**：对标题（h1-h4）、加粗、表头 `th`、引用块的 color / border / background 做加权投票，找出全文品牌色（黑/白/灰等中性色不参与）。
2. **点任意预设色**（经典蓝 / 翡翠绿 / 活力橘 / 柠檬黄 / 薰衣紫 / 天空蓝 / 玫瑰金 / 橄榄绿 / 石墨黑 / 雾烟灰 / 樱花粉）或自定义取色器，即把全文**所有命中品牌色的地方**（标题色、标题下划线、表头底色、引用边框、强调色，含同 RGB 不同透明度的浅色底）一次性替换。
3. 「识别」按钮可重新探测；「当前」chip 显示当前主题色。
4. 替换前自动快照，Ctrl+Z 可撤销。

---

## 公众号《内容结构检测》合规

对照微信官方规范 [《公众号插件内容规范》](https://developers.weixin.qq.com/doc/service/guide/product/plugin_spec.html)（结构校验开源实现：[verify-article-structure-spec](https://github.com/wechatjs/verify-article-structure-spec)），本工具在转换/导出时自动处理：

| 规范条目 | 问题 | 本工具的处理 |
|---|---|---|
| 1.6 text-align | `text-align: start / end` 触发「内容结构检测」告警（v1 的主要成因：`getComputedStyle` 默认返回 `start`） | 内联化时**直接丢弃** start/end（LTR 下等价 left） |
| 1.8 pre | `white-space: pre` 移动端横向截断 | 自动改写为 `pre-wrap` |
| 1.3 line-height | 行高小于字号叠字 | 行距控件最小 1.5；`normal` 一律不内联 |
| 1.4 width / 1.5 height | 固定宽/高溢出 | width/height/max/min 全部剥离，不内联 |
| 2.1 嵌套层级 | 同标签同样式单子节点链 >10 层被编辑器自动精简并告警 | **复制/下载前自动扁平化**：合并同样式 span 链、解包冗余 span、删除空 span |
| 3 字体 | 自定义 font-family 破坏跨端一致性 | 「移除全部字体」按钮一键清空全文 font-family，跟随公众号默认字体栈 |
| 1.1 opacity / 1.2 caret-color | 透明图/透明光标 | 白名单不含这些属性，天然不会输出 |
| 输出体积 | 内联样式冗余导致粘贴卡顿 | 裁掉 no-op 声明（0 宽边框、无背景图时的 background 长写、全 0 margin/padding、非列表 list-style 等），样例由 32.3 KB → **8.9 KB（−72%）**，关键样式有保真断言 |

> 提示：粘贴后若仍出现结构提示，多为图片缺少 `data-w` 属性（规范 1.4），与文本排版无关。

---

## 修复记录摘要（v1 → v2）

1. **段落背景归位**：从「底色行」移入「段落」分组，与行距/段距/对齐/段落边框同组。
2. **工具栏载入后变大**：真凶是**文章 CSS 泄漏**（v1 把 `<style>` 注入本页 div，`body{padding}`、`*{margin:0}` 等把工具栏重新排版），次要原因是 `selinfo` 提示条撑高、动态下拉加宽 —— v2 用 **iframe 沙箱隔离样式** + **固定高度状态栏** + **固定宽度下拉框** + **稳定栅格/常驻滚动条**，载入前后测量值完全一致（见 `test-layout.html`）。
3. **卡片线条色**：v2 新增「卡片边框」控件（颜色 + 宽度 + 移除），作用于 `findCard()` 找到的**卡片容器本身**。
4. **色块无法删除**：段落背景 / 卡片背景 / 卡片边框 / 文字背景全部提供「无」按钮，语义是**移除属性**（而非写 transparent），输出更干净。
5. **边框定位到段落**：v1 `findCard` 会把普通段落上方的 DIV（主体容器）误判为卡片 —— v2 重写：BLOCKQUOTE/ASIDE 直接命中，SECTION/DIV 需自带可见背景或边框，并跳过根节点与主体容器。
6. **Word 式功能区**：QAT 快捷栏 + 分组功能区（组名置底）+ 状态栏 + 折叠按钮。
7. **主题色**：见上节。
8. **版本管理**：git 仓库 + 标签（`v1.0.0` / `v2.0.0`）+ CHANGELOG + 版本徽章（页面右上角）。

---

## 项目结构

```
wechat-html-paster/
├── index.html            # v2.0.0 主程序（单文件，直接打开）
├── README.md             # 本文件
├── CHANGELOG.md          # 版本历史
├── test-harness.html     # 功能回归测试（23 项断言）
├── test-layout.html      # 布局稳定性 / CSS 隔离回归测试（11 项断言）
└── versions/
    └── v1.0.0.html       # v1 原始版本存档（来自用户上传）
```

## 开发说明

- 零依赖、零构建：纯原生 HTML + CSS + JS，单文件约 1000 行。
- 转换管线：`DOMParser` 解析 → 沙箱（`#sandbox`，578px，屏幕外）注入原 `<style>` → `getComputedStyle` 逐元素取白名单属性 → 拍平内联 → 包一层 `<section style="body样式">` → `sanitize`（去 class/id）。
- 撤销/重做：innerHTML 快照栈（上限 80 步），`beforeinput` 打点。
- 编辑保焦：工具条所有按钮 `mousedown` 阻止默认，不抢预览区选区。
- 预览宽度切换：375px（移动端模拟）/ 578px（默认）。

## 转换管线与样式隔离（重要）

```
原文 HTML ──► DOMParser 取 <style>
                    │
                    ▼
        ┌───────────────────────────────┐
        │ 隐藏 iframe 沙箱（578px 宽）  │  ← 文章 CSS 只作用于这里
        │ <style> + body 内容           │     不会污染本工具界面
        └───────────────────────────────┘
                    │ iframe.contentWindow.getComputedStyle
                    ▼
        白名单属性 → 内联样式（含合规修正：text-align/white-space）
                    │ document.importNode + sanitize（去 class/id）
                    ▼
        <section style="body 样式"> … </section> ──► 预览 / 复制 / 下载
```

> **v1 的严重缺陷**：它把文章 `<style>` 注入到工具页面内的一个 div 里，文章样式（`body{padding}`、`*{margin:0}`、`body{background}` 等）**会同时作用于工具自身**，导致载入文章后工具栏被重新排版、宽度变化、功能区换行错位 —— 即用户反馈的「工具栏莫名其妙增大」。v2 用 iframe 沙箱彻底隔离，并用 `test-layout.html` 做了回归断言。
>
> 另一处相关修正：输出端过滤 `STYLE/SCRIPT/LINK/META/TITLE/BASE/NOSCRIPT/TEMPLATE`，避免正文内嵌的 `<style>` 混进公众号输出。

## 测试

零依赖，用系统自带浏览器（Edge/Chrome）headless 跑：

```bash
# 功能回归（23 项）：转换、合规修正、主题色、卡片/段落边框、扁平化、字体剥离、撤销
msedge --headless=new --disable-gpu --allow-file-access-from-files \
       --virtual-time-budget=9000 --dump-dom file:///<绝对路径>/test-harness.html

# 布局稳定性 + CSS 隔离回归（11 项）
msedge --headless=new --disable-gpu --allow-file-access-from-files \
       --virtual-time-budget=9000 --dump-dom file:///<绝对路径>/test-layout.html
```

两个页面把结果以 `PASS/FAIL` 打在 `<pre id="results">` 里（`--dump-dom` 即可读到）。

## 版本管理

- Git 仓库本地初始化，提交历史即版本历史：
  ```bash
  git log --oneline
  git tag            # v1.0.0  v2.0.0
  ```
- 发新版本流程：改 `index.html` → 更新 `CHANGELOG.md` → 提交 → `git tag vX.Y.Z` →（可选）`cp index.html versions/vX.Y.Z.html` 存档。
