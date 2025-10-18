# Claude Code Configuration

> 基于 RPE 人机协作协议的 Claude Code 配置体系，提供可审计、可回滚的软件工程协作流程。

---

## 🎯 核心理念

**安全、可控、可审计** - 通过四大模式 + 提案式切换 + 结构化文档，确保人机协作过程：
- ✅ 每一步都可追溯
- ✅ 关键决策需确认
- ✅ 所有变更可回滚

---

## 📂 目录结构

```
claude-code-config/
├── claude/
│   ├── CLAUDE.md              # RPE 协作协议（核心 - 16KB）
│   └── commands/              # 模式指令与诊断模板（已精简 60%+ Token）
│       ├── begin.md           # 会话启动检查
│       ├── research.md        # RESEARCH 模式指令
│       ├── plan.md            # PLAN 模式指令
│       ├── execute.md         # EXECUTE 模式指令
│       ├── diagnose.md        # DIAGNOSE 模式指令
│       ├── wrap.md            # 会话结束收工清单
│       ├── diagnose-focus.xml # Focus 诊断模板 (1-2K tokens)
│       ├── diagnose-quick.xml # Quick 诊断模板 (3-5K tokens)
│       └── diagnose-full.xml  # Full 诊断模板 (8-12K tokens)
├── codex/
│   ├── AGENTS.md              # Codex 单文件协议（292 行,精简 36%）
│   ├── diagnose-focus.xml     # Focus 诊断模板
│   ├── diagnose-quick.xml     # Quick 诊断模板
│   └── diagnose-full.xml      # Full 诊断模板
├── mcp.json.template          # Claude Code MCP 配置模板
├── config.toml.template       # Serena/Codex MCP 配置模板
└── README.md                  # 本文档
```

**配置文件说明**：
- ✅ 提供 `mcp.json.template` 和 `config.toml.template` 作为参考
- ✅ 模板中已移除个人隐私信息（路径、API Key 等）
- ✅ 请根据实际环境修改模板后复制到 `~/.claude/` 或 `~/.codex/`

---

## 🚀 快速开始

### 方案 A：全局配置（推荐）

适用于所有项目使用相同的 RPE 协作协议。

```bash
# 1. 复制配置到全局 ~/.claude/ 目录
cp claude/CLAUDE.md ~/.claude/
cp -r claude/commands ~/.claude/

# 2. 验证安装
ls -lh ~/.claude/CLAUDE.md ~/.claude/commands/*.md

# 3. 重启 Claude Code
# 协议会自动加载，你将看到：
# [MODE: RESEARCH]
# 你已进入 RPE 协作模式，9 个 MCP 工具已就绪。
```

### 方案 B：项目级配置

适用于特定项目需要自定义协议。

```bash
# 在项目根目录创建 .claude/ 目录
mkdir -p /path/to/your/project/.claude
cp claude/CLAUDE.md /path/to/your/project/.claude/
cp -r claude/commands /path/to/your/project/.claude/

# Claude Code 会优先使用项目级配置
```

### 方案 C：Codex 配置

如果使用 Codex（单文件协议）：

```bash
# 复制 AGENTS.md 到 ~/.codex/
cp codex/AGENTS.md ~/.codex/

# 同时复制诊断 XML 模板
cp claude/commands/diagnose-*.xml ~/.codex/
```

### MCP 工具配置

本仓库提供 MCP 配置模板文件，按照以下步骤配置：

1. **复制并修改 MCP 配置模板**：
   ```bash
   # Claude Code 配置
   cp mcp.json.template ~/.claude/mcp.json

   # Serena/Codex 配置
   cp config.toml.template ~/.codex/config.toml
   ```

2. **修改配置文件中的占位符**：
   - `YOUR_EXA_API_KEY_HERE` → 替换为实际的 Exa API Key
   - `/path/to/mcp-debugger/...` → 替换为实际的 mcp-debugger 安装路径
   - 其他本地路径根据实际情况调整

