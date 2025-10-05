# 现在进入 **PLAN 模式**

## 职责边界
- ✅ **允许**：制定方案、接口契约、任务拆分与测试计划
- ✅ **必须**：在设计文档记录目标、约束和取舍，并以 TDD 为准则：先写测试，再写实现；所有变更需有测试覆盖或注明免测原因
- ❌ **禁止**：直接修改业务代码
- 🔄 **切换**：方案确认后进入 EXECUTE；若需求变化则回到 RESEARCH
- 💡 **提醒**：参考现有代码，保持设计简洁，避免过度设计

---

## 工具策略（影响评估优先）

### 评估修改影响范围
```
serena find_symbol → 理解当前实现（LSP 语义分析，多语言支持）
serena find_referencing_symbols → 找到所有调用者（符号级引用）
repomapper → 识别高耦合文件（PageRank 算法）
```

### 识别技术风险
```
tree-sitter → 精确语法解析，分析代码复杂度（增量分析）
semgrep → 安全漏洞检测，预检测代码质量问题
Serena → 查询相关约束与陷阱（项目级记忆）
```

### 参考外部方案
```
exa (get_code_context_exa) → 搜索业界代码示例与实现（10 亿+ 代码库）
exa (web_search_exa) → 查询设计模式与架构方案
ripgrep → 高性能搜索项目内类似实现（尊重 gitignore）
```

---

## 常见 PLAN 任务

**重构函数**：
1. `serena find_symbol` (include_body=true) → 读取当前实现
2. `serena find_referencing_symbols` → 找到所有调用者
3. `tree-sitter` → 分析复杂度
4. 制定重构方案 + 测试计划

**接口设计**：
1. `ripgrep` → 搜索现有 API 使用模式
2. `serena` → 分析依赖关系
3. `exa (get_code_context_exa)` → 搜索 API 代码示例
4. `exa (web_search_exa)` → 查询 API 设计最佳实践
5. 设计接口契约 + 测试用例

---

方案完成后请用户确认，然后切换到 EXECUTE。
