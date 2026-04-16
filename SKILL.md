---
name: wechat-publisher
description: |
  微信公众号一键全流程：选题 → 框架 → 写作 → 封面图 → 文章配图 → 排版发布草稿箱。
  自包含 skill，核心知识全在本目录内，不依赖读取其他 skill 的指令。
  图片生成走浏览器即梦，发布走 Playwright MCP 浏览器自动化（直接操控微信后台编辑器）。
  触发词：写公众号、一篇推文、公众号全流程、一键发公众号、从选题到发布、写一篇发一篇、
  全自动写公众号、帮我搞一篇公众号、今天发什么公众号、公众号内容生产。
  也覆盖：给公众号写篇文章、公众号创作流程、帮我写篇公众号文章。
  不应被"写博客"、"写周报"、"邮件"、"PPT"、"抖音/短视频"、"网站 SEO"触发——
  需要有公众号/微信/推文等明确上下文。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - AskUserQuestion
  - TaskCreate
  - TaskUpdate
  - TaskList
---

# WeChat Publisher — 公众号一键全流程

自包含公众号内容生产管线。选题、框架、素材、写作、SEO 的知识在本目录 `references/` 下。图片生成走浏览器即梦，发布走 Playwright MCP 浏览器自动化。

**路径约定**：`{dir}` = 本 SKILL.md 所在目录。

## 参考文档索引

| 文件 | 内容 | 加载时机 |
|------|------|---------|
| `references/topic-selection.md` | 选题评分规则（热度/相关度/切入价值三维打分） | Phase 1 |
| `references/frameworks.md` | 7 套写作框架（痛点/故事/清单/对比/热点解读/纯观点/复盘） | Phase 2 |
| `references/content-enhance.md` | 4 种增强策略（角度发现/密度强化/细节锚定/真实体感） | Phase 2 |
| `references/writing-guide.md` | 写作规范（反检测+结构+自检） | Phase 3 |
| `references/seo-rules.md` | 微信 SEO 规则（标题/摘要/关键词/完读率） | Phase 3 |
| `references/visual.md` | 封面+配图（5 维度自动选择+6 种配图类型+视觉锚定） | Phase 4-5 |
| `references/publishing.md` | 发布配置（API/Browser/预览三种方式+Metadata 预检） | Phase 6 |

## 运行时

```bash
which bun 2>/dev/null && BUN_X="bun" || BUN_X="npx -y bun"
```

缺失 → `npm install -g bun`

## 模式

- **自动模式**（默认）：一口气跑完 Phase 0-6，出错才停
- **交互模式**：用户说"交互模式"/"我要自己选"时，在选题、框架、封面、配图处暂停等用户确认

## 管线进度

启动时创建 7 个 Task：

```
Phase 0: 预检
Phase 1: 选题
Phase 2: 框架 + 素材
Phase 3: 写作 + SEO
Phase 4: 封面图
Phase 5: 文章配图
Phase 6: 发布
```

每开始一个 Phase → `TaskUpdate status=in_progress`。完成 → `TaskUpdate status=completed`。

---

## Phase 0: 预检

逐项检查，缺失的引导修复。

### 0.1 写作配置

```bash
test -f ~/.claude/skills/wewrite/style.yaml && echo "OK" || echo "MISSING"
python3 -c "import yaml" 2>&1
```

- `style.yaml` 缺失 → 读取 `{dir}/references/onboard.md` 引导设置
- Python yaml 缺失 → `pip install pyyaml`

### 0.2 发布环境

Playwright MCP 浏览器工具可用即可（Phase 6 使用 `browser_navigate`、`browser_click`、`browser_evaluate`、`browser_press_key` 等）。无需额外配置。

### 0.3 图片生成

**主方案：API 调用**（`toolkit/image_gen.py`）。支持 9 个 provider（doubao/openai/gemini/dashscope/minimax/replicate/azure_openai/openrouter/jimeng），多 provider 自动降级。

需要在 `config.yaml` 中配置至少一个 provider 的 API key：

```yaml
image:
  providers:
    - provider: gemini
      api_key: YOUR_GEMINI_API_KEY
    - provider: openai
      api_key: YOUR_OPENAI_API_KEY
```

**降级：浏览器即梦**（jimeng.jianying.com）。API 不可用或全部失败时，通过 Playwright MCP 打开即梦手动生成。无需 API key。

---

## Phase 1: 选题

**用户已给选题** → 记录关键词，跳到 Phase 2。

### 1.1 热点抓取

```bash
python3 {dir}/scripts/fetch_hotspots.py --limit 30
```

