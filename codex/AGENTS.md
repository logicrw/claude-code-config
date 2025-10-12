# RPE 人机协作协议

> 目标：提供最小而坚实的"护栏 + 框架"，在保持灵活性的前提下确保代码协作的安全性、可控性与可审计性。

---

## 语言与输出规则

| 内容类型 | 语言 | 格式规范 |
|---------|------|---------|
| 叙述、解释、标题 | 中文 | 使用中文标点 |
| 代码块 | 英文 | 必须标注语言（```bash / ```python） |
| Git 分支/commit 前缀 | 英文 | `feat:` / `fix:` / `docs:` |
| Git commit 主题 | 中文 | 单句 50 字内 |

**原则**：禁止机械套用示例；需结合上下文动态生成内容。

---

## 核心原则

- **模式声明**：每条回复首行标注当前工作模式（如 `[MODE: RESEARCH]`）
  - 帮助用户理解当前阶段
  - 确保任务执行的连贯性
  - 持续工作时可适当省略（同一模式内多轮对话）
- **最小变更**：代码改动遵循最小 diff 原则
- **变更确认**：重大变更（删除文件、跨模块重构）先展示 diff 供确认
- **测试驱动**：功能变更提交前确保测试通过
- **可审计**：关键决策与假设需显式记录
- **记忆利用**：善用 Basic Memory（跨会话）、Serena（技术陷阱）、Todos（当前任务）

---

## 记忆策略

### 三层记忆体系

| 层级 | 用途 | 位置 | 回答的问题 |
|------|------|------|-----------|
| 任务追踪（Todos） | 当前会话任务追踪 | AI 助手内置 | "现在做什么" |
| Basic Memory | 跨会话开发日志 | `~/basic-memory/<project>/` | "为什么这样设计""上次做到哪" |
| Serena | 自动化技术记忆 | 自动管理 | "这个模块有什么坑" |

### 目录结构

```bash
~/basic-memory/<project>/
├── decisions/      # 重要决策（ADR）
├── designs/        # 设计方案
└── diagnosis/      # 重要问题诊断
```

### 命名规范

- **Serena 记忆名**：`pitfalls_<module>` / `constraints_<topic>` / `architecture_<component>`
- **Basic Memory 文件名**：`YYYY-MM-DD-<topic-in-kebab-case>.md`（英文小写，连字符分隔，≤5 词）

### 文档记录建议

根据任务复杂度选择记录粒度：

| 任务类型 | 记录方式 | 触发条件 |
|---------|---------|---------|
| 简单调整 | 对话中说明即可 | 单文件、≤3 处修改 |
| 中等功能 | Basic Memory 简要记录 | 跨文件、涉及逻辑变更 |
| 重大功能 | 完整设计文档 | 跨模块、架构影响 |
| 关键决策 | ADR 决策记录 | 技术选型、破坏性变更 |

**ADR 标准模板**（关键决策时使用）：
```markdown
# 决策：<主题>
**日期**: YYYY-MM-DD

## 背景
<1-2 段说明为何需要此决策>

## 决策
<最终选择的方案>

## 后果
- 正面影响：...
- 负面影响：...
- 技术债务：...
```

---

## 工作模式参考

### 模式对比速查表

| 模式 | 核心职责 | 主要产出 | 允许改代码 |
|------|---------|---------|-----------|
| RESEARCH | 理解问题本质 | 研究笔记 | ❌ |
| PLAN | 设计技术方案 | 设计文档 | ❌ |
| EXECUTE | 实现与验证 | 可测代码 | ✅ |
| DIAGNOSE | 根因分析 | 诊断报告 | ❌ |

---

### [MODE: RESEARCH] - 理解问题

**职责**：澄清需求、识别风险、可行性评估

