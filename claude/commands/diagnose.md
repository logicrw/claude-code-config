---
description: 进入 DIAGNOSE 模式
---

## 🔬 核心职责

**诊断问题根因**，定位错误、分析性能瓶颈，生成 XML 诊断报告。

**产出**：根因分析报告 + 陷阱记录（重要问题记录到 Serena 或 `docs/troubleshooting/`）

---

## ✅ 执行清单

### 1. 选择诊断深度
| 模式 | Token | 适用场景 |
|------|-------|---------|
| **focus** | 1K-2K | 明确错误、快速定位 |
| **quick** | 3K-5K | 初步诊断（默认） |
| **full** | 8K-12K | 复杂问题、深度分析 |

### 2. 运行时调试（优先）
```bash
# 适用于：数据源不明、逻辑分支、复杂计算
mcp-debugger set_breakpoint <file> <line>
mcp-debugger step_over
mcp-debugger inspect_variable <name>
```

### 3. 定位问题代码
```bash
# 搜索错误位置
ripgrep "error_pattern"
serena find_symbol <error_source>
tree-sitter analyze <file>  # 语法问题
```

### 4. 分析影响范围
```bash
# 评估关联模块
repomapper                           # 高耦合文件
serena find_referencing_symbols      # 调用关系
semgrep scan                         # 安全漏洞
```

### 5. 追踪历史变更
```bash
git log --follow <file>
git diff HEAD~5 <file>
```

### 6. 生成诊断报告
- 读取模板：`~/.claude/commands/diagnose-{mode}.xml`
- 生成 XML 报告（在对话中展示）

### 7. 记录技术陷阱（重要问题）
```bash
# 立即记录到 Serena（通用陷阱）
serena write_memory "pitfalls_jwt_concurrent_refresh" """
问题：并发刷新 refresh token 导致竞态条件
原因：旧 token 立即失效，并发请求失败
解决：sliding window + 5 秒宽限期
测试：ab -n 100 -c 10 /api/refresh
"""

# 或保存到 Git（项目特定的重大问题）
# 路径：docs/troubleshooting/YYYY-MM-DD-<issue>.md
```

---

## 🔄 模式切换

- ✅ 问题已定位 → **EXECUTE** 修复
- ⚠️ 发现需求问题 → **RESEARCH**
- 🏗️ 需要架构调整 → **PLAN**

---

---

## 诊断模式选择

### 模式对比速查表

| 模式 | Token 范围 | 适用场景 | 核心工具 | 数据收集范围 |
|------|-----------|---------|---------|------------|
| **Focus** | 1K-2K | 明确错误、快速诊断 | ripgrep + git diff | 错误日志 + 错误代码 ±20 行 + 未提交变更 |
| **Quick** | 3K-5K | 初步诊断、简单问题（默认） | + mcp-debugger + repomapper + semgrep | + 项目结构 + 错误模式搜索 + 安全扫描 + 最近 5 次提交 |
| **Full** | 8K-12K | 复杂问题、深度分析、性能问题 | + serena + tree-sitter + exa | + 符号分析 + AST 分析 + 外部知识 + 历史分析 + 依赖分析 |

---

## 工具策略（并行调用）

### 运行时调试（优先）
```bash
mcp-debugger → 断点调试、单步执行、变量检查（适用于复杂问题、数据源不明）
```

### 核心诊断工具
```bash
ripgrep → 高性能定位错误位置（尊重 gitignore）
serena find_symbol → 理解相关符号实现（LSP 语义分析，多语言支持）
tree-sitter → 精确语法分析与复杂度检查（增量解析）
```

### 安全与质量
```bash
semgrep → 安全漏洞扫描与代码质量检测（beta，p/ci 规则集）
```

### 项目结构
```bash
repomapper → 识别关键文件与高耦合模块（PageRank 算法）
```

### 外部知识（Full 模式）
```bash
exa (get_code_context_exa) → 搜索相关库文档与 Stack Overflow（10 亿+ 代码库）
exa (web_search_exa) → 查询问题解决方案与最佳实践
```

### 历史分析
```bash
git log → 追踪相关文件的变更历史
git diff → 检查未提交变更
```

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
