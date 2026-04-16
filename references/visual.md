# 视觉生成模块

整合封面图和文章配图的完整视觉管线。使用已安装的图片生成脚本执行。

---

## 一、封面图

### 5 维度自动选择

当用户未指定时，根据文章内容信号自动匹配：

**Type**（画面类型）

| 信号 | 类型 |
|------|------|
| 产品、发布、公告 | `hero` |
| 架构、框架、技术模型 | `conceptual` |
| 观点、金句、标题 | `typography` |
| 哲学、成长、抽象概念 | `metaphor` |
| 故事、经历、叙事 | `scene` |
| 极简、聚焦、核心 | `minimal` |

**Palette**（色调）

| 信号 | 色调 |
|------|------|
| 个人故事、情感、生活 | `warm` |
| 商业、专业、职场 | `elegant` |
| 技术、代码、系统 | `cool` |
| 娱乐、电影、暗调 | `dark` |
| 自然、健康、旅行 | `earth` |
| 发布、游戏、活动 | `vivid` |
| 柔和、创意、幻想 | `pastel` |
| 极简、专注、纯粹 | `mono` |

**Rendering**（渲染风格）

| 信号 | 风格 |
|------|------|
| 现代、科技、信息图 | `flat-vector` |
| 手绘、笔记、个人感 | `hand-drawn` |
| 艺术、水彩、梦幻 | `painterly` |
| 数据、仪表盘、专业 | `digital` |

默认：`title-only` 文字级别，`balanced` 情绪，`clean` 字体，`2.35:1` 比例（公众号封面）。

### 封面生成 3 组创意

每组走不同视觉策略：

**创意 A: 直觉冲击** — 视觉隐喻直接表达核心观点，对比强烈
**创意 B: 氛围渲染** — 营造情绪场景引发好奇
**创意 C: 信息图表** — 简洁图形传递主题信息

### 实体锚定（必须）

生成前从文章提取 3-5 个具体实体（人物/产品/场景/数据）。每条提示词必须包含至少 2 个实体。禁止用"科技感""未来感"等泛化词替代具体内容。

**反例** → **正例**：
- ❌ "蓝色科技背景，数据流动" → ✅ "AI 生成的短剧角色走出手机屏幕，背景是废弃的真人拍摄片场"

### 提示词格式

```
视觉描述：{100-150字画面描述}
色调：{主色+辅色 hex}
构图：16:9 横版，主体位置，留白位置
文字区域：{标题位置和空间}
AI 提示词："{英文，含风格+构图+色调+光影，16:9 aspect ratio, no text no letters}"
```

### 封面确认后提取视觉锚点

```
视觉锚点：
- 色板：{主色 hex + 辅色 hex}
- 风格关键词：{如 "flat illustration, minimalist, bold outlines"}
- 画面调性：{冷调/暖调/中性}
```

后续所有配图必须复用这组锚点。

---

## 二、文章配图

### 分析流程

1. **提取结构**：列出所有 H2 标题及下属段落
2. **判断是否需要配图**：

| 需要 | 不需要 |
|------|--------|
| 有数据/统计 → 信息图强化 | 纯观点短段（<200字） |
| 有场景描写 → 画面还原 | 已有引用/代码块 |
| 转折/高潮处 → 视觉冲击 | 紧接另一张配图（<300字） |
| 长段落（>400字无图） → 节奏调节 | 结尾 CTA 段落 |

3. **选择图片类型**

| 类型 | 适用内容 |
|------|---------|
| `infographic` | 数据、统计、指标对比 |
| `scene` | 叙事场景、情绪渲染 |
| `flowchart` | 流程、步骤、工作流 |
| `comparison` | 方案/观点对比 |
| `framework` | 概念模型、架构关系 |
| `timeline` | 时间线、发展历程 |

4. **约束**
   - 总数 3-6 张（1500字→3张，2000字→4张，2500字→5-6张）
   - 相邻配图间隔 ≥ 300 字
   - 不在文章第一段之前放图
   - 不在结尾 CTA 段落放图

### 结构化提示词模板

每种类型使用对应的模板格式（非自由文本）：

**infographic**：Layout + Zones(数据点) + Labels(真实术语) + Colors(锚点色板) + Style
**scene**：Focal Point + Atmosphere + Mood + Color Temperature(与锚点一致) + Style
**flowchart**：Layout + Steps + Connections + Colors + Style
**comparison**：Left Side + Right Side + Divider + Colors(左右各一主色) + Style
**framework**：Structure + Nodes + Relationships + Colors + Style
**timeline**：Direction + Events + Markers + Colors + Style

每条提示词末尾附加视觉锚点的色板和风格关键词。配图位置插入在对应段落**之后**。

---

## 三、执行

提示词写好后，按优先级生成图片：

**主方案：API 调用**（快速、可批量、无需浏览器）

```bash
# 封面（2.35:1 比例）
python3 {dir}/toolkit/image_gen.py --prompt "{英文提示词}" --output {dir}/output/cover.png --size cover

# 配图（16:9 比例）
python3 {dir}/toolkit/image_gen.py --prompt "{英文提示词}" --output {dir}/output/illust-{N}.png --size article
```

支持 9 个 provider，多 provider 自动降级。需在 `config.yaml` 配置 API key。

**降级：浏览器即梦**（免费、无需 API key）

1. `browser_navigate` → `https://jimeng.jianying.com`
2. 在即梦输入框输入提示词，选择合适比例（封面 2.35:1，配图 16:9）
3. 等待生成完成，下载图片到 `{dir}/output/` 目录

**再降级**：输出提示词到文件，继续流程。
