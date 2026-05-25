---
title: npm link 无法执行 issue
date: 2026-05-20
status: published
published_at: 2026-05-25T23:36:02+08:00
updated_at: 2026-05-25T23:36:02+08:00
---

# npm link 创建 Symlink 导致 CLI 文件无执行权限的问题

## 现象

使用 `npm link` 注册本地 CLI 工具后，运行命令报 `Permission denied`（exit code 126），即使 `package.json` 的 `bin` 字段和 shebang 都配置正确。

## 根因

`npm link` 在全局 bin 目录下创建的是 **symlink**，直接指向 `dist/server/cli.js`，而非 npm 的 shim 脚本。symlink 执行时，操作系统要求目标文件本身具有可执行权限（`+x`）。

而 `tsc` 编译时总是创建新文件，文件权限由 umask 决定。默认 umask `0022` 意味着新建文件权限为 `644`（`rw-r--r--`），不可执行。因此每次构建后，`cli.js` 都会丢失执行权限。

## 两种链接方式对比

| | Symlink | Shim 脚本 |
|---|---|---|
| 内容 | `spec-lens -> ../lib/node_modules/spec-lens/dist/server/cli.js` | `#!/usr/bin/env node` + `require("spec-lens/dist/server/cli.js")` |
| 是否需要 `+x` | 是，操作系统直接执行目标文件 | 否，由 `node` 解释执行 |
| 何时生成 | `npm link` 在某些环境/版本下 | `npm install -g` 或旧版 `npm link` |

symlink 方式下，bash 解析到目标文件后检查其 `+x` 权限，如果没有则拒绝执行（exit 126），甚至不会读取 shebang。

## 解决方案

在 `package.json` 的 build 脚本末尾加 `chmod +x`：

```json
{
  "scripts": {
    "build": "tsc -b && vite build && tsc -p server/tsconfig.json && chmod +x dist/server/cli.js"
  }
}
```

**为什么不能在源文件上设权限？** 因为 `tsc` 编译时创建全新文件，不会保留源文件的权限位，只受 umask 控制。

## 验证方式

```bash
# 确认 link 类型
file $(which spec-lens)
# 输出: symbolic link to ... 说明是 symlink

# 确认权限
ls -la dist/server/cli.js
# 需要 -rwxr-xr-x，而非 -rw-r--r--

# 修复权限后验证
chmod +x dist/server/cli.js
spec-lens --help
```
