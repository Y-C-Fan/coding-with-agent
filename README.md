# coding-with-agent

Sharing my hands-on experience and pro tips for working with Claude Code and other AI coding assistants.

## 核心内容

### Claude Code Skills
本仓库包含多个实用的 Claude Code Skills，可直接复制到 `~/.claude/skills/` 目录下使用：

| Skill | 描述 |
|-------|------|
| [code-review](./.claude/skills/code-review/) | 代码审查 skill，包含安全、性能、可维护性检查框架 |
| [research-workflow](./.claude/skills/research-workflow/) | 科研项目工作流，适配基于现有项目的二次开发场景 |
| [project-explorer](./.claude/skills/project-explorer/) | 项目探索工具，快速理解项目结构和架构 |

### 工作流文档
- [workflow.md](./workflow.md) - 我的 coding agent 辅助科研项目工作流
- [core-taste.md](./core-taste.md) - 核心观点：context、需求拆解、测试、规划
- [CLAUDE.md](./CLAUDE.md) - AI 协作指南配置示例

## Skills 使用方法

将 skills 复制到 Claude Code 的 skills 目录：

```bash
# 方式一：复制整个 .claude 文件夹到你的用户目录
cp -r .claude ~/

# 方式二：只复制特定的 skill
mkdir -p ~/.claude/skills/
cp -r .claude/skills/code-review ~/.claude/skills/
```

## 核心观点

1. **Context 是关键** - 提供充分且精准的上下文，不要吝啬 token
2. **需求要拆解** - 不要一次性让 agent 做太复杂的任务
3. **自己测试** - agent 可能会欺骗你，尽可能自己验证
4. **先 plan 再动手** - 方向错了再重构很麻烦