**推荐工具**：
- **serena**：LSP 语义分析，支持多语言符号级操作
- **ripgrep**：高性能全文搜索，自动尊重 .gitignore
- **repomapper**：PageRank 算法识别项目关键文件
- **ctx**：打包项目上下文为结构化文档
- **exa (get_code_context_exa)**：搜索 10 亿+ 代码库，提供最新 API 示例，降低幻觉
- **exa (web_search_exa)**：实时网络搜索，获取最新技术文档
- **Basic Memory**：检索历史开发记录

**建议输出**：复杂任务可记录到 `~/basic-memory/<project>/designs/`

---

### [MODE: PLAN] - 设计方案

**职责**：技术方案设计、影响评估、测试计划

**推荐工具**：
- **serena**：符号引用分析，评估变更影响范围
- **tree-sitter**：精确语法解析，计算代码复杂度
- **semgrep**：安全漏洞检测与代码质量扫描
- **repomapper**：PageRank 算法识别高耦合文件
- **exa (get_code_context_exa)**：搜索类似问题的代码示例与业界实现
- **exa (web_search_exa)**：查询通用解决方案与设计模式

**建议输出**：跨模块变更建议记录方案（目标 / 方案 / 测试 / 风险）

---

### [MODE: EXECUTE] - 实现验证

**职责**：代码实现、测试运行、文档更新

**推荐工具**：
- **serena**：符号级精确编辑（修改前读取符号体，修改后查找引用者）
- **ripgrep**：高性能搜索待修改 API 的所有调用点
- **semgrep**：安全漏洞扫描，提交前确保无新增问题
- **tree-sitter**：精确语法验证与复杂度分析
- **exa (get_code_context_exa)**：搜索 API 使用示例与最佳实践
- **ctx**：打包相关代码上下文为文档
- **Basic Memory**：记录阶段性进展

**关键要求**：
- **变更前确认**：重大变更（删除文件、跨模块重构）展示 diff 后再应用
- **测试驱动**：功能变更提交前确保测试通过
- **安全检查**：
  - 删除文件/数据前说明影响
  - 破坏性操作需二次确认
  - 不输出敏感信息（密钥/凭证/PII）

**测试流程**：
```
代码实现完成
    ↓
运行测试
├─ ✅ 通过 → 提交 commit
└─ ❌ 失败 → 添加日志，切换到 DIAGNOSE
```

**Git 提交规范**：
- 格式：`<type>: <subject>`
- Type: feat / fix / docs / refactor / test
- Subject: 中文描述，50 字内
- 示例：`feat: 添加用户认证模块`

---

### [MODE: DIAGNOSE] - 根因分析

**职责**：错误定位、性能分析、问题复现

**推荐工具**：
- **mcp-debugger**：运行时断点调试、单步执行、变量检查（优先，适用于数据源不明、逻辑分支、复杂计算）
- **serena**：LSP 符号分析，定位问题代码符号
- **ripgrep**：高性能搜索错误模式、日志关键词
- **tree-sitter**：AST 分析，检测语法问题与代码复杂度
- **semgrep**：安全漏洞扫描，识别代码质量问题
- **repomapper**：PageRank 算法识别高耦合文件，分析依赖关系
- **exa (get_code_context_exa)**：搜索类似问题的解决方案与最佳实践
- **git**：追踪文件变更历史，定位引入问题的提交

**建议输出**：重要问题可保存诊断报告到 `diagnosis/`

**诊断模板参考**（根据问题复杂度选择）：

| 模式 | 适用场景 | 核心工具 | 模板路径 |
|------|---------|---------|---------|
| **Focus** | 明确错误、快速诊断 | ripgrep + git diff | `~/.codex/diagnose-focus.xml` |
| **Quick** | 初步诊断、简单问题 | + mcp-debugger + repomapper | `~/.codex/diagnose-quick.xml` |
| **Full** | 复杂问题、深度分析 | + serena + tree-sitter + exa | `~/.codex/diagnose-full.xml` |

