# lyrics — 多语言歌词展示站（纯 Hugo）

歌词直接写在歌曲页正文里，由 `lyrics` shortcode 在 **Hugo 模板内**解析 `汉字{注音}` 紧凑标记并渲染成 `<ruby>` 注音。**零外部脚本**，只有 Hugo。

## 快速开始

```powershell
hugo server          # 预览 http://localhost:1313
hugo --gc --minify   # 生产构建 → public/
```

歌词与元数据都在 `content/` 里，Hugo dev server 会监听——**改歌词保存即自动重建刷新**。

## 目录结构

| 路径 | 说明 |
|---|---|
| `content/songs/<slug>.md` | 每歌一页：front matter 元数据 + 正文 `{{< lyrics >}}` 歌词块 |
| `layouts/shortcodes/lyrics.html` | 核心：模板内解析 `{注音}` → ruby，含格式校验与三种复制文本 |
| `layouts/songs/single.html` | 歌曲页外壳 + 复制按钮脚本 |
| `layouts/index.json` | 全站搜索索引模板（home 的 JSON 输出 → `public/index.json`） |
| `content/search.md` | 搜索页 `/search/` |
| `static/css/style.css` | 样式（含 ruby 注音排版） |
| `docs/技术方案.md` | 设计与约定文档 |

## 歌词标注语法

在歌曲页正文的 `{{< lyrics >}}…{{< /lyrics >}}` 内书写：

- 一行 = 歌词显示的一行；空行 = 段落间隔。
- **汉字串紧跟 `{注音}`**，注音是该汉字串的整体读音（模板不做任何读音判断）。
- 只标汉字，词内送假名不标：`食{た}べる`
- 整词连标（含熟字訓 / 当て字）：`苦虫{にがむし}`、`明日{あした}`
- 数字+汉字组合若读音跨越两者，改写为汉字：`1人` → `一人{ひとり}`
- 非日语歌词（韩/中/英）按纯文本直出，不允许出现 `{}`。

校验（违反即构建报错并指出行号）：花括号不配对、注音含非假名、`{注音}` 前面不是汉字。
提示（WARN 不阻断）：某段含汉字但没注音（疑似漏标）。

## 加一首歌（两步）

1. 新建 `content/songs/<slug>.md`，复制 `arpeggio.md` 的 front matter，填题名/歌手/语言；
2. 在正文写 `{{< lyrics >}}` 与标注好的歌词、`{{< /lyrics >}}`，跑 `hugo server` 预览校对。

## 页面功能

- 复制工具栏：**原文 / 纯假名 / 带注音** 三种文本（模板从段结构自拼，非复制 DOM）。
- 站内搜索：`/search/` 输入即搜——构建时生成 `index.json`（元数据 + 去注音原文 + 假名全文），前端原生 JS 检索并显示命中行，无第三方依赖。
- 注音开关、GitHub Pages 部署：见 `docs/技术方案.md` 的演进预留与里程碑。

版权：歌词归原作者/唱片公司所有，本站仅学习展示。
