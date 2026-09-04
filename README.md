# lyrics — 多语言歌词展示站（Hugo + Blowfish + Passthrough ruby）

歌词以**普通 Markdown** 写在歌曲页正文，注音用行内 Passthrough 语法 `{文字|注音}`，
由渲染钩子输出任意字符/长度的 `<ruby>`；外观使用 **Blowfish** 主题。
没有自定义解析器、没有复制按钮脚本——选中即复制。

## 快速开始

```powershell
# 安装 Blowfish（固定提交 + 强制 LF，与 CI 完全一致；见「版本同步」）
git -c core.autocrlf=false clone https://github.com/nunocoracao/blowfish.git themes/blowfish
git -C themes/blowfish checkout 7ab6a75d42e0131f51a28fd361656e7a89e0a8d6

hugo server          # http://localhost:1313
hugo --gc --minify   # 生产构建 → public/
```

搜索：**Ctrl+K**（Blowfish 内置 Fuse，索引 = 去注音原文 + 注音）。

## 目录结构

| 路径 | 说明 |
|---|---|
| `config/_default/*.toml` | Blowfish 配置；`markup.toml` 注册行内 passthrough `{ }` |
| `layouts/_markup/render-passthrough.html` | **核心**：`{文字|注音}` → `<ruby>`（按第一个 `\|` 切分，无 `\|` 则字面输出） |
| `content/songs/<slug>.md` | 每歌一页：YAML front matter + Markdown 歌词正文 |
| `layouts/_default/single.html` | 歌曲页模板（自写精简版，不再整份拷贝主题）：`data-lang`（按语言切字体）+ header（ruby 标题/artist/links）+ 歌词正文 |
| `layouts/index.json` | 搜索索引（覆盖主题版：去注音原文 + 注音文本） |
| `layouts/index.html` / `_default/list.html` | 首页与歌曲列表 |
| `assets/css/custom.css` | 布局/歌词/居中样式（Blowfish 自动加载） |
| `assets/css/font.css` | 字体规则（extend-head.html 显式挂载） |
| `layouts/partials/extend-head.html` | Noto Serif 字体 + 挂载 font.css |
| `.github/workflows/deploy.yml` | GitHub Pages：clone 主题 → hugo → deploy |
| `themes/` | 主题本地 clone，**不入库**（.gitignore） |

## 歌词书写

正文就是普通 Markdown，规则：

- **注音**：`{文字|注音}`——两侧都**任意字符、任意长度**。
  例：`{明日|あした}`、`{食|た}べる`、`{クリア|Clear}`、`{窓|창}外{밖|밖}에`。
- **花括号是保留语法**：`{无竖线内容}` 按字面显示；行内代码 `` `{x|y}` `` 不受影响；
  页面里其它位置的 `{…|…}` 也会被处理（勿用于代码/JSON 示例）。
- **分行**：单换行即一行（`white-space: pre-line` 生效），无需硬换行标记；
  **空行 = 段落（段间隔）**。罗马字行直接照写即可（如 dawn 的 `(chang bakk-e …)`）。
- **Markdown 语法可用**：`**加粗**`、`*斜体*`、链接、列表等；下划线用 HTML `<u>…</u>`（已开 unsafe）。
- 注意：Markdown 智能引号会把 `I'm` 转成 `I’m`（如需关闭可再议）。
- front matter `language`（ja/ko/zh/en）决定页面字体族与元数据。

## 加一首歌

1. 新建 `content/songs/<slug>.md`：YAML front matter（title/romaji/artist/language/description）+ 正文歌词；
   `links` 用列表形式（显示名与 URL 都按原样）：
   ```yaml
   links:
     - name: 哔哩哔哩
       url: https://www.bilibili.com/video/xxx
     - name: Youtube
       url: https://www.youtube.com
   ```
2. `hugo server` 预览 → 提交推送，CI 自动部署。

## 页面功能

- 歌词 ruby 注音（任意字符）；页面按 `language` 用 Noto Serif JP/KR/SC（全站衬线）。
- 搜索（Ctrl+K）：搜题名/歌手/原文/注音。
- 深浅模式、响应式：Blowfish 自带。

## 版本同步（本地 = CI）

- **Hugo**：本地与 CI 都用 **0.165.0 extended**（workflow 里 `peaceiris/actions-hugo` 固定 `hugo-version: "0.165.0"`）。
- **Blowfish**：本地与 CI 都钉在提交 **`7ab6a75`**（`.github/workflows/deploy.yml` 的 checkout 行与本 README 的安装命令必须一致）。
- **行尾统一 LF**：本仓库 `.gitattributes` 已强制 `eol=lf`；主题克隆用 `-c core.autocrlf=false`。
  若 CSS 改动后本地与线上 bundle hash 不一致，先查行尾（CRLF vs LF）——这是曾导致漂移的根因。
- **更新主题（两步一起做，否则漂移）**：
  1. 本地：`Remove-Item -Recurse -Force themes\blowfish`，再按「快速开始」克隆到新提交；
  2. 把新提交的**完整 sha** 同步替换到 `.github/workflows/deploy.yml` 与本 README。
- 主题升级后检查 `layouts/_default/single.html`（自写精简版）是否仍兼容；`themes/` 不入库。