**降级**：脚本报错 → `WebSearch "今日热点 {style.yaml 的第一个 topic}"`

### 1.2 SEO + 历史分析

```bash
python3 {dir}/scripts/seo_keywords.py --json {关键词}
```

读取 `~/.claude/skills/wewrite/history.yaml`（不存在则跳过），近 7 天已写关键词降分去重。

**降级**：SEO 脚本报错 → LLM 自行判断。

### 1.3 生成选题

读取 `{dir}/references/topic-selection.md`，生成 **10 个选题**（7-8 热点 + 2-3 常青），每个含标题、评分、点击率潜力、推荐框架。

- 自动模式 → 选最高分
- 交互模式 → 展示全部，等用户选

**输出**：`{selected_topic}`

---

## Phase 2: 框架 + 素材

### 2.1 框架选择

读取 `{dir}/references/frameworks.md`，7 套框架自动选推荐指数最高的。

### 2.2 素材采集 + 内容增强

读取 `{dir}/references/content-enhance.md`，根据框架类型执行搜索策略，同时提取真实素材（5-8 条，具名来源）和增强材料（角度/密度/细节/体感）。

搜索策略：

| 框架 | 搜索关键词 |
|------|-----------|
| 热点解读/纯观点 | `"{关键词} 观点 OR 评论"` + `site:mp.weixin.qq.com` |
| 痛点/清单 | `"{关键词} 教程 OR 工具 OR 实操"` |
| 故事/复盘 | `"{人物/事件} 采访 OR 细节 OR 故事"` |
| 对比 | `"{A} vs {B} 评测 OR 踩坑 site:v2ex.com OR site:zhihu.com"` |

**降级**：WebSearch 不可用 → LLM 可验证的公开信息，告知用户多在编辑锚点加个人内容。

**输出**：`{framework_type}` + `{materials}` + `{enhance_strategy}`

---

## Phase 3: 写作 + SEO

### 3.1 加载写作配置

```
读取: {dir}/references/writing-guide.md
读取: {dir}/playbook.md（如存在）
读取: ~/.claude/skills/wewrite/history.yaml（最近 3 篇 dimensions + closing_type）
读取: {dir}/references/exemplars/index.yaml（如存在）
读取: {dir}/personas/{style.yaml 的 writing_persona}.yaml（默认 midnight-friend）
```

优先级：playbook（confidence ≥ 5）> persona > 范文风格 > writing-guide.md。

### 3.2 维度随机化

从 5 个维度随机激活 2-3 个，贯穿全文：

| 维度 | 选项 |
|------|------|
| 叙事视角 | 亲历者 / 旁观分析者 / 对话体 / 自问自答 |
| 时间线 | 顺叙 / 倒叙 / 插叙 |
| 类比域 | 体育 / 烹饪 / 军事 / 游戏 / 电影 / 建筑 |
| 情感基调 | 冷静克制 / 热血兴奋 / 毒舌调侃 / 温暖治愈 |
| 节奏型 | 急促短句 / 舒缓长叙述 / 快慢交替 |

如 history 有最近 3 篇的 dimensions 字段，避免重复组合。

### 3.3 范文注入

有 exemplar index.yaml 时，按框架类型匹配 top 3 范文，注入开头/情绪/转折/收尾风格。无范文库时读取 `{dir}/references/exemplar-seeds.yaml` 做 fallback。

### 3.4 写初稿

- H1 标题（20-28 字）+ H2 结构，1500-2500 字
- 素材 + 增强约束分散嵌入各 H2
- 按人格参数写作
- 2-3 个编辑锚点：`<!-- ✏️ 编辑建议：在这里加一句你自己的经历/看法 -->`
- 保存到 `{dir}/output/{date}-{slug}.md`

### 3.5 快速自检（当场修复）

5 项扫描命中则直接改：
1. **禁用词**：writing-guide.md 2.1 的列表，命中即替换
2. **句长方差**：连续 3 句以上长度接近则拆句或加短句
3. **开头钩子**：前 3 句是否制造悬念/冲突？平铺直叙则重写开头
4. **增强贯穿**：增强策略核心输出是否只出现在一段？是则补充到其他 H2
5. **金句**：全文是否有 ≥1 句可独立截图转发的句子？没有则在情绪高点补一句

### 3.6 SEO + 质量验证