3. **配置的 MCP 工具**（9 个）：
   - `mcp-debugger`（运行时调试，高优先级）
   - `serena`（LSP 符号分析）
   - `ripgrep`（文本搜索）
   - `basic-memory`（知识图谱）
   - `tree-sitter`、`semgrep`、`repomapper`、`exa`、`ctx`

4. **验证 MCP 工具**：
   ```bash
   # 启动 Claude Code 后输入：
   /mcp
   # 应显示已连接的 MCP 工具列表
   ```

---

## 🧭 核心架构

### 提案式切换（Proposal-based Mode Switching）

**解决的问题**：避免 AI 越权切换模式，同时保持流程流畅。

**工作方式**：
```
[MODE: EXECUTE]

测试完成后发现 3 个用例失败。

→ 提案：测试失败（3 个用例未通过），建议切换到 [DIAGNOSE]（输入"OK"继续，或"NO"留在当前模式）
```

用户只需输入 `OK` 或 `NO`，无需重复说明原因。

---

### 四大模式（Four Modes）

| 模式 | 目的 | 典型工具组合 | 产物 |
|------|------|--------------|------|
| **RESEARCH** | 理解需求、识别风险 | serena + ripgrep + repomapper | Basic Memory (designs/) |
| **PLAN** | 设计方案、拆分任务 | serena + tree-sitter + semgrep | Basic Memory (designs/决策/) |
| **EXECUTE** | 实现代码、运行测试 | serena + ripgrep + semgrep | 最小 diff + 测试通过的 commit |
| **DIAGNOSE** | 问题诊断、根因分析 | mcp-debugger + serena + ripgrep | XML 报告 + Basic Memory (diagnosis/) |

**模式切换规则**：
- 默认模式：`RESEARCH`
- 切换方式：提案式（AI 提案 → 用户确认）
- 未确认不得切换

---

### 三层记忆体系（Three-Layer Memory）

详细说明见 `claude/CLAUDE.md § 记忆策略`

**概要**：

| 层级 | 用途 | 位置 |
|------|------|------|
| Claude Code Todos | 当前会话任务追踪 | 内置 |
| Basic Memory | 跨会话开发日志 | `~/basic-memory/<project>/` |
| Serena | 自动化技术记忆 | 自动管理 |

---

## 🛠️ MCP 工具栈（9 个）

### 调试层（Debug Layer）
| 工具 | 用途 | 优先级 |
|------|------|--------|
| `mcp-debugger` | 运行时断点调试、单步执行、变量检查 | 🔴 高 |

### 记忆层（Memory Layer）
| 工具 | 用途 | 优先级 |
|------|------|--------|
| `serena` | LSP 符号级代码分析，多语言支持 | 🔴 高 |
| `basic-memory` | 本地 Markdown 知识图谱（Obsidian 集成）| 🟡 中 |

### 代码分析层（Code Analysis Layer）
| 工具 | 用途 | 优先级 |
|------|------|--------|
| `ripgrep` | 高性能文本搜索，尊重 gitignore | 🔴 高 |
| `tree-sitter` | 精确语法解析与增量分析 | 🟡 中 |
| `semgrep` | 安全漏洞扫描与代码质量检测 | 🟡 中 |
| `repomapper` | PageRank 算法识别项目关键文件 | 🟡 中 |

### 知识检索层（Knowledge Layer）
| 工具 | 用途 | 优先级 |
|------|------|--------|
| `exa` | 双模式搜索（web + 10 亿+ GitHub/Stack Overflow）| 🟡 中 |
| `ctx` | 项目上下文打包工具 | 🟢 低 |

**工具选择策略**：
- **RESEARCH**：serena（语义理解）+ ripgrep（全文搜索）+ repomapper（关键文件）
- **PLAN**：serena（影响范围）+ tree-sitter（复杂度）+ semgrep（风险）
- **EXECUTE**：serena（LSP 编辑）+ ripgrep（调用点）+ semgrep（安全）
- **DIAGNOSE**：mcp-debugger（优先）+ 其他所有工具（按 focus/quick/full 选择）

---

## 📋 诊断体系（Diagnosis System）

### 三级诊断模式

