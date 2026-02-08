本文将完整介绍 OpenClaw 的浏览器控制能力，详细讲解 CDP（Chrome DevTools Protocol）协议的集成方式，以及页面元素快照、表单自动填写、页面截图与导航等核心功能，帮助开发者快速实现 Web 自动化任务。

是否希望你的 AI 助手能够自动填写表单、抓取网页数据，或生成页面截图？OpenClaw 的 Browser 能力正是为此而生。它基于 Chrome DevTools Protocol（CDP）提供完整的浏览器控制能力，让 AI Agent 不再只是“讨论”网页，而是真正**操作**网页。

**核心价值：**
 通过阅读本文，你将学会如何利用 OpenClaw 提供的 5 项核心浏览器能力，构建一套从页面访问、交互到表单自动化的完整工作流。

翻译如下，保持技术文档风格，清晰、精准：

---

### OpenClaw 浏览器核心亮点

| 关键点          | 描述                                  | 价值                   |
| ------------ | ----------------------------------- | -------------------- |
| **CDP 协议控制** | 通过 Chrome DevTools Protocol 直接控制浏览器 | 绕过 GUI 限制，以机器速度执行操作  |
| **智能元素引用**   | 快照系统自动识别并编号交互元素                     | 无需手动编写选择器，AI 可直接引用元素 |
| **隔离浏览器环境**  | 独立的 OpenClaw 浏览器配置档                 | 完全隔离个人浏览数据，安全可控      |
| **多种快照模式**   | AI 快照模式与角色快照模式                      | 可适应不同场景下的元素识别需求      |
| **全动作支持**    | 点击、输入、拖拽、截图、导出 PDF                  | 覆盖所有常见网页自动化操作        |

---

### OpenClaw 浏览器的工作原理

OpenClaw 的浏览器控制能力基于一个核心理念：**直接执行代码，而非视觉推断**。传统 AI 的网页操作通常依赖截图和 UI 元素识别，这类方法容易出错且执行速度慢。而 OpenClaw 通过 CDP 协议直接与浏览器引擎通信，可实现毫秒级响应。

系统架构分为三层：

1. **浏览器层**：独立的 Chromium 实例，与个人浏览器完全隔离。
2. **控制层**：Gateway HTTP API 提供统一的控制接口。
3. **代理层**：AI 模型通过 OpenClaw CLI 调用浏览器操作。

这种架构的优势在于**安全与可控性**：AI 无法访问你的个人浏览数据，所有自动化操作均在隔离环境中完成。

---

