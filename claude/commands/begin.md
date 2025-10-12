# 会话启动检查（Session Start）

你已进入 **RPE 协作模式**，请确认以下核心规则：

## 元规则（不可协商）
- ✅ 每次输出必须首行声明 `[MODE: RESEARCH|PLAN|EXECUTE|DIAGNOSE]`
- ✅ 采用"提案式切换"：助手提出切换提案，用户以"OK/NO"单行确认；默认模式 **RESEARCH**
- ✅ **默认使用中文**，仅技术标准使用英文（模式声明、代码块、Git）
- ✅ **Git 提交信息使用中文** + 范围标签（如 `feat:`/`fix:`/`docs:`）
- ✅ 任何有副作用的操作需先展示**执行步骤**与影响说明，获批后再执行
- ✅ 文件修改遵循最小 diff 原则：先贴 patch，确认后再应用

---

## 9 个 MCP 工具已就绪

**调试工具**：
- `mcp-debugger` - 运行时断点调试、单步执行、变量检查（适用于复杂问题诊断）

**记忆层（三层记忆体系）**：
- **Claude Code Todos** - 当前会话任务追踪（内置，自动管理）
- **serena** - 基于 LSP 的项目级智能记忆，符号级代码分析与编辑（多语言支持：Python/JS/Java 等）
  - 自动捕获架构约束、模块陷阱、技术决策
  - 命名规范：`pitfalls_<module>` / `constraints_<topic>` / `architecture_<component>`
- **basic-memory** - 跨会话、跨项目的开发日志（`~/basic-memory/<project>/`）
  - 知识图谱导航（memory:// URLs）、本地 Markdown、Obsidian 集成
  - 目录结构：`decisions/`（重要决策）、`designs/`（设计方案）、`diagnosis/`（重要问题）
  - 核心能力：跨项目统一管理、全文搜索、持久化记录

**代码分析**：
- `ripgrep` - 高性能文本搜索与错误定位（尊重 gitignore）
- `tree-sitter` - 精确语法解析与增量分析（AST 分析与复杂度计算）
- `semgrep` - 安全漏洞扫描与代码质量检测（beta 状态）
- `repomapper` - 项目结构分析（PageRank 算法识别关键文件）

**知识检索**：
- `ctx` - 上下文打包与管理（生成结构化文档供跨 AI 对话使用）
- `exa` - 双模式搜索
  - `web_search_exa` - 通用网络搜索，获取实时信息
  - `get_code_context_exa` - 代码上下文检索（10 亿+ GitHub/文档/Stack Overflow，降低 AI 幻觉）

---

## 模式选择指南

**不清楚需求或需要理解代码？** → 保持 **RESEARCH** 模式
- 工具组合：`serena` + `ripgrep` + `repomapper`
- 记忆层：`Basic Memory`（检索历史）+ `Serena`（查询约束）
- 产物：研究笔记 → `~/basic-memory/<project>/designs/`（如需跨会话保存）

**需求明确，准备设计方案？** → 切换到 **PLAN** 模式
- 工具组合：`serena` + `tree-sitter` + `semgrep` + `repomapper`
- 记忆层：`Basic Memory`（创建决策记录）+ `Serena`（记录约束）
- 产物：设计方案 → `~/basic-memory/<project>/designs/` + 决策记录 → `~/basic-memory/<project>/decisions/`

**方案已定，开始编码？** → 切换到 **EXECUTE** 模式
- 工具组合：`serena` + `ripgrep` + `semgrep` + `tree-sitter`
- 记忆层：`Claude Code Todos`（追踪任务）+ `Basic Memory`（记录进展）
- 产物：代码变更 + 测试通过的 commit

**遇到错误或性能问题？** → 切换到 **DIAGNOSE** 模式
- 工具组合：全部 9 个 MCP（优先使用 `mcp-debugger` 进行运行时调试，按 focus/quick/full 选择其他工具）
- 记忆层：`Serena`（记录 pitfalls）+ `Basic Memory`（重要问题）
- 产物：XML 诊断报告 → 临时生成，重要问题可保存到 `~/basic-memory/<project>/diagnosis/`

---

准备好了吗？请告诉我你的任务，我将自动选择合适的模式与工具组合。

**常见任务快速入口**（详见 `../CLAUDE.md`）：
理解新模块 → RESEARCH | 重构函数 → PLAN | 修复安全漏洞 → DIAGNOSE + EXECUTE | 优化性能 → DIAGNOSE (Full) + EXECUTE | 接入新库 → RESEARCH + PLAN + EXECUTE | 调试复杂 Bug → DIAGNOSE (mcp-debugger 优先) | 代码迁移/升级 → RESEARCH + PLAN + EXECUTE
