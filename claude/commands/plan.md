---
description: 进入 PLAN 模式
---

## 🎨 核心职责

**形成可执行的技术方案**，包含测试计划、风险评估与任务拆分。

**产出**：设计文档（`docs/architecture/`）+ 决策记录（`docs/decisions/`，重要选型）

---

## ✅ 执行清单

### 1. 评估修改影响
```bash
# 找到所有关联代码
serena find_symbol <target>
serena find_referencing_symbols <target>
repomapper  # 识别高耦合文件
```

### 2. 识别技术风险
```bash
# 扫描潜在问题
tree-sitter analyze <file>  # 复杂度分析
semgrep scan                 # 安全检测
serena read_memory "constraints_<topic>"
serena read_memory "pitfalls_<module>"
```

### 3. 参考外部方案
```bash
# 搜索业界实践
exa get_code_context "类似实现"
exa web_search "设计模式"
ripgrep "项目内类似代码"

# 查询通用技术选型
serena read_memory "strategy_<comparison>"
serena read_memory "selection_<category>"
```

### 4. 撰写设计文档
**必含内容**（遵循 TDD 原则）：
- 目标与假设
- 方案选项对比（≥2 个备选）
- 验收标准（可测试）
- 测试计划（单元/集成/边界）

**保存路径**：`docs/architecture/YYYY-MM-DD-<topic>.md`

### 5. 决策记录协议（AI 自动触发）
**触发条件**：重要技术选型、架构调整

**AI 自动询问记录方式**：
1. 创建正式 ADR (`docs/decisions/YYYY-MM-DD-<topic>.md`)
2. 记录到 Serena (`strategy_<topic>` / `selection_<category>`) ← 如果是通用对比
3. 仅保留在 commit message ← 如果是简单选择

**默认选项**：1（创建 ADR）

---

## 🔄 模式切换

- ✅ 用户确认方案 → **EXECUTE**
- ⚠️ 需求变化 → **RESEARCH**

---

<details>
<summary>🛠️ 工具速查（点击展开）</summary>

| 工具 | 用途 | 关键能力 |
|------|------|---------|
| **serena** | 符号级影响分析 | 找调用者、查约束 |
| **repomapper** | 识别高耦合模块 | PageRank 算法 |
| **tree-sitter** | 语法与复杂度 | AST 分析 |
| **semgrep** | 安全风险扫描 | 代码质量检测 |
| **exa** | 外部方案搜索 | 10 亿+ 代码库 |

</details>