读取 `{dir}/references/seo-rules.md`：
- 3 个备选标题 + 摘要（≤40 字）+ 5 标签 + 关键词密度
- 写作质量 7 项检查（句长方差/词汇温度/段落节奏/情绪极性/禁用词/真实锚定/具体性）
- 内容质量检查（按框架类型，见 content-enhance.md）
- 脚本辅助：`python3 {dir}/scripts/humanness_score.py {article_path} --json`
- composite_score < 30 通过；30-50 修复最低分 1-2 项；> 50 修复 2-3 项，最多 2 轮

**输出**：`{article_path}`（终稿 markdown）

---

## Phase 4: 封面图

读取 `{dir}/references/visual.md` 的"封面图"部分。

### 4.1 实体提取 + 维度选择

1. 从终稿提取 3-5 个具体实体（人物/产品/场景/数据）
2. 根据 visual.md 的信号表自动选择 Type × Palette × Rendering

### 4.2 生成 3 组创意提示词

按直觉冲击/氛围渲染/信息图表三种策略生成差异化提示词。每条必须包含 ≥2 个文章实体，禁止泛化词。

### 4.3 执行生成

选最佳 1 组，优先通过 API 生成：

```bash
python3 {dir}/toolkit/image_gen.py --prompt "{英文提示词}" --output {dir}/output/cover-{date}.png --size cover
```

**降级**：API 失败时，通过浏览器即梦生成：

1. `browser_navigate` → `https://jimeng.jianying.com`
2. 在即梦输入框输入提示词，选择 2.35:1 比例
3. 等待生成完成，下载图片到 `{dir}/output/cover-{date}.png`

**再降级**：即梦也不可用时，将提示词保存到 `{dir}/output/cover-prompts.txt`。

失败重试 1 次。

### 4.4 提取视觉锚点

封面确认后提取色板 hex + 风格关键词 + 画面调性。后续 Phase 5 的所有配图必须引用这组锚点。

**输出**：`{cover_path}` + `{visual_anchor}`

---

## Phase 5: 文章配图

读取 `{dir}/references/visual.md` 的"文章配图"部分。

### 5.1 分析结构

列出所有 H2，判断每段是否需要配图（有数据→infographic，有场景→scene，>400字→节奏调节等）。

### 5.2 确定配图

- 总数 3-6 张（按字数比例）
- 相邻间隔 ≥ 300 字
- 每张选最匹配的类型（infographic/scene/flowchart/comparison/framework/timeline）

### 5.3 生成 + 插入

按 visual.md 的结构化模板写提示词，每条末尾附加 Phase 4 的视觉锚点。优先通过 API 批量生成：

```bash
python3 {dir}/toolkit/image_gen.py --prompt "{英文提示词}" --output {dir}/output/illust-{N}.png --size article
```

**降级**：API 失败时，通过浏览器即梦生成：

1. 在即梦页面逐个输入提示词（16:9 比例）
2. 下载图片到 `{dir}/output/illust-{N}.png`

**再降级**：即梦也不可用时，将所有提示词保存到 `{dir}/output/illustration-prompts.txt`。

在 markdown 中对应段落后插入 `![](output/illust-{N}.png)`

**输出**：`{article_path}`（已插入配图的 markdown）

---

## Phase 6: 发布

读取 `{dir}/references/publishing.md`。

### 6.1 Metadata 预检

| 检查项 | 标准 | 不通过时 |
|--------|------|---------|
| H1 标题 | 5-64 字节 | 自动修正 |
| 摘要 | ≤ 120 UTF-8 字节 | 从首段生成 |
| 封面图 | 存在 | 警告但仍可推送 |
| 正文字数 | ≥ 200 字 | 警告 |
| 图片 | ≤ 10 张 | 移除末尾多余 |

### 6.2 读取配置

```bash
# 排版主题：从 style.yaml 读取，默认 professional-clean
theme=$(python3 -c "import yaml; d=yaml.safe_load(open('$HOME/.claude/skills/wewrite/style.yaml')); print(d.get('theme','professional-clean'))")

# 署名：从 style.yaml 读取
author=$(python3 -c "import yaml; d=yaml.safe_load(open('$HOME/.claude/skills/wewrite/style.yaml')); print(d.get('author',''))")
```

### 6.3 浏览器自动化发布（Playwright MCP）

使用 Playwright MCP 直接操控微信后台编辑器，无需 API 凭证。流程：

**Step 1: 打开微信后台并确认登录**

```
browser_navigate → https://mp.weixin.qq.com
```

如页面显示登录二维码 → 提示用户扫码，等待登录完成。

**Step 2: 进入文章编辑器**

```
browser_click → "新的创作" 下的 "文章"
```

**Step 3: 填写标题和作者**

```
browser_click → 标题输入框
browser_type → "{标题}"
browser_click → 作者输入框
browser_type → "{作者}"
```

