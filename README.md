# lyrics — 多语言歌词展示站（Hugo + Blowfish）

歌词写在歌曲页正文里，由 `lyrics` shortcode 在 **Hugo 模板内**解析 `汉字{注音}` 紧凑标记并渲染成 `<ruby>` 注音；
页面外观使用 **Blowfish** 主题（Tailwind，自带搜索/深浅模式/多语言）。零外部脚本。

## 快速开始

```powershell
# 1. 安装主题（已 gitignore，本地/CI 都要先 clone）
git clone --depth 1 https://github.com/nunocoracao/blowfish.git themes/blowfish

# 2. 预览 / 构建
hugo server          # http://localhost:1313
hugo --gc --minify   # 生产构建 → public/
```

搜索：右上角放大镜或 **Ctrl+K**（Blowfish 内置 Fuse 搜索）。

## 目录结构

| 路径 | 说明 |
|---|---|
| `config/_default/*.toml` | Blowfish 配置（hugo / params / languages.zh-cn / menus） |
| `content/songs/<slug>.md` | 每歌一页：front matter 元数据 + 正文 `{{< lyrics >}}` 歌词块 |
| `layouts/shortcodes/lyrics.html` | 核心：模板内解析 `{注音}` → ruby + 复制数据（格式校验 errorf/warnf） |
| `layouts/index.json` | 全站搜索索引（覆盖主题版：歌词 content = 去注音原文 + 假名，跨注音词可命中） |
| `layouts/_default/single.html` | 主题 single 的站点拷贝：仅给 `<article>` 加 `data-lang`（按页面语言切字体；主题升级后需同步此文件，删掉即回退主题默认） |
| `layouts/partials/extend-head.html` | 加载 Google Noto 字体（拉丁 / 日 / 韩 / 简中） |
| `layouts/partials/extend-footer.html` | Blowfish 钩子：注入「复制歌词」按钮脚本 |
| `layouts/index.html` / `_default/list.html` | 首页与歌曲列表（自制，配 custom.css） |
| `assets/css/custom.css` | 歌词 ruby / 工具栏 / 列表样式（主题自动合并进 bundle） |
| `.github/workflows/deploy.yml` | GitHub Pages：clone 主题 → hugo → deploy |
| `themes/` | 主题本地 clone，**不入库**（.gitignore） |

## 歌词标注语法

在歌曲页正文的 `{{< lyrics >}}…{{< /lyrics >}}` 内书写：

- 一行 = 歌词显示的一行；空行 = 段落间隔。
- **汉字串紧跟 `{注音}`**，注音是该汉字串的整体读音（模板不做读音判断）。
- 只标汉字，送假名不标：`食{た}べる`；整词连标：`苦虫{にがむし}`、`明日{あした}`
- 数字+汉字组合读音跨越两者时改写为汉字：`1人` → `一人{ひとり}`
- 语言由页面 front matter `language` 决定（缺省 ja），决定读音字符集校验：
  - `ja`：汉字注音为假名；**片假名上方可标英文原词**，如 `クリア{Clear}`（不进入复制文本与搜索，样式 `rt.rt-gloss` 单独调）；
  - `ko`：韩语同样支持「汉字{谚文读音}」，如 `夜{밤}`、`窗{창}外{밖에}`；
  - 中文 / 英文等其他语言：纯文本直出，不允许 `{}`。
- **罗马字注音行**：整行被 `(…)` 包住的行（如 dawn.md 的 `(changbakke namgyeojin gieogi)`）
  是罗马字注音而非歌词正文——不进复制文本与搜索索引；
  工具栏「罗马字：开/关」按钮可整块开关（localStorage 记忆，默认开）；
  样式在 `assets/css/custom.css` 的 `.romaji-line` 单独调。
- **下划线**：歌词块内是原始文本（不解析 Markdown），需要下划线时用 `__文字__`，
  渲染为 `<u>`（可跨注音标注）；三种复制文本会自动去掉 `__` 标记。

校验（构建报错带行号）：花括号不配对 / 注音含非假名 / `{注音}` 前不是汉字。
提示（WARN）：含汉字但未注音的片段（疑似漏标）。

## 加一首歌（三步）

1. 新建 `content/songs/<slug>.md`，复制 `arpeggio.md` 的 front matter，填题名/歌手/语言/description；
2. 正文写 `{{< lyrics >}}` 与标注好的歌词、`{{< /lyrics >}}`；
3. `hugo server` 预览校对 → 提交推送，CI 自动部署。

## 页面功能

- 复制工具栏：原文 / 读音（ja=假名、ko=谚文）/ 带注音（模板从段结构自拼，不复制 DOM）。
- 罗马字注音：`(…)` 整行识别为罗马字，工具栏按钮开关显示（默认开，记忆偏好），样式 `.romaji-line` 独立控制。
- 搜索（Ctrl+K）：Blowfish Fuse；歌词索引为去注音原文 + 读音，搜「喜び / 笑えない / Alexandros / 밤」均可命中。
- 深浅模式切换、响应式布局：Blowfish 自带。
- 字体：Google **Noto Serif**（衬线；Noto Serif JP/KR/SC + Noto Serif），全站衬线基准，
  页面 front matter `language` 决定 `article[data-lang]` → 字体族（ja→Noto Serif JP 优先，ko→KR，zh→SC，其余→Noto Serif）。
  浏览器只下载实际用到的 unicode 分片；不想要 webfont 时删除 `layouts/partials/extend-head.html` 即可回退系统字体。

## 主题更新

Blowfish 未固定版本（clone 默认分支）。更新：删掉 `themes/blowfish` 后重新 clone。
