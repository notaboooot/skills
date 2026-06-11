# 改动报告：百度 iCode 仓库支持

## 概述

本次改动为 skills CLI 工具添加了对百度 iCode 仓库 URL 的支持，并修复了非通用代理的安装目录问题。

## 改动文件

### 1. `src/source-parser.ts`

**新增功能**：百度 iCode URL 解析

- 支持解析以下 URL 格式：
  - `https://console.cloud.baidu-int.com/devops/icode/repos/owner/group/repo/tree/branch/path`
  - `https://console.cloud.baidu-int.com/devops/icode/repos/owner/group/repo/tree/branch`
  - `https://console.cloud.baidu-int.com/devops/icode/repos/owner/group/repo`

- URL 转换逻辑：
  - 将 HTTPS URL 转换为 SSH 格式：`ssh://icode.baidu.com:8235/owner/group/repo`
  - 自动提取分支名称（ref）和子路径（subpath）

- 修改 `isWellKnownUrl()` 函数，排除 `console.cloud.baidu-int.com` 域名

### 2. `src/source-parser.test.ts`

**新增测试用例**：

- 测试带分支和路径的 URL 解析
- 测试仅带分支的 URL 解析
- 测试不带分支的基础 URL 解析
- 测试简单的 owner/repo 格式

### 3. `src/installer.ts`

**修复问题**：非通用代理安装目录错误

- **问题**：之前所有代理都安装到 `.agents/skills/` 目录，导致 Claude Code 无法识别
- **修复**：
  - 非通用代理（如 claude-code）直接安装到代理特定目录：`.claude/skills/`
  - 通用代理（如 cline, codex 等）继续使用 `.agents/skills/` 目录

**修改的函数**：

- `installSkillForAgent()` - 简化安装逻辑，移除符号链接机制
- `installRemoteSkillForAgent()` - 直接写入代理目录
- `installWellKnownSkillForAgent()` - 直接写入代理目录
- `installBlobSkillForAgent()` - 直接写入代理目录

### 4. `src/agents.ts`

**新增函数**：

```typescript
export function getCanonicalSkillsDirForAgent(agentType: AgentType, global: boolean, cwd?: string): string
```

- 为非通用代理获取正确的安装目录

## 使用示例

```bash
# 安装百度 iCode 仓库的技能
skills add "https://console.cloud.baidu-int.com/devops/icode/repos/baidu/personal-code/dinghchenguang-skills/tree/master"

# 指定分支和路径
skills add "https://console.cloud.baidu-int.com/devops/icode/repos/baidu/personal-code/dinghchenguang-skills/tree/master/skills/demo"

# 列出仓库中的技能
skills add "https://console.cloud.baidu-int.com/devops/icode/repos/baidu/personal-code/dinghchenguang-skills/tree/master" --list
```

## 行为变更

| 代理类型 | 之前 | 现在 |
|---------|------|------|
| claude-code (项目级) | `.agents/skills/` | `.claude/skills/` |
| claude-code (全局级) | `~/.agents/skills/` | `~/.claude/skills/` |
| cline, codex 等 (项目级) | `.agents/skills/` | `.agents/skills/` |
| cline, codex 等 (全局级) | `~/.agents/skills/` | `~/.agents/skills/` |

## 注意事项

- 百度 iCode 仓库需要通过 SSH 访问，请确保：
  - 已配置 `~/.ssh/config` 包含 `icode.baidu.com` 的 SSH 设置
  - 已连接百度内网 VPN
  - SSH 密钥已添加到 iCode 账户
