# 微信发布模块

将终稿 markdown 发布到微信公众号草稿箱。
使用 Playwright MCP 浏览器自动化直接操控微信后台编辑器。

---

## 发布方式

仅一种方式：**Playwright MCP 浏览器自动化**。

通过 Playwright MCP 的 `browser_navigate`、`browser_click`、`browser_evaluate`、`browser_press_key` 等工具，直接操控 Chrome 浏览器操作微信后台。

---

## 前置条件

- Playwright MCP 可用（浏览器工具已连接）
- 用户已登录微信后台（或愿意扫码登录）

### 排版偏好

从 `~/.claude/skills/wewrite/style.yaml` 读取：
- `theme`: 排版主题（用于选择内联样式配色）
- `author`: 署名

## 发布前预检

| 检查项 | 标准 | 不通过时 |
|--------|------|---------|
| H1 标题 | 存在且 5-64 字节 | 自动修正 |
| 摘要 | 存在且 ≤ 120 UTF-8 字节 | 从首段自动生成 |
| 封面图 | 存在 | 警告但仍可推送 |
| 正文字数 | ≥ 200 字 | 警告"内容过短" |
| 图片数量 | ≤ 10 张 | 移除末尾多余图片 |

## 发布流程

### Step 1: 打开微信后台

```
browser_navigate → https://mp.weixin.qq.com
```

- 如果页面显示登录二维码 → `AskUserQuestion` 提示用户扫码
- `browser_wait_for` 等待登录完成（页面出现"公众号"标题）

### Step 2: 创建新文章

```
browser_click → "新的创作" 区域的 "文章" 按钮
```

等待编辑器页面加载完成（URL 包含 `appmsg_edit`）。

### Step 3: 填写标题和作者

```
browser_click → 标题输入框
browser_type → "{标题}"（使用 textbox placeholder "请在这里输入标题"）
browser_click → 作者输入框
browser_type → "{作者}"（使用 textbox placeholder "请输入作者"）
```

### Step 4: 生成内联样式 HTML

将 markdown 终稿转为带内联样式的 HTML。这是核心步骤——微信编辑器（ProseMirror）需要通过剪贴板接收 HTML 格式内容。

**内联样式规范**：

| 元素 | 样式 |
|------|------|
| H2 | `font-size: 22px; font-weight: 700; color: #1a1a1a; margin: 28px 0 14px 0; padding: 8px 0 8px 12px; border-left: 4px solid #2563eb; border-bottom: 1px solid #e5e7eb; line-height: 1.4` |
| H3 | `font-size: 18px; font-weight: 600; color: #333; margin: 24px 0 12px 0; line-height: 1.4` |
| 段落 | `font-size: 16px; line-height: 1.8; color: #333; margin: 12px 0` |
| 加粗 | `font-weight: 700; color: #1a1a1a` |
| 行内代码 | `font-family: "SFMono-Regular", Consolas, monospace; font-size: 14px; background: #f1f5f9; color: #d946ef; padding: 2px 6px; border-radius: 4px` |
| 代码块 | `background: #1e293b; color: #e2e8f0; padding: 16px; border-radius: 8px; overflow-x: auto; margin: 16px 0; line-height: 1.6; white-space: pre-wrap; word-wrap: break-word` |

### Step 5: 通过剪贴板粘贴正文

**重要**：微信后台编辑器使用 ProseMirror，直接设置 innerHTML 不可靠。必须通过剪贴板粘贴。

```javascript
// browser_evaluate 执行以下代码：
const htmlContent = `<步骤4生成的HTML>`;

// 写入剪贴板（HTML 格式）
const clipboardItem = new ClipboardItem({
  'text/html': new Blob([htmlContent], { type: 'text/html' })
});
await navigator.clipboard.write([clipboardItem]);

// 聚焦 ProseMirror 编辑器
const editor = document.querySelector('.ProseMirror');
editor.focus();

// 选中全部现有内容（覆盖占位文字）
document.execCommand('selectAll');
```

然后执行键盘粘贴：

```
browser_press_key → Meta+v (Mac) 或 Control+v
```

### Step 6: 验证内容

```javascript
// browser_evaluate 检查：
const editor = document.querySelector('.ProseMirror');
return {
  hasContent: editor.children.length > 1,
  text: editor.textContent.substring(0, 100)
};
```

如果 `hasContent` 为 false → 重新执行 Step 5。

### Step 7: 填写摘要

```
browser_click → 摘要输入框
browser_type → "{摘要}"
```

### Step 8: 保存草稿

```
browser_click → "保存为草稿" 按钮
browser_wait_for → "已保存" 文本出现
```

### Step 9: 记录结果

从页面确认保存成功（状态栏显示"XX:XX已保存"，正文字数 > 0）。

## 降级

| 情况 | 处理 |
|------|------|
| Playwright MCP 不可用 | 保存 HTML 到本地文件，提示用户手动打开微信后台粘贴 |
| 微信后台未登录且用户拒绝扫码 | 同上 |
| 剪贴板粘贴失败 | 重试 1 次；仍失败则尝试直接 `editor.innerHTML = htmlContent` |
| 编辑器找不到 ProseMirror | 查找 `[contenteditable]` 元素作为 fallback |

## 完成报告

```
WeChat Publishing Complete!

Method: browser-playwright
Article: {title} by {author}

Result:
✓ Draft saved to WeChat
  状态：{已保存时间}
  正文字数：{count}

Next: https://mp.weixin.qq.com (内容管理 → 草稿箱)
```
