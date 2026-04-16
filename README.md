# WeChat Publisher

> Claude Code Skill：微信公众号一键全流程内容生产管线

选题 → 框架 → 素材采集 → 写作 → 封面图 → 文章配图 → 发布草稿箱，一条龙。

## 它能做什么

给它一个选题（或者让它自己找热点），它会：

1. **选题评分** — 抓取热点 + SEO 分析，自动打分选出最优选题
2. **框架搭建** — 7 套写作框架（痛点/故事/清单/对比/热点解读/纯观点/复盘）自动匹配
3. **素材采集** — 搜索真实素材 + 4 种内容增强策略
4. **写初稿** — 5 个维度随机化（视角/时间线/类比域/情感/节奏），反 AI 检测，1500-2500 字
5. **质量自检** — 禁用词扫描、句长方差、开头钩子、金句检测、人类感评分
6. **封面图** — 5 维度自动选择（类型/色调/渲染/文字/情绪），通过即梦生成
7. **文章配图** — 6 种配图类型智能匹配，3-6 张配图自动插入
8. **发布草稿箱** — 通过 Playwright 浏览器自动化，直接粘贴到微信公众号后台

## 安装

### 前置要求

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Python 3.10+
- [Playwright MCP](https://github.com/anthropics/anthropic-quickstarts/tree/main/mcp-playwright)（用于浏览器自动化发布和即梦图片生成）

### 安装步骤

```bash
# 1. 克隆到 Claude Code 的 skills 目录
git clone https://github.com/Ryan-99/wechat-publisher.git ~/.claude/skills/wechat-publisher

# 2. 安装 Python 依赖
pip install -r ~/.claude/skills/wechat-publisher/requirements.txt
```

### 首次使用

启动 Claude Code 后说「写一篇公众号文章」，首次运行会触发 onboard 引导：

- 设置公众号名称、写作方向、风格偏好
- 选择写作人格（5 种内置人格）
- 选择排版主题（16 种内置主题）
- 可选：导入历史文章学习你的写作风格

## 项目结构

```
wechat-publisher/
├── SKILL.md                 # Skill 主文件（触发规则 + 管线定义）
├── playbook.md              # 写作风格 Playbook（从你的历史文章学习生成）
├── requirements.txt         # Python 依赖
│
├── references/              # 参考文档（按需加载）
│   ├── topic-selection.md   # 选题评分规则
│   ├── frameworks.md        # 7 套写作框架
│   ├── content-enhance.md   # 4 种内容增强策略
│   ├── writing-guide.md     # 写作规范（反检测 + 结构 + 自检）
│   ├── seo-rules.md         # 微信 SEO 规则
│   ├── visual.md            # 封面图 + 配图生成规则
│   ├── publishing.md        # 发布流程（Playwright 浏览器自动化）
│   ├── onboard.md           # 首次设置引导
│   ├── learn-edits.md       # 从用户编辑中学习
│   ├── effect-review.md     # 文章效果复盘
│   ├── exemplar-seeds.yaml  # 范文种子
│   └── exemplars/           # 范文库（可扩展）
│
├── personas/                # 写作人格定义
│   ├── midnight-friend.yaml # 像深夜朋友聊天（推荐个人号）
│   ├── warm-editor.yaml     # 温暖故事驱动
│   ├── industry-observer.yaml # 克制专业分析
│   ├── sharp-journalist.yaml  # 短句利落锐评
│   └── cold-analyst.yaml    # 严谨数据分析
│
├── scripts/                 # 工具脚本
│   ├── fetch_hotspots.py    # 热点抓取
│   ├── seo_keywords.py      # SEO 关键词分析
│   ├── humanness_score.py   # 人类感评分
│   ├── fetch_stats.py       # 微信文章数据拉取
│   ├── learn_theme.py       # 从公众号文章学习排版主题
│   ├── build_playbook.py    # 从语料生成写作 Playbook
│   └── ...                  # 其他辅助脚本
│
├── toolkit/                 # 核心工具箱
│   ├── cli.py               # CLI 入口（preview/publish/themes/gallery）
│   ├── converter.py         # Markdown → 内联样式 HTML 转换
│   ├── theme.py             # 主题系统
│   ├── publisher.py         # 微信草稿发布
│   ├── wechat_api.py        # 微信 API 封装
│   ├── image_gen.py         # AI 图片生成（多 provider）
│   └── themes/              # 16 种排版主题
│       ├── professional-clean.yaml
│       ├── midnight.yaml
│       ├── sspai.yaml
│       ├── github.yaml
│       └── ...              # 更多主题
│
└── output/                  # 生成文件输出（gitignore）
```

## 使用方式

### 基本用法

在 Claude Code 中直接说：

```
写一篇公众号文章
```

它会自动跑完整个管线。

### 指定选题

```
帮我写一篇关于 MCP 协议的公众号文章
```

跳过选题阶段，直接进入框架搭建。

### 交互模式

```
用交互模式写一篇公众号
```

在选题、框架、封面、配图等关键节点暂停，让你做选择。

### 后续操作

| 说... | 效果 |
|-------|------|
| 润色 / 缩写 / 扩写 | 编辑当前文章 |
| 换封面 | 重新生成封面图 |
| 换配图 | 重新生成文章配图 |
| 换选题 | 回到选题阶段重来 |
| 学习我的修改 | 从你的编辑中学习写作风格 |

## 写作人格

| 人格 | 风格 | 适合场景 |
|------|------|---------|
| `midnight-friend` | 深夜朋友聊天，极度口语化 | 个人号 / 自媒体 |
| `warm-editor` | 故事驱动，温暖共鸣 | 生活 / 文化 / 情感 |
| `industry-observer` | 克制的专业分析，偶尔锐利 | 行业媒体 / 深度分析 |
| `sharp-journalist` | 短句利落，观点鲜明 | 新闻 / 评论 |
| `cold-analyst` | 严谨数据，专业措辞 | 财经 / 投研 |

## 排版主题

16 种内置主题，覆盖从极简到花哨的各种风格：

`professional-clean` · `minimal` · `midnight` · `sspai` · `github` · `newspaper` · `bauhaus` · `bytedance` · `tech-modern` · `elegant-rose` · `minimal-gold` · `warm-editorial` · `ink` · `bold-green` · `bold-navy` · `focus-red`

预览全部主题：

```bash
cd ~/.claude/skills/wechat-publisher/toolkit
python3 cli.py gallery
```

## 技术架构

### 图片生成

通过 Playwright MCP 浏览器自动化操作 [即梦](https://jimeng.jianying.com)（字节跳动 AI 图片生成工具）生成图片，不需要 API key。

- 封面图：2.35:1 比例（公众号封面）
- 文章配图：16:9 比例
- 视觉锚点：封面确认后提取色板 + 风格关键词，后续配图保持视觉统一

### 发布

通过 Playwright MCP 直接操控 Chrome 浏览器操作微信公众号后台：

1. 打开 `mp.weixin.qq.com`（如需扫码会提示你）
2. 自动创建新文章、填写标题和作者
3. 将 Markdown 转为内联样式 HTML，通过剪贴板粘贴到 ProseMirror 编辑器
4. 填写摘要，保存到草稿箱

不需要 API 凭证，只需要扫码登录。

### 降级策略

| 阶段 | 主方案 | 降级方案 |
|------|--------|---------|
| 热点抓取 | 脚本抓取 | WebSearch 替代 |
| 素材采集 | WebSearch 搜索 | LLM 公开信息 |
| 封面/配图 | 即梦浏览器生成 | 保存提示词手动生成 |
| 发布 | Playwright 浏览器自动化 | 保存 HTML 文件手动粘贴 |

## 配置文件

配置文件位于 `~/.claude/skills/wewrite/` 目录（首次 onboard 时自动创建）：

- `style.yaml` — 写作风格配置（公众号名、方向、人格、主题等）
- `history.yaml` — 发布历史（每篇文章的元数据和写作参数）

这些文件已被 `.gitignore` 排除，不会上传到 GitHub。

## 许可

MIT
