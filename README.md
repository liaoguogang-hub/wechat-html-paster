# wechat-html-paster

> 把任意 HTML 一键转成「可直接带格式粘贴到微信公众号后台编辑器」的内联化 HTML，并在浏览器里做局部可视化编辑。

单文件 HTML 工具，零依赖、零安装。打开页面 → 粘贴 HTML / 选文件载入 → 转换 → 在预览区里像 Word 一样改 → 一键复制，粘到 `mp.weixin.qq.com` 的公众号后台即可保留排版。

## 为什么需要这个

微信公众号编辑器的粘贴逻辑很挑：

- 会**过滤掉** `<style>` / `class` / `id`，所以 CSS 只能写在每个标签的 `style="..."` 上（"内联化"）。
- 只支持**基础标签**（`section` / `p` / `strong` / `blockquote` / `figure` / `img` 等），多余 class 全部剥掉。

GitHub 上主流的「Markdown → 公众号」工具（[doocs/md](https://github.com/doocs/md)、[xiaolin-paper](https://github.com/xiaolinbaba/xiaolin-paper) 等）都不接受 HTML 输入，而真实素材常常就是 HTML。本工具填补这条空白：**HTML 进 → 公众号内联 HTML 出**。

## 工作原理

1. 把用户输入的 HTML 塞进一个 **578px 宽的隐藏 `<section>` 沙箱**（公众号正文宽度）。
2. 让浏览器原生 CSS 引擎渲染（`getComputedStyle`），自动展开 `var()`、走通级联与 `@media`。
3. 按公众号**白名单**遍历每个节点，把计算后的样式**拍平成内联 `style`**——`font-family`、字号、颜色、背景、行距、段距、对齐等只挑白名单允许的属性。
4. 把根 `<section>` 上 `body` 的字体声明继承下去（`juice` / 手写工具不会自动内联继承值）。
5. 「复制（带格式）」用 Clipboard API 写 `text/html` + `text/plain` 双轨，失败降级到 `execCommand('copy')`。

完全在浏览器里跑，不上传任何内容。

## 主要功能

### 转换
- HTML 文本粘贴 / 选择本地 `.html` 文件 / 一键载入样例
- 在 578px 沙箱中按公众号白名单**内联化**

### 可视化编辑（预览区 contenteditable）
- **字符级**：字体、字号（12–28px）、字色、文字背景、**B / I / U**
- **段落级**：行距、段距、4 种对齐、段落背景
- **插入**：分割线 `<hr>`、引用卡片
- **三层底色**（避免"改了等于没改"）：
  - 整篇底色 = 页面层 + 主体层
  - 主体底色 = 内容卡片层
  - 卡片底色 = 引用块 / 信息卡
- **选区回显**：在工具栏实时显示当前字体 / 字号 / 行距 / 对齐

### 工程化体验
- **吸顶工具条**：滚到长文底部也能继续改
- **撤销 / 重做**：自管历史栈（容量 80），`Ctrl+Z` / `Ctrl+Y` / `Ctrl+Shift+Z`；不依赖浏览器原生 undo
- **标题滚动折叠**：下滚自动收标题
- **左栏吸顶**：随时换文章 / 重新转换
- **Word 式功能区折叠**：一键收起格式化区
- **Range API 实现**：避开 `execCommand`（会被公众号吃掉 `<font>` 改写）

## 用法

1. 双击 `wechat-html-paster.html`，或在 Chrome / Edge 里打开。
2. 左栏粘 HTML（或点「载入样例」）。
3. 点「转换并适配公众号」。
4. 右栏预览区里选中文字，用顶部吸顶工具条调整。
5. 点「复制（带格式）」，粘到公众号后台。

## 兼容性

- 现代 Chromium 内核（Chrome / Edge / Brave）最佳
- Firefox 基本可用（Clipboard API 写入需要权限授予）
- Safari / 移动端浏览器部分能力受限（剪切板、文件选择等）

## 项目说明

这是一个**单文件静态工具**，全部逻辑都在 `wechat-html-paster.html` 里（含 HTML / CSS / JS）。不需要 Node、npm 或构建工具。

配套工具（如果要把一整个真实 HTML 文件批量转成可粘贴公众号的成品）：见 `cxo-players-wechat.html` 的 Node 转换脚本思路（`juice` 内联 + `background` 归一化 + 喂给浏览器前把 `"` 换成 `'`）。

## License

MIT