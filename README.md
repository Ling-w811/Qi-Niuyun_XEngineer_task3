# ⚡ AI PR Review 助手

> 基于大语言模型的智能代码评审工具，帮助开发者提升 Pull Request Review 效率与质量。

![版本](https://img.shields.io/badge/版本-v2.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![语言](https://img.shields.io/badge/语言-HTML%20%2F%20JavaScript-orange)

---

## 📖 项目简介

AI PR Review 助手是一个纯前端工具，用户只需粘贴 GitHub PR 链接，系统自动获取代码变更并调用 AI 进行多维度智能分析，辅助发现潜在问题，生成 Review 建议，大幅降低代码评审的时间成本。

### 解决的核心问题

- 🕐 **效率问题**：大型 PR 文件多、改动多，人工逐行阅读耗时数小时
- 🔍 **漏报问题**：人工 Review 容易遗漏安全漏洞、性能隐患
- 🧩 **理解问题**：不熟悉某模块时，难以快速理解改动意图
- 📝 **一致性问题**：不同 Reviewer 标准不一，建议质量参差不齐

---

## ✨ 功能特性

| 功能 | 说明 |
|------|------|
| 📊 **PR 变更摘要** | AI 语义理解改动意图、影响范围、关键变更点 |
| ⚠️ **风险代码识别** | 自动识别安全漏洞、性能问题、逻辑缺陷，分高/中/低三级 |
| 💡 **Review 建议生成** | 给出具体可操作的改进建议，附改进前后代码对比 |
| 📝 **差异视图** | 真实渲染 GitHub diff，高亮增删行，标注风险位置 |
| 📊 **综合评分** | 代码质量、安全、性能、测试覆盖、文档五维评分 |
| 🤖 **多模型支持** | 支持 DeepSeek / OpenAI / Moonshot / 智谱 / 通义 / 自定义 |

---

## 🚀 快速开始

### 方式一：直接用浏览器打开（推荐）

1. 下载 `ai_pr_review_v2.html`
2. 用 VS Code 安装 **Live Server** 扩展
3. 右键文件 → **Open with Live Server**
4. 浏览器自动打开 `http://127.0.0.1:5500/...`

> ⚠️ 必须通过 Live Server 或本地服务器打开，直接双击（`file://`）会因浏览器 CORS 限制导致 API 请求失败。

### 方式二：任意 HTTP 服务器

```bash
# Python
python -m http.server 8080

# Node.js
npx serve .
```

---

## 🔧 使用说明

### 第一步：配置 AI 提供商

在左侧面板选择 AI 提供商并填入对应 API Key：

| 提供商 | 模型 | 获取地址 | 备注 |
|--------|------|----------|------|
| **DeepSeek** | deepseek-chat | [platform.deepseek.com](https://platform.deepseek.com) | 推荐，性价比高，有免费额度 |
| **智谱 GLM** | glm-4-flash | [open.bigmodel.cn](https://open.bigmodel.cn) | 完全免费 |
| OpenAI | gpt-4o-mini | [platform.openai.com](https://platform.openai.com) | 需海外账号 |
| Moonshot | moonshot-v1-8k | [platform.moonshot.cn](https://platform.moonshot.cn) | 国内可用 |
| 通义千问 | qwen-turbo | [dashscope.console.aliyun.com](https://dashscope.console.aliyun.com) | 国内可用 |
| 自定义 | 任意 | 填入兼容 OpenAI 格式的端点 | 支持本地 Ollama |

### 第二步：填写 PR 信息

- **GitHub PR 链接**：格式为 `https://github.com/owner/repo/pull/123`
- **GitHub Token**（可选）：公开仓库匿名访问每小时限 60 次，建议填入 Token 提升至 5000 次/小时

**申请 GitHub Token：**
```
https://github.com/settings/tokens/new
```
权限只需勾选 `public_repo` 即可。

### 第三步：选择分析模式

| 模式 | 适用场景 |
|------|----------|
| 🔍 全量分析 | 常规 Review，分析最全面（推荐） |
| 🔒 安全优先 | 涉及认证、权限、数据处理等敏感改动 |
| ⚡ 性能优先 | 涉及数据库、渲染、算法等性能敏感改动 |
| 💨 快速摘要 | 只需快速了解 PR 大意，时间紧迫时使用 |

---

## 🏗️ 系统设计

### 模型选择策略

本工具采用兼容 OpenAI 格式的统一接口调用各模型，核心设计考量：

- **temperature=0.2**：降低随机性，保证分析结果稳定可重现
- **强制 JSON 输出**：system prompt 约束模型只输出结构化 JSON，确保前端解析稳定
- **max_tokens=4000**：在响应速度和分析深度之间取得平衡
- **多模型兼容**：统一使用 OpenAI Chat Completions 格式，方便扩展新提供商

### 上下文获取方式

```
GitHub PR 链接
    │
    ├─► GitHub API v3 → PR 元数据（标题、描述、分支、标签、统计）
    │
    └─► GitHub API v3（diff格式）→ Unified Diff（文件变更内容）
            │
            └─► 智能截断（前 12000 字符）→ 构建结构化 Prompt → AI 分析
```

**分层上下文设计：**
1. PR 描述和元数据：理解改动意图
2. 文件变更列表：把握影响范围
3. 具体 diff 内容：进行细粒度分析

### 误报与漏报控制

- **结合意图分析**：将 PR 描述注入 prompt，帮助 AI 判断变更是否有意为之，减少误报
- **三级风险分层**：高/中/低风险分级，开发者按优先级处理，避免噪音
- **角色专注**：system prompt 明确 AI 的代码审查专家角色，减少无关输出
- **格式约束**：强制 JSON schema 输出，字段缺失时有默认值兜底

---

## 📁 项目结构

```
ai-pr-review/
├── ai_pr_review_v2.html    # 主程序（单文件，开箱即用）
└── README.md               # 项目说明文档
```

---

## 🔮 未来扩展方向

- [ ] **GitHub Actions 集成**：作为 CI 步骤自动触发，Review 结果直接评论到 PR
- [ ] **GitLab / Gitee 支持**：扩展到更多代码托管平台
- [ ] **代码库 RAG**：索引整个代码库，理解跨文件调用链，提升分析准确性
- [ ] **团队风格学习**：从历史 Review 评论中学习团队偏好，个性化建议
- [ ] **测试用例生成**：基于变更内容自动补全缺失的单元测试
- [ ] **多模型协作**：安全扫描专用模型 + 通用分析模型分工协作

---

## 🔒 安全与隐私

- API Key 仅保存在浏览器内存中，页面关闭即清除，不持久化
- GitHub Token 不会上传至任何第三方服务器
- 代码内容通过 HTTPS 传输至所选 AI 提供商的官方接口
- 企业场景可选择本地部署的模型（自定义模式 + Ollama）确保代码不出内网

---

## 📄 License

MIT License © 2025
