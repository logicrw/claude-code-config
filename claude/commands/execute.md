---
description: 进入 EXECUTE 模式
---

## ⚡ 核心职责

**在既定方案下实现与验证**，小步快跑、可回滚。

**产出**：可通过测试的最小变更集 + Git commit（中文信息 + 决策说明）

---

## ✅ 执行清单

### 1. 修改前检查
```bash
# 理解当前实现
serena find_symbol <target> --include-body
ripgrep "<api_usage>"  # 搜索所有调用点
```

### 2. 实施修改（最小 diff）
- 严格按设计文档编码
- **先贴 diff/patch 供审核，确认后再应用**

### 3. 修改后验证
```bash
# 检查影响范围
serena find_referencing_symbols <target>
semgrep scan           # 安全检测
tree-sitter validate   # 语法验证

# 运行测试（必须）
npm test  # 或项目测试命令
```

### 4. 测试驱动流程
```
✅ 测试通过 → 提交 commit（增强格式，见下文）
❌ 测试失败 → 切换到 DIAGNOSE，添加 debug 日志重新诊断
```

### 5. Git Commit 增强格式
```bash
git commit -m "feat: 添加 JWT 认证模块

- 决策：选择 JWT 而非 Session（见 docs/decisions/2025-10-18-jwt-auth.md）
- 关联 Serena：strategy_jwt_vs_session
- TODO: 添加 refresh token 轮转机制

相关文件：
- src/auth/jwt.ts (新增)
- src/middleware/auth.ts (修改)
"
```

### 6. 记忆更新（如遇坑）
```bash
# 遇到技术陷阱 → 立即记录到 Serena
serena write_memory "pitfalls_jwt_concurrent_refresh" """
问题：并发刷新 refresh token 导致竞态条件
原因：旧 token 立即失效，并发请求失败
解决：sliding window + 5 秒宽限期
"""

# 文档更新（如需）
# 同步更新 docs/architecture/ 或 docs/api/
```

---

## ⚠️ 异常处理

**遇到设计/需求冲突时**：
1. 立即停止修改
2. 输出问题诊断
3. 建议切回 RESEARCH 或 PLAN

---

<details>
<summary>🛠️ 工具速查（点击展开）</summary>

| 阶段 | 工具 | 用途 |
|------|------|------|
| **修改前** | serena, ripgrep | 理解实现 + 搜索调用 |
| **验证** | semgrep, tree-sitter | 安全扫描 + 语法检查 |
| **提交** | Git | 增强格式 commit（含决策说明） |
| **记忆** | serena | 记录陷阱（如遇坑） |

</details>
