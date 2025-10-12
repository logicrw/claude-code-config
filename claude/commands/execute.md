# 现在进入 **EXECUTE 模式**

## 职责边界
- ✅ **允许**：在既定方案下进行编码、运行/修复测试、更新文档与任务状态
- ✅ **必须**：
  - 先贴出 diff/patch（代码块），确认后再应用
  - **提交信息必须使用中文** + 范围标签（如 `feat:`/`fix:`/`docs:`）
  - 完成后更新文档（测试结果、任务状态）
  - 在重大进展后提醒版本管理流程（详见 `../CLAUDE.md §8 版本管理与发布提醒`）
- ✅ **产物**：
  - 代码变更（最小 diff）
  - 测试通过的 commit
  - Claude Code Todos（任务追踪）
  - Basic Memory（重要进展记录，如需跨会话保存）
- ❌ **禁止**：越权改动需求或方案，未确认情况下大范围重构或删除
- 💡 **提醒**：严格参考设计文档执行，保持与当前实现的兼容性，避免破坏已有功能
- ⚠️ **异常处理**：若遇到设计/需求冲突，立即停止，输出诊断结论与后续步骤，并建议切回合适模式

---

## 工具策略（编码 → 验证 → 记忆）

### 修改前
```bash
serena find_symbol (include_body=true) → 理解当前实现（LSP 符号级分析）
ripgrep → 高性能搜索所有调用点（尊重 gitignore）
```

### 实施修改
```bash
严格按照设计文档编码
保持最小 diff 原则
```

### 修改后验证
```bash
serena find_referencing_symbols → 检查是否需同步修改（符号级引用）
semgrep → 安全漏洞扫描/代码质量检测
tree-sitter → 精确语法验证与复杂度分析（增量解析）
运行测试（必须）
```

### 测试驱动流程
```
运行测试 → 通过则提交 commit
         → 失败则切换到 DIAGNOSE，主动添加 debug 日志，生成诊断报告
```

### 提交前
```bash
贴 diff/patch 供审核
确认后提交（中文 commit message）
```

### 记忆更新（如需）
```bash
# Serena - 记录重要修复/发现（项目级记忆）
mcp__serena__write_memory(
  memory_name="pitfalls_<module>",  # 或 constraints_<topic>
  content="..."
)

# Basic Memory - 记录阶段性进展（跨会话保存）
mcp__basic-memory__write_note(folder="designs", ...)  # 重要进展
mcp__basic-memory__edit_note(operation="append", ...)  # 追加到原有文档
```

