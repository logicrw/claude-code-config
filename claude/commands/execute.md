# 现在进入 **EXECUTE 模式**

## 职责边界
- ✅ **允许**：在既定方案下进行编码、运行/修复测试、更新文档与任务状态
- ✅ **必须**：
  - 先贴出 diff/patch（英文代码块），确认后再应用
  - **提交信息必须使用中文** + 范围标签（如 `feat:`/`fix:`/`docs:`）
  - 完成后更新文档（Tests 结果、Tasks 状态）
  - 在重大进展后提醒进行版本管理（推送分支 → 创建 PR → CI 通过 → 合并 → 同步默认分支 → 更新 Changelog → 创建 Tag/Release → 验证与回滚准备）
- ❌ **禁止**：越权改动需求或方案，未确认情况下大范围重构或删除
- 💡 **提醒**：严格参考设计文档执行，保持与当前实现的兼容性，避免破坏已有功能
- ⚠️ **异常处理**：若遇到设计/需求冲突，立即停止，输出 Diagnosis（中文）与 Next Steps（英文），并建议切回合适模式

---

## 工具策略（编码 → 验证 → 记忆）

### 修改前
```
serena find_symbol (include_body=true) → 理解当前实现（LSP 符号级分析）
ripgrep → 高性能搜索所有调用点（尊重 gitignore）
```

### 实施修改
```
严格按照设计文档编码
保持最小 diff 原则
```

### 修改后验证
```
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
```
贴 diff/patch 供审核
确认后提交（中文 commit message）
```

### 记忆更新（如需）
```
Serena write_memory → 记录重要修复/发现（项目级记忆）
Basic Memory → 记录阶段性进展（本地 Markdown）
```

---

## 常见 EXECUTE 任务

**实现新功能**：
1. 按设计文档编码
2. `semgrep` + `tree-sitter` → 验证
3. 运行测试 → 失败则切换到 DIAGNOSE
4. 贴 diff → 确认 → 提交

**修复 Bug**：
1. `ripgrep` → 定位错误位置
2. `serena find_symbol` → 理解实现
3. `exa (get_code_context_exa)` → 搜索正确用法示例（如需）
4. 实施修复
5. 运行测试 → 失败则切换到 DIAGNOSE
6. `semgrep` → 验证
7. `Serena write_memory` (pitfalls) → 记录反模式

**重构代码**：
1. `serena find_referencing_symbols` → 确认影响范围
2. 实施重构（保持向后兼容）
3. `tree-sitter` → 验证复杂度降低
4. `semgrep` → 验证质量提升
5. 运行测试 → 失败则切换到 DIAGNOSE

---

完成后更新文档与任务状态，重大进展提醒版本管理。
