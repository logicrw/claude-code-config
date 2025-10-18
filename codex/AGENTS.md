# RPE 人机协作协议

> 以最小"护栏 + 流程"确保代码协作**安全、可控、可审计**;平衡灵活性与质量一致性。

---

## 🚀 核心理念

1. **模式驱动**:每条回复首行声明 `[MODE: RESEARCH|PLAN|EXECUTE|DIAGNOSE]`,默认 RESEARCH
2. **最小 diff**:所有文件改动先贴 patch 审核,确认后再应用
3. **默认中文**:叙述使用中文,仅技术标准用英文(模式声明、代码块、Git 前缀)
4. **副作用控制**:任何有副作用的操作需先展示执行步骤与影响,获批后再执行

---

## 📋 四大模式

| 模式 | 核心职责 | 主要产出 | 改代码 |
|------|---------|---------|--------|
| 🔍 **RESEARCH** | 理解问题本质 | 研究笔记 | ❌ |
| 🎨 **PLAN** | 设计技术方案 | 设计文档 | ❌ |
| ⚡ **EXECUTE** | 实现与验证 | 可测代码 | ✅ |
| 🔬 **DIAGNOSE** | 根因分析 | 诊断报告 | ❌ |

---

## 💾 文档产出

各模式产出保存到 `docs/`:
- **RESEARCH**:研究笔记 `docs/research/`
- **PLAN**:设计文档 `docs/architecture/`、架构决策 `docs/decisions/`
- **EXECUTE**:接口规范 `docs/api/`
- **DIAGNOSE**:问题诊断 `docs/troubleshooting/`

---

## 🛠️ 工具生态

| 分类 | 工具 | 核心能力 |
|------|------|---------|
| **调试** | mcp-debugger | 断点、单步、变量检查 |
| **记忆** | serena | 符号分析 + 通用知识库 |
| **分析** | ripgrep | 高性能全文搜索 |
| | tree-sitter | AST 语法解析 |
| | semgrep | 安全漏洞扫描 |
| | repomapper | 关键文件识别 |
| **检索** | ctx | 上下文打包 |
| | exa | 代码示例 + 网络搜索 |

---

## 🔄 典型工作流

```
┌─────────────────┐
│  🔍 RESEARCH    │ → 需求澄清与风险识别
└────────┬────────┘
         ↓ 必要时记录到 Serena
┌─────────────────┐
│  🎨 PLAN        │ → 方案设计与测试计划
└────────┬────────┘
         ↓ 用户确认 + 决策记录协议
┌─────────────────┐
│  ⚡ EXECUTE     │ → diff 审核 → 应用 → Git commit
└────────┬────────┘
         ↓ 测试失败
┌─────────────────┐
│  🔬 DIAGNOSE    │ → 根因分析 → 记录陷阱到 Serena
└─────────────────┘
         ↓
      回到 EXECUTE
```

### 模式切换决策

| 当前状态 | 触发条件 | 切换到 |
|---------|---------|--------|
| 任意 | 需求不清、范围模糊 | 🔍 RESEARCH |
| RESEARCH | 需求已明确、需设计 | 🎨 PLAN |
| PLAN | 方案确认、准备实现 | ⚡ EXECUTE |
| EXECUTE | 测试失败、出现 bug | 🔬 DIAGNOSE |
| DIAGNOSE | 问题已定位 | ⚡ EXECUTE |

---

## 🔍 RESEARCH 模式

**理解问题本质**,澄清需求边界,识别技术风险。

**核心流程**:项目背景 → 代码结构 → 具体实现 → (外部知识) → 研究结论

**关键工具**:
- 项目背景:`git log --grep` / `serena read_memory`
- 代码结构:`repomapper` / `serena get_symbols_overview`
- 具体实现:`ripgrep` / `serena find_symbol`
- 外部知识:`exa web_search` / `exa get_code_context`

**产出**:口头总结 + 研究文档(`docs/research/`,如需) + Serena 记忆(通用知识,如需)

**停止条件**:关键不确定性收敛 / 出现阻塞点 / 达到时间盒

---

## 🎨 PLAN 模式

**形成可执行的技术方案**,包含测试计划、风险评估与任务拆分。

**核心流程**:评估影响 → 识别风险 → 参考方案 → 撰写文档 → 决策记录

**关键工具**:
- 评估影响:`serena find_referencing_symbols` / `repomapper`
- 识别风险:`tree-sitter analyze` / `semgrep scan` / `serena read_memory pitfalls_*`
- 参考方案:`exa` / `ripgrep` / `serena read_memory strategy_*`

**设计文档必含**:
- 目标与假设
- 方案选项对比(≥2 个备选)
- 验收标准(可测试)
- 测试计划(单元/集成/边界)

**决策记录协议**(重要选型时 AI 自动询问):
1. 创建 ADR (`docs/decisions/YYYY-MM-DD-<topic>.md`) ← 默认
2. 记录 Serena (`strategy_<topic>`) ← 通用对比
3. 仅 commit message ← 简单选择

**产出**:设计文档(`docs/architecture/`) + 决策记录(`docs/decisions/`,重要选型)

---

## ⚡ EXECUTE 模式

**在既定方案下实现与验证**,小步快跑、可回滚。

**核心流程**:修改前检查 → 实施修改 → 修改后验证 → 测试驱动 → Git commit → 记忆更新

**关键工具**:
- 修改前:`serena find_symbol --include-body` / `ripgrep`
- 修改后:`serena find_referencing_symbols` / `semgrep scan` / `tree-sitter validate`
- 测试:`npm test`(或项目测试命令)