**Step 4: 生成内联样式 HTML**

将 markdown 终稿转为带内联样式的 HTML。关键样式规则：
- 段落：`font-size: 16px; line-height: 1.8; color: #333; margin: 12px 0`
- H2：`font-size: 22px; font-weight: 700; color: #1a1a1a; padding: 8px 0 8px 12px; border-left: 4px solid #2563eb; border-bottom: 1px solid #e5e7eb`
- H3：`font-size: 18px; font-weight: 600; color: #333; margin: 24px 0 12px 0`
- 加粗：`font-weight: 700; color: #1a1a1a`
- 行内代码：`font-family: monospace; font-size: 14px; background: #f1f5f9; color: #d946ef; padding: 2px 6px; border-radius: 4px`
- 代码块：`background: #1e293b; color: #e2e8f0; padding: 16px; border-radius: 8px; white-space: pre-wrap`

**Step 5: 通过剪贴板粘贴正文**

微信后台编辑器使用 ProseMirror，直接设置 innerHTML 不可靠。必须通过剪贴板粘贴：

```javascript
// browser_evaluate 执行：
const htmlContent = `<内联样式HTML>`;
const clipboardItem = new ClipboardItem({
  'text/html': new Blob([htmlContent], { type: 'text/html' })
});
await navigator.clipboard.write([clipboardItem]);

const editor = document.querySelector('.ProseMirror');
editor.focus();
document.execCommand('selectAll');
```

然后：

```
browser_press_key → Meta+v  (Mac) 或 Control+v  (其他)
```

**Step 6: 填写摘要**

```
browser_click → 摘要输入框
browser_type → "{摘要}"
```

**Step 7: 保存草稿**

```
browser_click → "保存为草稿" 按钮
```

等待页面显示"已保存"确认。

**降级**：如 Playwright MCP 不可用 → 将 HTML 保存到 `{dir}/output/{date}-{slug}.html`，提示用户手动打开微信后台粘贴。

### 6.4 写入历史

```yaml
# → ~/.claude/skills/wewrite/history.yaml
- date: "{日期}"
  title: "{标题}"
  topic_source: "热点抓取"  # 或 "用户指定"
  topic_keywords: ["{词1}", "{词2}"]
  output_file: "{article_path}"
  framework: "{框架}"
  enhance_strategy: "{增强策略}"
  word_count: {字数}
  cover_path: "{cover_path}"
  illustrations: {配图数}
  publish_method: "{api/browser/preview}"
  media_id: "{id}"
  dimensions:
    - "{维度}: {选项}"
```

---

## 完成报告

```
✅ 公众号文章全流程完成！

📝 文章
  标题：{title}
  备选：{alt1} | {alt2}
  摘要：{digest}
  标签：{tags}
  字数：{count}
  框架：{framework}

🎨 视觉
  封面：{cover_path}
  配图：{N} 张

📦 发布
  方式：{api/browser/preview}
  media_id：{id}
  管理：https://mp.weixin.qq.com

📋 后续操作
  - 草稿箱：https://mp.weixin.qq.com（内容管理 → 草稿箱）
  - 文章有编辑锚点，建议加入你自己的话
  - 说"润色"/"缩写"/"扩写"可修改文章
  - 说"换封面"可重新生成封面
  - 说"学习我的修改"可更新写作风格
```

---

## 降级策略

| 阶段 | 降级方案 |
|------|---------|
| 热点抓取 | WebSearch 替代 |
| SEO 脚本 | LLM 自行判断关键词 |
| 素材采集 | LLM 公开信息 + 提示用户多加个人内容 |
| 封面/配图生成 | API (image_gen.py) → 浏览器即梦 → 保存提示词 |
| 浏览器发布 | Playwright MCP 不可用 → 保存 HTML 文件，提示手动粘贴 |
| 全部图片失败 | 输出提示词 + 备选图库关键词 |
| 脚本报错 | 按错误信息引导修复 |

---

## 后续操作

| 用户说 | 动作 |
|--------|------|
| 润色/缩写/扩写/换语气 | 编辑文章 |
| 换封面 | 重新 Phase 4 |
| 换配图 | 重新 Phase 5 |
| 换选题 | 回到 Phase 1 |
| 用框架 B 重写 | 回到 Phase 2 |
| 学习我的修改 | 读取 `{dir}/references/learn-edits.md` |
| 检查文章质量 | `python3 {dir}/scripts/humanness_score.py {path} --json` |
| 看看文章数据 | 读取 `{dir}/references/effect-review.md` |
