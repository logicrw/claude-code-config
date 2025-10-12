# 现在进入 **PLAN 模式**

## 职责边界
- ✅ **允许**：制定方案、接口契约、任务拆分与测试计划
- ✅ **必须**：
  - 在设计文档记录目标、约束和取舍；遵循 TDD 原则（测试先行）
  - 重要决策需创建决策记录（简化 ADR）到 `~/basic-memory/<project>/decisions/`
- ✅ **产物**：
  - 设计方案 → `~/basic-memory/<project>/designs/`
  - 决策记录 → `~/basic-memory/<project>/decisions/`（重要技术选型时）
  - 任务追踪 → Claude Code Todos（会话内）
- ❌ **禁止**：直接修改业务代码
- 🔄 **切换**：方案确认后进入 EXECUTE；若需求变化则回到 RESEARCH
- 💡 **提醒**：参考现有代码，保持设计简洁，避免过度设计

---

## 工具策略（影响评估优先）

### 评估修改影响范围
```bash
serena find_symbol → 理解当前实现（LSP 语义分析，多语言支持）
serena find_referencing_symbols → 找到所有调用者（符号级引用）
repomapper → 识别高耦合文件（PageRank 算法）
```

### 识别技术风险
```bash
tree-sitter → 精确语法解析，分析代码复杂度（增量分析）
semgrep → 安全漏洞检测，预检测代码质量问题
Serena → 查询相关约束与陷阱（项目级记忆）
```

### 参考外部方案
```bash
exa (get_code_context_exa) → 搜索业界代码示例与实现（10 亿+ 代码库）
exa (web_search_exa) → 查询设计模式与架构方案
ripgrep → 高性能搜索项目内类似实现（尊重 gitignore）
```

