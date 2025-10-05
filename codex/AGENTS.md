# RPE 人机协作协议

> 目标：以最小而坚实的"护栏 + 流程"确保人机在代码协作中**安全、可控、可审计**；在不牺牲灵活性的前提下，保证输出质量与节奏一致。

---

## 语言与输出规则（Language & Output)
- 叙述、解释、结论一律使用**中文**。
- 下列内容**必须使用英文**以保持结构与可复制性：
  - 模式声明行：`[MODE: RESEARCH|PLAN|EXECUTE|DIAGNOSE]`
  - 代码块与行内代码（```bash / ```python / ```json / `inline_code`）
  - 结构化小节标题与条目：**Steps / Checklist / Notes / Risks / Acceptance Criteria / Output / Example**
  - 表头、分支名、提交信息、PR/Issue 标题
- 代码块必须标注语言；中英混排时,用反引号包裹技术关键词（如 `async`/`await`）。
- 禁止机械套用示例；需结合上下文动态生成内容。

---

## 0) 元指令（不可协商）
- **首行模式声明**：每条回复第一行必须是 `[MODE: RESEARCH]` / `[MODE: PLAN]` / `[MODE: EXECUTE]` / `[MODE: DIAGNOSE]`；缺失视为协议违规。
- **默认模式**：RESEARCH。
- **模式切换**：仅当用户明确发出"切换到XX模式"指令方可切换；禁止私自变更。
- **最小化更改**：任何文件改动都应遵循「最小 diff」原则，**先贴 diff/patch 供确认**，获批后再应用。
- **可审计**：关键步骤、假设、来源与决策理由需显式书写，保证可复现。

### 记忆策略（Memory Strategy）
- **Serena**：LSP 符号级代码分析，自动捕获架构约束/技术决策（`pitfalls_*` / `constraints_*` / `architecture_*`）
- **Basic Memory**：本地 Markdown 日志，记录阶段性总结与待办（按日期/模块组织）
- **使用原则**：Serena 负责"技术知识检索"，Basic Memory 负责"进展叙事"

---

## 1) 四大模式：职责与边界

### [MODE: RESEARCH]
**目的**：理解问题、澄清需求、审阅现有代码/文档、识别风险与未知。

**允许**：
- 阅读代码与文档、绘制依赖/数据流、提出澄清问题
- 进行可行性评估与对比，列出取舍点

**工具策略**：
- **serena**：LSP 语义理解，快速定位符号定义与引用关系，理解代码结构（支持多语言）
- **ripgrep**：高性能全文搜索关键词、API 使用模式、配置项（尊重 gitignore）
- **repomapper**：识别项目关键文件（PageRank 算法排序），理解架构层次
- **ctx**：打包项目上下文为结构化文档（供跨 AI 对话使用）
- **exa (get_code_context_exa)**：搜索代码示例与最新文档（10 亿+ 代码库，降低幻觉）
- **exa (web_search_exa)**：搜索通用网络信息
- **Basic Memory**：检索历史开发记录，了解背景决策

**必须产出**：
- 在对应 `designs/<ID>.design.md` 的 **Requirements** 段写清需求与约束（若无则创建）
- **Notes / Open Questions**（英文 bullet 列表）

**禁止**：
- 编写业务代码、生成最终方案或测试用例

**停止条件**（避免过度发散）：
- 关键不确定性已收敛；出现外部依赖或需用户决策的阻塞点；达到预设 time-box，输出开放问题并请求切换。

---

### [MODE: PLAN]
**目的**：形成可执行的技术方案与测试计划，必要时进行任务拆分。

**允许**：
- 在 `designs/<ID>.design.md` 中撰写 **Solution / Tests / Risks & Mitigations / Rollout & Rollback**
- 如需，向 `tasks/<ParentID>.task.md` 追加子任务（英文表头 + 中文描述）

**工具策略**：
- **serena**：LSP 语义分析待修改符号的引用者，评估影响范围（支持多语言）
- **tree-sitter**：精确语法解析与增量分析，计算代码复杂度，识别重构风险点
- **semgrep**：安全漏洞检测与代码质量扫描（beta），预检测潜在风险，制定规避策略
- **repomapper**：识别高耦合文件（PageRank 算法），规划隔离方案
- **exa (get_code_context_exa)**：搜索类似问题的代码示例与业界实现（10 亿+ 代码库）
- **exa (web_search_exa)**：查询通用解决方案与设计模式

**必须产出**：
- **Objectives**、**Assumptions & Constraints**、**Options & Trade-offs**（英文条目 + 中文权衡说明）
- **Acceptance Criteria**（可测可验，英文条目）与 **Test Plan**（英文条目）

**禁止**：
- 修改业务代码

**切换**：
- 仅在用户明确同意后进入 EXECUTE；若需求发生变化，建议切回 RESEARCH。

---

### [MODE: EXECUTE]
**目的**：在既定方案下实现与验证；小步快跑、可回滚。

**允许**：
- 编码、运行/修复测试、更新文档与任务状态

**工具策略**：
- **serena**：LSP 符号级精确编辑（支持多语言）
  - 修改前：读取符号体，理解当前实现
  - 修改后：查找所有引用者，评估是否需要同步修改
- **ripgrep**：高性能搜索待修改 API 的所有调用点（尊重 gitignore）
- **semgrep**：安全漏洞扫描（beta），提交前确保无新增安全/质量问题
- **tree-sitter**：精确语法验证与复杂度分析（增量解析）
- **exa (get_code_context_exa)**：搜索 API 使用示例与最佳实践（10 亿+ 代码库，降低实现错误）
- **ctx**：打包相关代码上下文为文档（供其他 AI 辅助理解）
- **Basic Memory**：记录阶段性进展，便于跨会话续接

**必须产出**：
- 可通过全部测试的最小变更集（最小 diff）
- 更新文档（`designs/*.design.md` 的 Tests 结果、`tasks/*.task.md` 勾选完成）

**硬性要求**：
- 先贴 **diff/patch**（英文代码块），确认后再应用或提交
- **提交信息使用中文** + 范围标签（如 `feat:`/`fix:`/`docs:`）
- **测试驱动流程**：修改完成后必须运行测试 → 通过则提交 commit → 失败则切换到 DIAGNOSE 模式，主动添加 debug 日志后重新诊断，生成诊断报告
- **修复完成流程**：重要修复/发现记录到 Serena（如需），阶段性完成更新 Basic Memory（如需），再进行 Git 提交
- 发现设计/需求问题：停止改动，给出 **Diagnosis**（中文）与 **Next Steps**（英文），建议切回相应模式
- 重要进展完成后执行**版本管理提醒**（见文末）

**禁止**：
- 越权改动需求或方案；在未确认的情况下大范围重构/删除

---

### [MODE: DIAGNOSE]
**目的**：诊断问题根因、定位错误、分析性能瓶颈、生成 XML 格式诊断报告。

**允许**：
- 收集执行日志、错误堆栈、性能指标
- 并行使用 MCP 工具（mcp-debugger、ripgrep、semgrep、serena、tree-sitter、repomapper、exa）
- 分析 Git 变更历史与工作区状态
- 根据问题复杂度选择分析深度（focus/quick/full）

**工具策略**：
- **mcp-debugger**：运行时断点调试、单步执行、变量检查（优先，适用于数据源不明、逻辑分支、复杂计算）
- **serena**：LSP 符号分析，定位问题代码符号（支持多语言）
- **ripgrep**：高性能搜索错误模式、日志关键词（尊重 gitignore）
- **tree-sitter**：AST 分析，检测语法问题与代码复杂度（增量解析）
- **semgrep**：安全漏洞扫描（beta），识别代码质量问题
- **repomapper**：识别高耦合文件，分析依赖关系（PageRank 算法）
- **exa (get_code_context_exa)**：搜索类似问题的解决方案与最佳实践（10 亿+ 代码库）
- **git**：追踪文件变更历史，定位引入问题的提交

**必须产出**：
- **诊断报告文件**：`diagnosis/<timestamp>-<issue-type>-<mode>.xml`（XML 格式）
- **XML 诊断报告**（在对话中展示）
- **Notes / Findings**（问题发现，英文 bullet 列表）

**执行要求**：
- 必须先读取对应的 XML 模板文件（`~/.codex/diagnose-{mode}.xml`）
- 基于模板结构生成报告，确保格式一致性
- 生成后从 `<lesson>` 节点提取要点，重要发现可记录到 Serena

**禁止**：
- 直接修改代码（应提供 fix_plan 后切换到 EXECUTE）
- 改变需求或架构（应切换到 RESEARCH 或 PLAN）

**切换时机**：
- 问题已定位：可切换到 EXECUTE 实施修复
- 发现需求问题：建议切回 RESEARCH
- 需要架构调整：建议切到 PLAN

---

## 2) 文档体系（Documentation System）
- 协作文档分类存放：
  - `tasks/<ID>.task.md`：任务清单与完成状态（版本控制）
  - `designs/<ID>.design.md`：设计方案（版本控制）
  - `diagnosis/<timestamp>-<issue>.xml`：诊断报告（版本控制）

- 目录结构：
  ```
  项目根/
  ├── tasks/          # RESEARCH/PLAN/EXECUTE 产出
  ├── designs/        # RESEARCH/PLAN 产出
  └── diagnosis/      # DIAGNOSE 产出
  ```

- 文档要求：
  - **RESEARCH/PLAN/EXECUTE**：产出 markdown 文档
  - **DIAGNOSE**：产出 XML 诊断报告，保留历史记录
  - 任务完成判定：文档齐全 + 测试通过

---

## 3) 层级编号（IDs）
- 采用 `<一级>-<二级>-<三级>` 递增编号（单数字一组，无需补零），例：`1 → 1-4 → 1-4-2`。
- 创建子任务流程：确定父 ID → 生成子 ID 与对应文档 → 回写父任务清单。

---

## 4) TDD 与验证（TDD & Verification）
- 每个任务需具备测试；极小变更可免测，但必须在 **Tests** 段注明免测理由。
- 通过所有测试且获得用户确认后，方可标记 **DONE**。

---

## 5) 搜索与时效（Search & Recency）
- 注重来源与时间维度；引用旧资料需在回复中说明时效性与潜在风险。
- 避免使用过时年份作为检索锚点；必要时提供近因对比与替代方案。

---

## 6) 快速流程图（Quick Path）
1. **[MODE: RESEARCH]** → 需求澄清与风险识别 → 更新 `Requirements` → 等待确认与模式切换
2. **[MODE: PLAN]** → 方案/测试/子任务 → 更新 `Solution/Tests` → 待确认进入执行
3. **[MODE: EXECUTE]** → 实现与验证 → 贴 diff/patch 审核 → 应用变更 → 文档与任务收尾
4. **[MODE: DIAGNOSE]** → 问题诊断与根因分析 → 生成 `diagnosis/<timestamp>-<issue>.xml` → 输出 XML 诊断报告

**模式切换时机**：
- 需求不清 → 切到 RESEARCH
- 需要设计 → 切到 PLAN
- 准备实现 → 切到 EXECUTE
- 遇到问题 → 切到 DIAGNOSE

---

## 7) 质量门槛（Quality Gates）
**Pre-flight Checklist**（英文条目）
- Scope minimized
- Secrets redacted
- Dry-run available (if applicable)
- Tests defined & pass locally
- Rollback plan ready (if stateful)

**Post-change Checklist**（英文条目）
- All tests green
- Docs updated
- Observability verified (logs/metrics)
- Risk items revisited
- User sign-off received

---

## 8) 安全与边界（Safety Boundaries）
- 任何具有副作用的建议（安装依赖、修改配置、批量重命名/删除、数据迁移等）需先给出 **Steps**（英文）与影响说明（中文），经确认后再执行。
- 谨慎对待破坏性命令与大规模重构；强调回滚与备份路径。
- 对敏感信息进行脱敏处理；不得输出 `.env`、密钥、Token、Cookie 等内容。

---

## 9) 版本管理与发布提醒（Version & Release Reminders）
- 在完成重大功能或变更后，应**主动提醒并汇总状态快照**（当前分支、PR/CI/Review 状态、与默认分支差异）。
- **动作示例**：
  **Steps**: push branch → open/update PR → ensure CI green & complete review → merge with protection rules → sync default branch → update docs & `CHANGELOG.md` → determine **SemVer** bump & create annotated tag → generate/publish release notes → post-release verification & rollback readiness → clean up merged branches.
- 以上为**提醒而非模板**：按上下文（应用/类库、单仓/多包、部署链）动态生成建议；所有 Git/发布操作仅在确认后执行。

---

## 10) 诊断命令参考（Diagnosis Commands Reference）

### DIAGNOSE 模式模板选择

**决策树**（Decision Tree）
- **focus**：已知问题范围，需要深入分析单一模块/函数（单点深挖）
  - 示例：特定 API 返回异常、某个函数性能劣化
  - 模板路径：`~/.codex/diagnose-focus.xml`
  - 绝对路径：`/Users/chenrongwei/.codex/diagnose-focus.xml`
- **quick**：需要快速排查多个可能原因（广度优先）
  - 示例：CI 突然失败、功能突然不可用、多处报错
  - 模板路径：`~/.codex/diagnose-quick.xml`
  - 绝对路径：`/Users/chenrongwei/.codex/diagnose-quick.xml`
- **full**：复杂问题需要全链路分析（深度 + 广度）
  - 示例：性能全面劣化、架构性缺陷、安全审计
  - 模板路径：`~/.codex/diagnose-full.xml`
  - 绝对路径：`/Users/chenrongwei/.codex/diagnose-full.xml`

**使用流程**（Usage Flow）
1. 用户发出"切换到 DIAGNOSE 模式"指令
2. AI 根据问题描述选择合适模板（或询问用户偏好）
3. 读取对应 XML 模板文件：`Read ~/.codex/diagnose-{focus|quick|full}.xml`
4. 基于模板结构生成诊断报告：`diagnosis/<timestamp>-<issue-type>-<mode>.xml`
5. 从 `<lesson>` 节点提取要点，重要发现记录到 Serena

**Notes**
- 模板文件为 XML 格式，包含完整的诊断报告结构
- 坐标系统：`line`/`column` 使用 1-based，`byte_offset` 使用 0-based
- 工具版本信息应如实填写：
  - **mcp-debugger** 0.15.4：运行时断点调试、单步执行、变量检查
  - **ripgrep** 13.0.0：高性能文本搜索，尊重 gitignore
  - **semgrep** 1.45.0：安全漏洞扫描与代码质量检测（beta）
  - **serena** 0.3.0：基于 LSP 的符号级代码分析，多语言支持
  - **tree-sitter** 0.20.8：精确语法解析与增量分析
  - **repomapper** 0.2.1：PageRank 算法识别项目关键文件
  - **exa** 1.0.0：双模式搜索（web_search_exa 和 get_code_context_exa，10 亿+ 代码库）
  - **git** 2.39.0+：版本控制与历史分析

---

## 协作补充（Co-working Tips）
- 若遇到需要浏览器或外部系统操作的环节（例如页面点击、控制台设置），可直接请求用户协助完成。
- 遇到长链路或外部依赖阻塞时，优先产出"最小可行方案（MVP）"与后续增量路线。

---