**修改原则**:
- 严格按设计文档编码
- **先贴 diff/patch 供审核,确认后再应用**

**测试驱动流程**:
```
✅ 测试通过 → Git commit(增强格式)
❌ 测试失败 → 切换到 DIAGNOSE,添加 debug 日志
```

**Git Commit 增强格式**:
```bash
git commit -m "feat: 添加 JWT 认证模块

- 决策:选择 JWT 而非 Session(见 docs/decisions/2025-10-18-jwt-auth.md)
- 关联 Serena:strategy_jwt_vs_session
- TODO:添加 refresh token 轮转机制

相关文件:
- src/auth/jwt.ts (新增)
- src/middleware/auth.ts (修改)
"
```

**遇坑处理**:立即记录到 Serena `pitfalls_*` 或 `docs/troubleshooting/`

**异常处理**:设计/需求冲突时立即停止 → 输出诊断 → 切回 RESEARCH/PLAN

**产出**:可通过测试的最小变更集 + Git commit(中文信息 + 决策说明)

---

## 🔬 DIAGNOSE 模式

**诊断问题根因**,定位错误、分析性能瓶颈。

**核心流程**:选择深度 → 运行时调试 → 定位代码 → 分析影响 → 追踪历史 → 生成报告 → 记录陷阱

**报告模板**:读取 `~/.codex/diagnose-{mode}.xml` (focus/quick/full),生成 XML 诊断报告

### 诊断深度选择

| 模式 | Token | 适用场景 | 核心工具 | 数据收集范围 |
|------|-------|---------|---------|------------|
| **focus** | 1K-2K | 明确错误、快速定位 | ripgrep + git diff | 错误日志 + 错误代码 ±20 行 + 未提交变更 |
| **quick** | 3K-5K | 初步诊断(默认) | + mcp-debugger + repomapper + semgrep | + 项目结构 + 错误模式搜索 + 安全扫描 + 最近 5 次提交 |
| **full** | 8K-12K | 复杂问题、深度分析 | + serena + tree-sitter + exa | + 符号分析 + AST 分析 + 外部知识 + 历史分析 + 依赖分析 |

### Debug Logging 策略

**触发条件**(主动添加 debug 日志):
- 计算结果异常(值不符合预期)
- 数值异常(负值、过大/过小、NaN/Infinity)
- 异步操作无效(调用后无变化)
- 数据源不明确(多个 API 返回相似字段)

**核心原则**:输出公式中的**所有变量**,而不仅仅是结果

```javascript
// ❌ Bad: 只输出结果
console.log('[DEBUG] result:', result);

// ✅ Good: 输出公式所有参数 + 上下文
console.log('[DEBUG] Calculation:', {
  formula: 'result = a - b + c',
  a: a,
  b: b,
  c: c,
  result: a - b + c,
  context: { id: entity.id, name: entity.name }
});

// ✅ Good: 多数据源对比(当有相似字段时)
console.log('[DEBUG] Data sources:', {
  field_from_api1: obj.field,
  field_from_api2: obj.alternativeField,
  actually_used: usedValue,
  context: { id: obj.id }
});
```

**关键要点**:
- 使用结构化对象(便于快速对比)
- 标注数据来源(区分相似变量)
- 包含上下文(ID、名称、类型)
- 显示公式(而非只有数值)

**产出**:根因分析报告 + 陷阱记录(重要问题记录到 Serena 或 `docs/troubleshooting/`)

---

## 📐 核心规则

<details>
<summary><strong>语言与输出规范</strong>(点击展开)</summary>

| 内容类型 | 语言 | 格式规范 |
|---------|------|---------|
| 叙述、解释 | 中文 | 中文标点 |
| 模式声明 | 英文 | `[MODE: ...]` |
| 代码块 | 英文 | 必须标注语言 |
| Git commit | 中文主题 + 英文前缀 | `feat: 添加认证模块` |
| PR/Issue | 英文 | 便于跨协作 |
| 文件名 | 英文 | `kebab-case` |
| 代码注释 | 短英文,长中文 | 避免混排 |

</details>

<details>
<summary><strong>质量门槛</strong>(点击展开)</summary>

### 执行前检查
- [ ] 变更范围最小化
- [ ] 敏感信息已脱敏
- [ ] 测试用例已定义
- [ ] 关键路径已验证
- [ ] 回滚方案已就绪

### 执行后检查
- [ ] 所有测试通过
- [ ] 文档已同步更新
- [ ] 日志/监控已验证
- [ ] 风险项已复核
- [ ] 用户已明确确认

</details>

<details>
<summary><strong>安全边界</strong>(点击展开)</summary>

### 敏感信息处理
**🔴 禁止输出**:`.env` 文件、API keys、密码、Token、PII

**✅ 正确做法**:
```bash
DATABASE_URL=postgresql://***:***@localhost/db  # 占位符
process.env.API_KEY  # 引用变量名,不输出值
```

### 副作用操作
| 操作 | 要求 |
|------|------|
| 安装依赖 | 列出包名、版本、许可证 |
| 修改配置 | 展示 diff、说明影响 |
| 批量删除 | 列出清单、提供回滚 |
| 数据迁移 | 备份 + 回滚脚本 |

### 破坏性操作
- ⚠️ 谨慎:`rm -rf`、`DROP TABLE`、大规模重构
- 🔄 必须:回滚方案 + 备份路径
- ✋ 二次确认:高风险操作需明确输入"确认执行"

</details>
