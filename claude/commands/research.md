# 现在进入 **RESEARCH 模式**

## 职责边界
- ✅ **允许**：澄清需求、阅读代码与文档、识别风险与未知
- ✅ **必须**：产出研究笔记与待解问题
- ✅ **产物**：研究笔记（会话内）→ 重要发现可保存到 `~/basic-memory/<project>/designs/`（跨会话）
- ❌ **禁止**：编写业务代码、生成最终实现或测试用例
- 🛑 **停止条件**：不确定性收敛、出现阻塞依赖、到达时间盒（time-box）

---

## 工具策略（优先级排序）

### 理解项目结构
```bash
repomapper → 识别关键文件（PageRank 算法排序）
ctx directory-list → 列举目录结构
```

### 理解代码实现
```bash
serena get_symbols_overview → 获取文件顶层结构（LSP 语义理解）
serena find_symbol → 深入重要符号（多语言支持）
ripgrep → 高性能全文搜索关键词/API（尊重 gitignore）
```

### 获取历史背景
```bash
# Basic Memory - 检索开发日志与决策记录
mcp__basic-memory__search_notes("关键词")  # 跨项目搜索
mcp__basic-memory__recent_activity(project="<name>", timeframe="7d")  # 最近活动

# Serena - 查询项目约束与陷阱
mcp__serena__read_memory(memory_name="constraints_<topic>")
```

### 外部知识补充
```bash
exa (get_code_context_exa) → 搜索代码示例与最新文档（降低幻觉）
exa (web_search_exa) → 查询最佳实践与设计模式
ctx → 打包项目上下文供跨 AI 对话使用（如需）
```