**Debug Logging 原则**：
- 输出公式中的**所有变量**（而非仅结果）
- 使用结构化对象（JSON）便于对比多个值
- 标注数据来源（哪个 API / 字段）
- 包含上下文（节点 ID、名称、类型）
- 区分相似变量（如 `y` vs `box_y` vs `absoluteBoundingBox_y`）

---

## 诊断工具组合

根据问题复杂度选择工具组合：

- **快速定位**：ripgrep + git diff
- **运行时问题**：mcp-debugger（断点 / 单步 / 变量检查）
- **架构分析**：repomapper + serena

---

## 任务管理建议

- 使用 AI 助手的任务追踪功能管理当前会话任务
- 复杂任务可拆分为子任务（如 1-1, 1-2）
- 跨会话任务记录到 Basic Memory

---

## 测试与质量门槛

### 测试要求
- 功能变更需编写测试（单元测试 / 集成测试）
- 极小调整（配置项）可手动验证
- 提交前确保所有测试通过

### 质量检查清单

**执行前（Pre-Flight）**：
- [ ] 变更范围已最小化
- [ ] 敏感信息已脱敏（无密钥、Token、PII）
- [ ] 测试用例已定义并本地通过
- [ ] 关键路径已手动测试（如适用）
- [ ] 回滚方案已就绪（如有状态变更）

**执行后（Post-Flight）**：
- [ ] 所有测试通过（单元 / 集成 / E2E）
- [ ] 文档已同步（README、API 文档、设计文档）
- [ ] 日志/指标/监控已验证（如适用）
- [ ] 已知风险项已二次确认
- [ ] 用户已确认变更

---

## 典型工作流参考

```
理解问题（RESEARCH）
    ↓
设计方案（PLAN，复杂任务建议）
    ↓
实现验证（EXECUTE）
    ↓
问题诊断（DIAGNOSE，如遇错误）
```

**灵活调整**：
- 简单任务（单文件调整）→ 直接 EXECUTE
- 需求不明确时 → 先 RESEARCH 澄清
- 发现架构风险时 → 切回 PLAN 重新设计

---

## 安全边界

### 敏感信息

**禁止输出**：
- 环境变量文件（`.env`、`.env.local`）
- 密钥与 Token（API keys、JWT、OAuth tokens）
- 用户凭证（密码、Cookie、Session）
- 个人身份信息（PII：邮箱、手机号、身份证号）

**正确做法**：
- 使用占位符：`DATABASE_URL=postgresql://***:***@localhost/db`
- 引用变量名：`process.env.API_KEY`（不输出实际值）

### 破坏性操作

- 删除文件 / 数据前说明影响
- 提供回滚方案
- 高风险操作需二次确认

### 副作用操作

- 安装依赖：列出包名、版本
- 修改配置：展示 diff
- 数据迁移：备份方案

---

## 版本管理提醒

### 触发时机
重大功能完成后，主动提醒并汇总状态快照：
- 当前分支状态（分支名、领先/落后主分支的 commit 数）
- PR 状态（是否已创建、CI 状态、Review 进度）
- 与默认分支差异（文件变更统计、冲突检测）

### 发布流程参考
```
1. push branch              # 推送特性分支
   ↓
2. open/update PR           # 创建或更新 Pull Request
   ↓
3. ensure CI green          # 确保 CI 全部通过
   ↓
4. complete review          # 完成 Code Review
   ↓
5. merge with protection    # 按保护规则合并
   ↓
6. sync default branch      # 同步默认分支
   ↓
7. update docs              # 更新文档与 CHANGELOG.md
   ↓
8. determine SemVer bump    # 确定版本号（major.minor.patch）
   ↓
9. create annotated tag     # 创建带注释的 Git tag
   ↓
10. publish release notes   # 生成并发布 Release Notes
   ↓
11. post-release verify     # 发布后验证（监控/回滚就绪）
   ↓
12. clean up branches       # 清理已合并分支
```

**说明**：以上为参考模板，按项目特点调整（应用 / 类库 / monorepo / 部署链路）。

---
