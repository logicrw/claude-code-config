# Claude Code Configuration Backup

这是我的 Claude Code 和 Codex 配置备份仓库,包含完整的 RPE 人机协作协议和诊断命令系统。

## 目录结构

```
.
├── claude/
│   ├── CLAUDE.md              # Claude Code 的 RPE 协作协议
│   └── commands/              # 诊断命令与 XML 模板
│       ├── begin.md           # 启动指令
│       ├── diagnose.md        # DIAGNOSE 模式文档
│       ├── diagnose-focus.xml # Focus 模式诊断模板 (1-2K tokens)
│       ├── diagnose-quick.xml # Quick 模式诊断模板 (3-5K tokens)
│       ├── diagnose-full.xml  # Full 模式诊断模板 (8-12K tokens)
│       ├── execute.md         # EXECUTE 模式文档
│       └── ...                # 其他命令文件
├── codex/
│   └── AGENTS.md              # Codex 的 RPE 协作协议
├── mcp.json.template          # Claude Code MCP 配置模板
├── config.toml.template       # Codex 配置模板
└── README.md                  # 本文档

```

## MCP 工具栈 (9 个)

| MCP Server | 版本 | 功能 | 优先级 |
|-----------|------|------|--------|
| mcp-debugger | 0.15.4 | 运行时断点调试、单步执行、变量检查 | 🔴 高 |
| serena | 0.3.0 | LSP 符号级代码分析,多语言支持 | 🔴 高 |
| ripgrep | 13.0.0 | 高性能文本搜索,尊重 gitignore | 🔴 高 |
| tree-sitter | 0.20.8 | 精确语法解析与增量分析 | 🟡 中 |
| semgrep | 1.45.0 | 安全漏洞扫描与代码质量检测 (beta) | 🟡 中 |
| repomapper | 0.2.1 | PageRank 算法识别项目关键文件 | 🟡 中 |
| exa | 1.0.0 | 双模式搜索 (web + 10 亿+ 代码库) | 🟡 中 |
| basic-memory | latest | 本地 Markdown 知识图谱 | 🟢 低 |
| ctx | latest | 项目上下文打包工具 | 🟢 低 |

## 安装指南

### 1. 安装配置文件

```bash
# Claude Code (macOS/Linux)
cp mcp.json.template ~/.claude/mcp.json
cp -r claude/commands ~/.claude/
cp claude/CLAUDE.md ~/.claude/

# Codex
cp config.toml.template ~/.codex/config.toml
cp codex/AGENTS.md ~/.codex/
```

### 2. 配置敏感信息

在 `~/.claude/mcp.json` 和 `~/.codex/config.toml` 中替换:
```
YOUR_EXA_API_KEY_HERE  →  你的真实 Exa API Key
```

### 3. 安装 mcp-debugger

```bash
# 克隆并构建 mcp-debugger
git clone https://github.com/modelcontextprotocol/servers.git ~/mcp-debugger-source
cd ~/mcp-debugger-source/src/mcp-debugger
npm install
npm run build

# 复制到指定位置
mkdir -p ~/mcp-debugger
cp -r dist ~/mcp-debugger/
```

### 4. 验证安装

```bash
# 检查 Claude Code 配置
cat ~/.claude/mcp.json | jq '.mcpServers | keys'

# 检查 Codex 配置
grep -A 3 "\[mcp_servers\]" ~/.codex/config.toml
```

## RPE 协作协议

### 四大模式

1. **[MODE: RESEARCH]** - 理解需求、识别风险、澄清问题
2. **[MODE: PLAN]** - 形成技术方案、拆分任务、制定测试计划
3. **[MODE: EXECUTE]** - 实现代码、运行测试、更新文档
4. **[MODE: DIAGNOSE]** - 问题诊断、根因分析、生成 XML 报告

### 诊断命令三级体系

| 模式 | Token 范围 | 适用场景 | 模板文件 |
|-----|-----------|---------|---------|
| Focus | 1-2K | 单点问题快速定位 | diagnose-focus.xml |
| Quick | 3-5K | 常规问题诊断 (默认) | diagnose-quick.xml |
| Full | 8-12K | 复杂问题深度分析 | diagnose-full.xml |

### 工具使用策略

**RESEARCH 模式**:
- serena: 语义理解符号关系
- ripgrep: 全文搜索关键词
- repomapper: 识别关键文件
- exa: 搜索代码示例

**PLAN 模式**:
- serena: 分析影响范围
- tree-sitter: 计算复杂度
- semgrep: 预检测风险
- repomapper: 识别高耦合

**EXECUTE 模式**:
- serena: LSP 精确编辑
- ripgrep: 搜索调用点
- semgrep: 安全扫描
- exa: API 最佳实践

**DIAGNOSE 模式**:
- mcp-debugger: 运行时调试 (优先)
- serena: 定位问题符号
- ripgrep: 搜索错误模式
- tree-sitter: AST 分析
- semgrep: 漏洞扫描

## 记忆策略

- **Serena**: LSP 自动记忆,符号级技术知识
  - 命名: `pitfalls_<module>` / `constraints_<topic>` / `architecture_<component>`

- **Basic Memory**: 手动开发日志,叙事性进展记录
  - 组织: 按日期或功能模块分类

## 版本管理流程

完成重大功能后的提交流程:

**执行步骤**:
1. 推送分支到远程
2. 创建/更新 PR
3. 确保 CI 通过并完成代码审查
4. 使用保护规则合并
5. 同步默认分支
6. 更新文档与 `CHANGELOG.md`
7. 确定 SemVer 版本号并创建标注标签
8. 生成/发布版本说明
9. 发布后验证与回滚准备
10. 清理已合并分支

## 质量门槛

**执行前检查清单**:
- 范围已最小化
- 敏感信息已脱敏
- 可用性测试（如适用）
- 测试已定义并本地通过
- 回滚方案已就绪（如有状态变更）

**执行后检查清单**:
- 所有测试通过
- 文档已更新
- 可观测性验证（日志/指标）
- 风险项已复核
- 用户已确认

## 安全边界

- 任何副作用操作需先确认
- 破坏性命令必须有回滚方案
- 敏感信息脱敏处理
- 禁止输出 `.env`、密钥、Token

## 许可证

Private - Personal Use Only