| 模式 | Token 范围 | 适用场景 | 数据收集 |
|------|-----------|---------|----------|
| **Focus** | 1-2K | 明确错误、快速定位 | 错误日志 + ±20 行代码 + git diff |
| **Quick** | 3-5K | 常规问题诊断（默认）| Focus + debugger + 项目结构 + 安全扫描 |
| **Full** | 8-12K | 复杂问题、深度分析 | Quick + AST 分析 + 外部知识 + 依赖分析 |

### 诊断流程

```
问题报告 → 自动选择模式 → 读取 XML 模板 → 主动添加 Debug 日志
→ 并行调用 MCP 工具 → 生成 XML 报告 → 保存到 diagnosis/
→ 提取 <lesson> 写入 Serena（如需）
```

**Debug Logging 原则**（关键）：
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
```

---

## 🧠 记忆策略（Memory Strategy）

### 三层记忆体系

**层级 1：Claude Code Todos**
- 用途：当前会话任务列表
- 生命周期：会话内有效

**层级 2：Basic Memory**
- 用途：跨会话开发日志（decisions/designs/diagnosis）
- 位置：`~/basic-memory/<project>/`

**层级 3：Serena**
- 用途：自动化技术记忆
- 命名：`pitfalls_*` / `constraints_*` / `architecture_*`

**使用原则**：
- Claude Code Todos：回答"现在做什么"
- Basic Memory：回答"上次做到哪了""为什么这样设计"
- Serena：回答"这个模块有什么坑"

---

## ✅ 质量门槛与安全边界

详见 `claude/CLAUDE.md § 质量门槛` 和 `§ 安全与边界`

---

## 🎓 使用示例

### 场景 1：理解新模块

```
[MODE: RESEARCH]

我需要理解 payment-gateway 模块的实现。

Claude 会自动：
1. 使用 Todos 追踪研究任务
2. 使用 repomapper 识别关键文件
3. 使用 serena 读取符号概览
4. 使用 ripgrep 搜索关键接口
5. 将研究成果写入 Basic Memory:
   ~/basic-memory/payment-system/designs/2025-10-12-payment-gateway-research.md
```

### 场景 2：修复 Bug

```
[MODE: EXECUTE]

修改完成后运行测试...

→ 提案：测试失败（calculateTotal 返回负值），建议切换到 [DIAGNOSE]（输入"OK"继续）

用户：OK

[MODE: DIAGNOSE]

选择 Quick 模式，使用 mcp-debugger 单步执行...

发现问题：discount 未做非负校验

→ 提案：问题已定位，建议切换到 [EXECUTE]（输入"OK"继续）

用户：OK

[MODE: EXECUTE]

添加非负校验，重新运行测试，通过 ✅
```

### 场景 3：重要技术选型

```
[MODE: PLAN]

需要选择缓存方案...

Claude 生成方案对比（Redis vs Memcached）

用户确认选择 Redis

创建 Basic Memory 决策记录：
~/basic-memory/payment-system/decisions/2025-10-15-redis-cache-selection.md

填写：
- 背景: 需要支持集群、持久化
- 决策: 使用 Redis
- 放弃的方案: Memcached（不支持持久化）
- 后果: 需要维护 Redis 集群

