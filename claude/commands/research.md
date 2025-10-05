# 现在进入 **RESEARCH 模式**

## 职责边界
- ✅ **允许**：澄清需求、阅读代码与文档、识别风险与未知
- ✅ **必须**：产出 Notes 与 Open Questions（英文 bullet）
- ❌ **禁止**：编写业务代码、生成最终实现或测试用例
- 🛑 **停止条件**：不确定性收敛、出现阻塞依赖、到达 time-box

---

## 工具策略（优先级排序）

### 理解项目结构
```
repomapper → 识别关键文件（PageRank 算法排序）
ctx directory-list → 列举目录结构
```

### 理解代码实现
```
serena get_symbols_overview → 获取文件顶层结构（LSP 语义理解）
serena find_symbol → 深入重要符号（多语言支持）
ripgrep → 高性能全文搜索关键词/API（尊重 gitignore）
```

### 获取历史背景
```
Basic Memory → 检索开发日志与决策记录（本地 Markdown，知识图谱导航）
Serena → 查询项目约束与陷阱（constraints/pitfalls，符号级记忆）
```

### 外部知识补充
```
exa (get_code_context_exa) → 搜索代码示例与最新文档（降低幻觉）
exa (web_search_exa) → 查询最佳实践与设计模式
ctx → 打包项目上下文供跨 AI 对话使用（如需）
```

---

## 常见 RESEARCH 任务

**理解新模块**：
1. `repomapper` → 识别关键文件
2. `serena get_symbols_overview` → 理解顶层结构
3. `serena find_symbol` (depth=1-2) → 深入重要符号
4. `Basic Memory` → 检索相关历史

**可行性评估**：
1. `ripgrep` → 搜索类似实现
2. `exa (get_code_context_exa)` → 搜索代码示例
3. `exa (web_search_exa)` → 查询业界方案
4. `serena` → 分析现有架构约束
5. 输出 **Options & Trade-offs**

---

准备好进入 PLAN 或继续深入 RESEARCH？
