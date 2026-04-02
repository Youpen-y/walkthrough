# Walkthrough

[**English**](./README.md) | **中文**

一个用于生成交互式 HTML 演示文档的技能，通过可点击的 Mermaid 图表（流程图和 ER 图）来解释代码库的功能、流程、架构和数据库模式。

灵感来自 [Amp's Shareable Walkthroughs](https://ampcode.com/news/walkthrough)。

**[在线演示 — walkthrough 技能自身工作原理演示](https://youpen-y.github.io/walkthrough/examples/walkthrough-how-it-works.html)**

## 使用方法
通过以下提示词触发此技能：

```
walkthrough how does the drawing tool work
walkthrough the user authentication flow
explain this database schema
walk me through the API request lifecycle
```

AI Agent 将会：

1. 理解你要解释的范围
2. 启动并行子代理探索代码库
3. 选择合适的图表类型（流程图或 ER 图）
4. 生成一个独立的 HTML 文件，包含可点击的交互式图表

## 安装

### 快速安装

```bash
npx skills add https://github.com/Youpen-y/walkthrough --skill walkthrough
```

### 手动安装

将 `skills/walkthrough/` 目录复制到项目的 `.claude/skills/` 文件夹中：

```
your-project/
  .claude/
    skills/
      walkthrough/
        skill.md
        references/
          html-patterns.md
```

## 结构

```
skills/walkthrough/
  skill.md                      # 主要技能定义
  references/
    html-patterns.md            # HTML 模板、CSS 和 JS 模式参考
```

- **skill.md** — Agent 遵循的技能提示。定义了工作流程：范围理解、并行代码库探索、图表类型选择和 HTML 生成。
- **references/html-patterns.md** — 生成 HTML 文件的完整参考：React 组件架构、Mermaid 配置、Shiki 设置、调色板、平移/缩放实现，以及生成可工作的 walkthrough 所需的所有模式。

## 技术栈（生成的文件）

输出的 HTML 文件是完全独立的，使用 CDN 依赖：
- **Mermaid 11** (ESM) — 图表渲染（流程图和 ER 图）
- **Shiki** (ESM) — 语法高亮，使用 `vitesse-dark` 主题

无需构建步骤。直接在浏览器中打开 HTML 文件即可。

## 关键特性

### 节点类型与颜色编码

| 类型 | 颜色 | 用途 |
|------|------|------|
| `component` | 紫色 | 主要组件、页面、处理器 |
| `utility` | 蓝色 | 工具函数、辅助函数、纯函数 |
| `external` | 灰色 | 库、浏览器 API、外部服务 |
| `event` | 青色 | 事件、用户操作、触发器、入口点 |
| `data` | 绿色 | 存储、状态、数据结构、配置 |

### 可选代码片段

每个节点*可以*包含一个可选的代码片段（最多 5 行）。大多数节点不应有代码 —— 只有在真正必要时才包含最具说明性的那一段代码。

### 节点尺寸

节点通过 Mermaid 的 `style` 语法使用显式的宽度/高度样式，以确保文本正确显示。

### TL;DR 摘要

渲染为可折叠的手风琴（默认折叠），因此用户首先看到的是图表。

### 深色模式

每个 walkthrough 使用纯黑背景（`#000000`）、白色文字和紫色强调色。绝不生成浅色模式的 walkthrough。

## 测试

`evals/` 目录包含一个评估工具，可以对一组测试提示运行技能并给输出打分。

### 前提条件

- 已安装并认证 `claude` CLI
- Node.js >= 18

### 运行评估

```bash
# 运行所有 16 个测试提示
bash evals/run.sh

# 仅运行 4 个关键提示（更快的反馈循环）
bash evals/run.sh --subset

# 通过 ID 运行单个提示
bash evals/run.sh --id explicit-01

# 跳过 LLM 评分器（仅确定性检查）
bash evals/run.sh --skip-llm

# 使用特定模型（默认：sonnet）
bash evals/run.sh --model opus
```

你也可以通过环境变量设置默认值：

```bash
EVAL_MODEL=opus EVAL_MAX_BUDGET=3.00 bash evals/run.sh
```

### 工作原理

每次评估运行：

1. 将项目复制到临时目录并安装技能
2. 使用 `evals/prompts.csv` 中的每个提示运行 `claude -p`
3. 收集任何生成的 `walkthrough-*.html` 文件
4. 运行两个评分器：
   - **确定性** (`graders/deterministic.mjs`) — 检查文件存在性、HTML 结构、CDN 依赖、节点数量、图表类型
   - **LLM 评分** (`graders/llm-rubric.mjs`) — 使用 Claude 对可读性、描述、代码片段和图表准确性进行评分，依据是 `graders/rubric.md`
5. 在 `evals/results/<timestamp>/summary.json` 生成摘要报告

结果保存在 `evals/results/`（gitignored）。`latest` 符号链接始终指向最近的运行。

### 测试提示

`evals/prompts.csv` 中的提示覆盖：
- **显式触发** — `$walkthrough how does X work`
- **隐式触发** — `walk me through X`, `explain this flow`
- **图表类型** — 流程图和 ER 图情况
- **负面情况** — 不应触发技能的提示
- **边缘情况** — 模糊提示、广泛范围