🎯 **实践建议**
OpenClaw Browser 需要调用大语言模型（LLM）来理解网页内容并做出操作决策。通过 [APIYI](https://apiyi.com) 平台，你可以使用统一接口访问 Claude、GPT 等模型，让你的浏览器自动化变得更加智能。

翻译如下，保持技术文档风格和术语准确：

---

### 功能 1：浏览器配置管理

OpenClaw 支持三种浏览器配置模式，以应对不同的使用场景：

| 配置模式         | 描述                          | 使用场景         |
| ------------ | --------------------------- | ------------ |
| **openclaw** | 独立的 Chromium 实例，拥有专用的用户数据目录 | 推荐默认模式；安全性最高 |
| **chrome**   | 通过扩展控制现有 Chrome 标签页         | 当你需要利用已登录状态时 |
| **remote**   | 连接远程 CDP 端点，例如 Browserless  | 云端部署或无头服务场景  |

#### 创建自定义配置档

```bash
openclaw browser create-profile --name myprofile --color "#FF6B35"
```

配置会保存在 `~/.openclaw/openclaw.json` 文件中，并支持如下选项：

```json
{
  "browser": {
    "headless": false,
    "noSandbox": false,
    "executablePath": "/path/to/chrome"
  },
  "profiles": {
    "myprofile": {
      "cdpUrl": "http://localhost:9222",
      "color": "#FF6B35"
    }
  }
}
```

---

### 功能 2：页面导航与标签管理

导航控制是浏览器自动化的核心功能。OpenClaw 提供了完整的标签页管理能力：

#### 打开网页

```bash
# 使用 OpenClaw 浏览器配置档打开 URL
openclaw browser --browser-profile openclaw open https://example.com
```

#### 标签管理

```bash
# 列出所有打开的标签页
openclaw browser tabs

# 聚焦到指定标签页
openclaw browser focus <tab-id>

# 关闭指定标签页
openclaw browser close <tab-id>
```

#### 智能等待机制

确定页面何时加载完成通常是自动化中最难的部分。OpenClaw 支持多种等待条件，简化这一过程：

```bash
openclaw browser wait "#main" \
  --url "**/dashboard" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

| 等待类型   | 参数       | 描述                                           |
| ------ | -------- | -------------------------------------------- |
| URL 匹配 | `--url`  | 等待 URL 匹配特定模式                                |
| 加载状态   | `--load` | 支持 `load`、`domcontentloaded` 和 `networkidle` |
| 选择器    | 默认参数     | 等待元素出现在 DOM 中                                |
| JS 条件  | `--fn`   | 自定义 JavaScript 表达式                           |

---

### 功能 3：元素快照与引用系统

这是 OpenClaw 浏览器最强大的功能之一。快照系统会自动扫描页面，为所有可交互元素分配引用编号。AI 可以直接使用这些编号与元素交互，无需手动编写 CSS 选择器。

#### 两种快照模式

| 模式            | 引用格式            | 特性                    | 依赖         |
| ------------- | --------------- | --------------------- | ---------- |
| AI Snapshot   | 数字 (12, 23)     | 默认格式，针对 AI 处理优化       | Playwright |
| Role Snapshot | 元素引用 (e12, e23) | 基于 Accessibility Tree | Playwright |

#### 获取快照

```bash
# AI Snapshot（数字引用）
openclaw browser snapshot

# Role Snapshot（带交互标记）
openclaw browser snapshot --interactive

# 带可视化标注的截图
openclaw browser snapshot --labels
```

#### 示例快照输出

```
[1] Search Box <input type="text" placeholder="Search...">
[2] Login Button <button>Login</button>
[3] Register Link <a href="/register">Free Sign Up</a>
[4] Nav Menu <nav>Products | Pricing | Docs</nav>
```

> **重要说明：**
> 一旦页面发生跳转，元素引用将失效。如果操作失败，需要重新获取快照并使用更新后的引用编号。

翻译如下，保持技术文档风格，清晰、准确：

---

### 功能 4：元素交互操作

得益于快照引用系统，OpenClaw 支持多种元素交互操作：

#### 点击操作

```bash
# 点击编号为 12 的元素
openclaw browser click 12

# 使用 Role 引用点击元素
openclaw browser click e12

# 高亮元素（调试时非常有用）
openclaw browser highlight e12
```

#### 输入文本

```bash
# 在输入框 23 中输入文本
openclaw browser type 23 "Hello OpenClaw"

# 先清空再输入
openclaw browser type 23 "New Content" --clear
```

#### 表单填写

```bash
# 批量填写多个字段
openclaw browser fill \
  --field "username:myuser" \
  --field "password:mypass" \
  --field "email:test@example.com"
```

#### 其他交互操作

| 操作 | 命令                    | 描述              |
| -- | --------------------- | --------------- |
| 拖拽 | `drag 12 23`          | 从元素 12 拖动到元素 23 |
| 选择 | `select 12 "option1"` | 从下拉菜单选择一个选项     |
| 滚动 | `scroll --y 500`      | 垂直滚动 500 像素     |
| 悬停 | `hover 12`            | 将鼠标悬停在某个元素上     |

💡 **实用提示**
表单自动化是 OpenClaw 浏览器的核心应用场景之一。结合大语言模型的推理能力，你可以智能识别表单结构并自动填写。通过 APIYI 获取 Claude API，还可以让你的表单自动化更加智能化。

翻译如下，保持技术文档风格和条理清晰：

---

### OpenClaw 浏览器快速上手

#### 最简示例

下面是浏览器自动化的最简单工作流程：

```bash
# 1. 启动浏览器
openclaw browser --browser-profile openclaw start

# 2. 打开网页
openclaw browser open https://example.com

# 3. 获取页面快照
openclaw browser snapshot

# 4. 点击元素（假设搜索框编号为 [1]）
openclaw browser click 1

# 5. 输入搜索内容
openclaw browser type 1 "OpenClaw tutorial"

# 6. 保存截图
openclaw browser screenshot --output result.png
```

---

#### 查看完整自动化脚本示例

##### Python 集成示例

如果你希望使用 Python 控制 OpenClaw 浏览器，可以参考如下方式：

```python
import subprocess
import json

def openclaw_browser(command: str) -> str:
    """执行 OpenClaw 浏览器命令并返回结果"""
    result = subprocess.run(
        f"openclaw browser {command}",
        shell=True,
        capture_output=True,
        text=True
    )
    return result.stdout

# 打开页面
openclaw_browser("open https://example.com")

# 获取快照
snapshot = openclaw_browser("snapshot --json")
elements = json.loads(snapshot)

# 点击第一个按钮
openclaw_browser("click 1")

# 截图
openclaw_browser("screenshot --output page.png")
```

💡 **实用提示**
通过 [APIYI](https://apiyi.com) 调用大语言模型 API，你可以将 Python 脚本与 AI 的推理能力结合，实现更智能的网页自动化操作。
