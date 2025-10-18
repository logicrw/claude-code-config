---
description: 进入 RESEARCH 模式
---

## 🔍 核心职责

**理解问题本质**，澄清需求边界，识别技术风险与未知项。

**产出**：口头总结 + 研究文档（`docs/research/`，如需）+ Serena 记忆（通用知识，如需）

---

## ✅ 执行清单

### 1. 理解项目背景
```bash
# 检查 Git 历史
git log --oneline -20
git log --grep="关键词"

# 查询 Serena 通用知识
serena read_memory "strategy_<topic>"
serena read_memory "constraints_<topic>"
```

### 2. 分析代码结构
```bash
# 识别关键文件
repomapper
serena get_symbols_overview <file>
```

### 3. 定位具体实现
```bash
# 搜索相关代码
ripgrep "关键词"
serena find_symbol <symbol_name>
```

### 4. 补充外部知识（如需）
```bash
# 搜索最佳实践
exa web_search "技术方案"
exa get_code_context "API 用法"
```

### 5. 输出研究结论
- 口头总结核心发现与待解问题
- 有价值的研究成果保存到 `docs/research/YYYY-MM-DD-<topic>.md`
- 通用技术知识记录到 Serena（如 `strategy_*`）

---

## 🛑 停止条件

- 关键不确定性已收敛
- 出现外部依赖或需用户决策的阻塞点
- 达到预设时间盒（time-box）

---

<details>
<summary>🛠️ 工具速查（点击展开）</summary>

| 工具 | 用途 | 示例 |
|------|------|------|
| **Git** | 历史记录查询 | `git log`, `git grep` |
| **repomapper** | 识别关键文件（PageRank） | `repomapper` |
| **serena** | 代码分析 + 通用知识 | `find_symbol`, `read_memory "strategy_*"` |
| **ripgrep** | 高性能全文搜索 | `ripgrep "pattern"` |
| **exa** | 外部知识搜索 | `web_search`, `get_code_context` |
| **ctx** | 上下文打包 | `ctx directory-list` |

</details>
