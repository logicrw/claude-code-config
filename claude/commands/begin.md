---
description: 会话启动
---

## 会话启动

### 会话恢复模式

**续接上次工作？** → 使用 **`/begin --resume`**

AI 将自动执行：
1. `git log -10` - 查看最近提交
2. `git status` - 检查未提交改动
3. `git diff --stat` - 统计变更文件
4. 搜索代码中的 `TODO` / `FIXME` 注释
5. 总结进展并询问："从哪里继续？"

---

## 模式选择指南

**不清楚需求或需要理解代码？** → **RESEARCH** 模式
- 产物：研究笔记 `docs/research/`

**需求明确，准备设计方案？** → **PLAN** 模式
- 产物：设计方案 `docs/architecture/` + 架构决策 `docs/decisions/`

**方案已定，开始编码？** → **EXECUTE** 模式
- 产物：代码变更 + 测试通过的 Git commit

**遇到错误或性能问题？** → **DIAGNOSE** 模式
- 产物：根因分析 + 问题诊断 `docs/troubleshooting/`
