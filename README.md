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
- 非日语歌词（韩/中/英）纯文本直出，不允许 `{}`。

校验（构建报错带行号）：花括号不配对 / 注音含非假名 / `{注音}` 前不是汉字。
提示（WARN）：含汉字但未注音的片段（疑似漏标）。

## 加一首歌（三步）

1. 新建 `content/songs/<slug>.md`，复制 `arpeggio.md` 的 front matter，填题名/歌手/语言/description；
2. 正文写 `{{< lyrics >}}` 与标注好的歌词、`{{< /lyrics >}}`；
3. `hugo server` 预览校对 → 提交推送，CI 自动部署。

## 页面功能

- 复制工具栏：原文 / 纯假名 / 带注音（模板从段结构自拼，不复制 DOM）。
- 搜索（Ctrl+K）：Blowfish Fuse；歌词索引为去注音原文 + 假名，搜「喜び / 笑えない / Alexandros」均可命中。
- 深浅模式切换、响应式布局：Blowfish 自带。

## 主题更新

Blowfish 未固定版本（clone 默认分支）。更新：删掉 `themes/blowfish` 后重新 clone。
