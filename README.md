# claude-channels-patch (v2.1.179+ fork)

[English version: README.en.md](./README.en.md)

> **本项目由 Claude Code (DeepSeek V4 Pro 后端) 协助完成**，在 [wuzf/claude-channels-patch](https://github.com/wuzf/claude-channels-patch) 的基础上，由 AI 辅助修改适配 Claude Code v2.1.179+。
>
> **免责声明**：本项目仅供学习和研究使用。请勿将其用于你不拥有或未获授权分析的软件或服务。

这是一个针对 **Claude Code** 的二进制补丁，可在无需 `claude.ai` OAuth 认证的情况下启用 `--channels` 功能。

本仓库是 [wuzf/claude-channels-patch](https://github.com/wuzf/claude-channels-patch) 的改进版本，主要适配 **Claude Code v2.1.179+**（原版仅测试到 v2.1.83）。

## 与原版的区别

| 项目 | 原版 | 本版 |
|------|------|------|
| 测试版本 | v2.1.81 ~ v2.1.83 | **v2.1.179** |
| 模式匹配阈值 | >=2 份副本（主线程 + Worker） | **>=1 份副本**（新版本合并了代码路径） |
| Auth/Allowlist 旁路 | 必须找到才能继续 | **降级为可选**（找不到跳过，不影响主流程） |
| NoAuth UI 旁路 | 必须找到 | **降级为可选** |

## 为什么需要这个补丁？

Claude Code 的 `--channels` 功能被三层检查限制：

| 限制项 | 检查内容 | 影响 |
|------|------|------|
| **功能开关** | GrowthBook `tengu_harbor` / `tengu_harbor_permissions` | 默认 `false`，API key 用户和第三方中转用户无法启用 |
| **OAuth** | `accessToken` 检查 | 使用 DeepSeek 等第三方后端的用户没有 claude.ai token |
| **Allowlist** | 插件/服务器白名单 | 非官方插件（如微信 weixin）不在白名单中 |

本补丁通过修改二进制中的字节码和特征标志，绕过这些限制。

## 系统要求

- **Python 3.10+**
- Claude Code v2.1.80+

## 兼容性

- ✅ **Claude Code v2.1.179**（本版测试通过）
- ✅ Windows / Linux / macOS
- ✅ macOS 自动 ad-hoc codesign
- ✅ 第三方 API 中转用户（DeepSeek 后端验证通过）
- ✅ 自动检测 Native Install / Homebrew / WinGet 安装

## 用法

```bash
# 克隆仓库
git clone git@github.com:842655waxdss/claude-channels-patch.git
cd claude-channels-patch

# 应用补丁（自动检测所有已安装的二进制）
python patch.py

# 只检查，不修改文件
python patch.py --check

# 指定二进制路径
python patch.py --binary /path/to/claude

# 恢复原始状态
python patch.py revert
```

## 工作原理

### 核心策略：Bun Source Fallback

Claude Code 是 Node.js SEA（Single Executable Application），内部嵌有 JavaScript 源码和编译后的 bytecode。补丁的核心是将 `@bytecode` 标记改为 `@source__`，强制运行时使用修改过的源码而非原始 bytecode。这是所有旁路的基础。

### 功能开关默认值修改

```javascript
// 修改前
l$("tengu_harbor", !1)          // default = false
l$("tengu_harbor_permissions", !1)  // default = false

// 修改后
l$("tengu_harbor", !0)          // default = true
l$("tengu_harbor_permissions", !0)  // default = true
```

### Allowlist 检查旁路（服务端）

```javascript
// 修改前
if(!D.dev) return {action:"skip", kind:"allowlist", reason:"server..."}

// 修改后
if( D.dev) return {...}  // D.dev 为 undefined，条件永远为假
```

### 降级处理（v2.1.179 适配）

新版本中 Auth 和 NoAuth 相关的代码路径已被重构，原版锚点模式不再匹配。这些旁路在新版本中降级为**可选**——如果能找到模式就应用，找不到就跳过。实际测试表明，Bun Source Fallback + 功能开关修改已经足够启用 Channels。

## 安全性

- **备份**：修改前自动保存 `*.bak` 备份
- **原子写入**：使用临时文件 + `os.replace()`，不会损坏运行中的二进制
- **可恢复**：`python patch.py revert` 一键回滚
- **无硬编码偏移**：所有修改通过模式匹配动态定位

## 许可证

MIT

## 致谢

- 原作者：[wuzf/claude-channels-patch](https://github.com/wuzf/claude-channels-patch)
- 哈雷彗星 [Claude Code Channels 工人能智版](https://linux.do/t/topic/1787422)
