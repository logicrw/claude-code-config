---
description: 会话结束收工清单
---

## 🏁 会话收工

### 收工前建议

#### 1. 检查 Git 状态（最重要）
```bash
git status
git diff --stat
```

**如有未提交变更**：
- ✅ 确认是否需要提交（功能完整、测试通过）
- ✅ 检查是否遗漏测试
- ✅ 更新相关文档（docs/）
- ⚠️ 如未完成：在代码中添加 TODO 注释标记断点

**添加 TODO 注释示例**：
```typescript
// TODO: 完成 refresh token 轮转机制
//   - 实现 sliding window 策略
//   - 添加并发测试
//   - 更新文档 docs/api/auth.md
function refreshToken() { ... }
```

#### 2. 记录重要发现到 Serena（如有）
```bash
# 仅记录通用的技术陷阱、约束、优化策略
serena write_memory "pitfalls_<module>" """
问题描述...
原因分析...
解决方案...
"""
```

### 下次启动提醒
使用 **`/begin --resume`** 自动恢复上下文：
- 自动检查 Git 状态（未提交改动）
- 自动搜索 TODO 注释
- 自动总结进展
- 询问："从哪里继续？"