同时在 Serena 中创建 constraints_caching 记忆
```

---

## 🎯 优化成果（2025-10-18）

### Token 使用优化

**CLAUDE.md + commands/ 精简**：
- 原始行数：730+ 行 ≈ 7300+ tokens
- 精简后：923 行（含新增 wrap.md）
- **精简策略**：删除冗余示例、合并重复内容、优化结构层次

**AGENTS.md 精简**：
- 原始行数：460 行 ≈ 4600 tokens
- 精简后：292 行 ≈ 2920 tokens
- **节省 36% Token 使用**

**精简原则**：
- ❌ 删除重复的"核心规则"章节（底部与顶部重复）
- ❌ 压缩各模式的详细步骤（从分步 bash 示例改为流程描述）
- ❌ 移除冗余的模式切换决策表格
- ✅ 保留核心流程、关键工具、实操示例（Git commit、Debug logging）
- ✅ 新增 wrap.md 会话结束清单

### 一致性优化

**路径引用统一**：
- Basic Memory：`~/basic-memory/<project>/designs/`
- 相对路径：`commands/diagnose.md`（从 CLAUDE.md）
- 向上引用：`../CLAUDE.md`（从 commands/*.md）
- XML 模板：`./diagnose-{mode}.xml`（相对于 commands/）

**记忆体系统一**：
- 所有文档使用相同的三层记忆表格
- 统一的命名规范（Serena / Basic Memory / Claude Code Todos）
- 统一的提案式切换机制

### 新增功能

1. **主动提示规则**（所有模式）
   - 阶段完成后自动提示下一步行动
   - 提供具体的文件路径和命令示例

2. **提案式切换机制**
   - AI 提案 → 用户确认（OK/NO）
   - 避免越权切换，保持流程可控

3. **settings.json 增强**（已应用到 `~/.claude/`）
   - SessionStart Hook：模式检查 + /begin 引导
   - PostToolUse Hook：Edit 后记忆提示 + Bash 后 Debug 提示
   - PreToolUse Hook：修改前 checklist（新增）
   - Stop Hook：收工前清单 + Git 提醒

---

## 🔄 版本管理流程

完成重大功能后的提交流程：

```bash
# 1. 推送分支
git push origin feature-branch

# 2. 创建 PR
gh pr create --title "feat: 新增支付风控" --body "..."

# 3. 确保 CI 通过
gh pr checks

# 4. 合并
gh pr merge --squash

# 5. 更新文档
vim CHANGELOG.md

# 6. 创建版本标签
git tag -a v1.2.0 -m "feat: 新增支付风控"
git push origin v1.2.0
```

---

## 🤝 贡献指南

这是个人配置仓库，但欢迎：
- 🐛 报告 Bug
- 💡 提出改进建议
- 📖 分享使用经验

---

## 📄 许可证

Private - Personal Use Only

---

## 📚 相关资源

- [Claude Code 官方文档](https://docs.claude.com/en/docs/claude-code)
- [MCP 协议规范](https://modelcontextprotocol.io/)
- [ADR 标准格式](https://adr.github.io/madr/)
- [Serena LSP 工具](https://github.com/cognibuild/serena)

---

---

## 📊 配置文件对比

| 配置类型 | 位置 | 文件大小 | 行数 | 特点 |
|---------|------|---------|------|------|
| **Claude Code** | `~/.claude/` | CLAUDE.md (16KB)<br>+ commands/*.md | 923 行（含 wrap.md） | 分离式结构，支持斜杠命令 |
| **Codex** | `~/.codex/` | AGENTS.md (12KB) | 292 行 | 单文件协议，精简 36% |

---

## 🔍 故障排查

### 问题 1：斜杠命令不工作

```bash
# 检查配置文件是否存在
ls -la ~/.claude/CLAUDE.md ~/.claude/commands/

# 检查文件权限
chmod 644 ~/.claude/CLAUDE.md
chmod 644 ~/.claude/commands/*.md

# 重启 Claude Code
```

### 问题 2：MCP 工具未连接

```bash
# 启动 Claude Code 后输入
/mcp

# 如果工具未显示，检查 mcp.json 配置
cat ~/.claude/mcp.json

# 参考官方文档配置 MCP 工具：
# https://docs.claude.com/en/docs/claude-code/mcp
```

### 问题 3：XML 模板找不到

```bash
# DIAGNOSE 模式需要 XML 模板
ls ~/.claude/commands/diagnose-*.xml

# 如果缺失，重新复制
cp claude/commands/diagnose-*.xml ~/.claude/commands/
```

---

**最后更新**: 2025-10-18
- ✅ **大幅精简文档**：AGENTS.md 减少 36%，CLAUDE.md + commands/ 优化结构
- ✅ 新增 wrap.md 会话结束清单
- ✅ 移除 AGENTS.md 底部重复的"核心规则"章节
- ✅ 压缩各模式详细步骤，改为流程描述 + 关键工具列表
- ✅ 保留高价值实操示例（Git commit 格式、Debug logging 策略）
- ✅ Codex 配置新增 XML 诊断模板支持
