# 现在进入 **DIAGNOSE 模式**

## 职责边界
- ✅ **允许**：诊断问题根因、定位错误、分析性能瓶颈、生成 XML 格式诊断报告
- ✅ **必须**：
  - 先读取对应的 XML 模板文件（`diagnose-{mode}.xml`）
  - 基于模板结构生成报告，确保格式一致性
  - 生成后从 `<lesson>` 节点提取要点，重要发现可记录到 Serena
- ❌ **禁止**：直接修改代码（应提供 fix_plan 后切换到 EXECUTE）、改变需求或架构
- 🔄 **切换**：问题已定位 → EXECUTE 实施修复；发现需求问题 → RESEARCH；需要架构调整 → PLAN
- 💡 **提醒**：根据问题复杂度选择分析深度（focus/quick/full），避免过度收集数据

---

## 工作流程

```
问题报告 → 选择模式 → 读取 XML 模板 → **主动添加 Debug 日志** → 并行调用 MCP 工具 → 收集分析数据 → 基于模板生成报告 → 保存到 diagnosis/
```

**关键原则**：当遇到计算错误、数值异常、逻辑问题时，**必须主动添加结构化 Debug 日志**，而不是等待用户请求。

---

## 诊断模式选择

### Focus 模式（1-2K tokens）
- **Token 范围**：1,000-2,000
- **适用场景**：明确错误、快速诊断
- **数据收集**：
  - 自动捕获的错误日志
  - 错误位置代码（ripgrep: 错误文件 ±20行）
  - 最近的 git diff（未提交变更）
  - 可选：mcp-debugger（仅当需要快速检查变量值时）
  - 跳过：项目结构、深度分析
- **模板文件**：`~/.claude/commands/diagnose-focus.xml`

### Quick 模式（3-5K tokens，默认）
- **Token 范围**：3,000-5,000
- **适用场景**：初步诊断、简单问题
- **数据收集**：
  - Focus 的所有内容
  - mcp-debugger（推荐，用于数据源不明、逻辑分支问题）
  - 项目结构概览（repomapper: PageRank 识别关键文件）
  - 错误模式搜索（ripgrep: ERROR|FAIL|Exception）
  - 安全扫描（semgrep: p/ci 规则集）
  - Git 变更（最近5个提交）
  - 跳过：AST分析、外部知识
- **模板文件**：`~/.claude/commands/diagnose-quick.xml`

### Full 模式（8-12K tokens）
- **Token 范围**：8,000-12,000
- **适用场景**：复杂问题、深度分析、性能问题
- **数据收集**：
  - Quick 的所有内容
  - mcp-debugger（优先使用，完整 stack trace + 多层 scopes + 单步执行）
  - 符号分析（serena: LSP 语义分析，相关函数/类的完整定义）
  - AST 分析（tree-sitter: 精确语法解析与代码结构问题）
  - 外部知识（exa get_code_context_exa: 10 亿+ 代码库搜索相关库文档、Stack Overflow）
  - 历史分析（git log: 相关文件的变更历史）
  - 依赖分析（package.json、requirements.txt 等）
- **模板文件**：`~/.claude/commands/diagnose-full.xml`

---

## 工具策略（并行调用）

### 运行时调试（优先）
```
mcp-debugger → 断点调试、单步执行、变量检查（适用于复杂问题、数据源不明）
```

### 核心诊断工具
```
ripgrep → 高性能定位错误位置（尊重 gitignore）
serena find_symbol → 理解相关符号实现（LSP 语义分析，多语言支持）
tree-sitter → 精确语法分析与复杂度检查（增量解析）
```

### 安全与质量
```
semgrep → 安全漏洞扫描与代码质量检测（beta，p/ci 规则集）
```

### 项目结构
```
repomapper → 识别关键文件与高耦合模块（PageRank 算法）
```

### 外部知识（Full 模式）
```
exa (get_code_context_exa) → 搜索相关库文档与 Stack Overflow（10 亿+ 代码库）
exa (web_search_exa) → 查询问题解决方案与最佳实践
```

### 历史分析
```
git log → 追踪相关文件的变更历史
git diff → 检查未提交变更
```

---

## 执行步骤

**Steps:**
1. Analyze problem type and auto-select mode (focus/quick/full)
2. **Read the corresponding XML template file**:
   - Focus: `~/.claude/commands/diagnose-focus.xml`
   - Quick: `~/.claude/commands/diagnose-quick.xml`
   - Full: `~/.claude/commands/diagnose-full.xml`
3. Auto-capture recent execution logs from Claude Code history
4. Execute targeted MCP analysis in parallel based on selected mode
5. Process and deduplicate collected data
6. Generate XML diagnosis report following the template structure
7. Save to `diagnosis/<timestamp>-<issue-type>-<mode>.xml`
8. Extract key findings from `<lesson>` node for memory storage (if critical)

---

## Debug 策略选择

**优先使用 mcp-debugger（运行时调试）**：
- 数据源不明确（多个 API 返回相似字段）
- 逻辑分支问题（不确定走了哪个分支）
- 复杂计算错误（需要查看中间步骤）

**使用 Debug Logging（快速诊断）**：
- 简单问题（已知要查看的变量）
- 无法使用断点的场景（异步/回调密集）

---

## Debug Logging 策略

**触发条件**（主动添加 debug 日志）：
- 计算结果异常（值不符合预期）
- 数值异常（负值、过大/过小、NaN/Infinity）
- 异步操作无效（调用后无变化）
- 数据源不明确（多个 API 返回相似字段）

**核心原则**：输出公式中的**所有变量**，而不仅仅是结果

```javascript
// ❌ Bad: 只输出结果
console.log('[DEBUG] result:', result);

// ✅ Good: 输出公式所有参数 + 上下文
console.log('[DEBUG] Calculation:', {
  formula: 'result = a - b + c',
  a: a,
  b: b,
  c: c,
  result: a - b + c,
  context: { id: entity.id, name: entity.name }
});

// ✅ Good: 多数据源对比（当有相似字段时）
console.log('[DEBUG] Data sources:', {
  field_from_api1: obj.field,
  field_from_api2: obj.alternativeField,
  actually_used: usedValue,
  context: { id: obj.id }
});
```

**关键要点**：
- 使用结构化对象（便于快速对比）
- 标注数据来源（区分相似变量）
- 包含上下文（ID、名称、类型）
- 显示公式（而非只有数值）

---

## 常见 DIAGNOSE 任务

**快速错误定位（Focus）**：
1. 自动捕获错误日志与堆栈
2. `ripgrep` → 定位错误代码位置
3. `git diff` → 检查最近变更
4. 生成 Focus 模式诊断报告

**安全漏洞分析（Quick）**：
1. `semgrep` (p/ci) → 扫描安全问题
2. `ripgrep` → 搜索相似漏洞模式
3. `repomapper` → 识别影响范围
4. 生成 Quick 模式诊断报告

**性能瓶颈诊断（Full）**：
1. `tree-sitter` → 识别高复杂度函数
2. `serena find_symbol` → 分析热点函数实现
3. `repomapper` → 识别高耦合依赖
4. `exa (get_code_context_exa)` → 搜索性能优化方案
5. 生成 Full 模式诊断报告

---

诊断完成后，根据 fix_plan 切换到 EXECUTE 实施修复。
