---
title: playwright 下载浏览器 issue
date: 2026-05-19
status: published
published_at: 2026-05-25T23:36:02+08:00
updated_at: 2026-05-25T23:36:02+08:00
---

# Playwright 浏览器下载问题记录

## 问题

全局安装 Playwright npm 包很顺利，但 `npx playwright install chromium` 下载浏览器时卡住或 404。

## 原因

- Playwright npm 包（~15MB JS 代码）和浏览器二进制（~150-200MB）是分开托管的
- 新版 Playwright (1.52+) 使用 Chrome for Testing CDN (`builds/cft/` 路径)，npmmirror 镜像未同步此路径，返回 404
- Google 原始 CDN (`storage.googleapis.com`) 国内访问极慢或无法连接
- 旧版 Playwright (1.48 及更早) 使用 `builds/chromium/` 路径，npmmirror 镜像有同步

## 解决方案

降级 Playwright 到 1.48.0，配合国内镜像安装浏览器：

```bash
# 1. 安装 1.48.0
npm install -g playwright@1.48.0

# 2. 用 npmmirror 镜像下载浏览器
PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright npx playwright install chromium
```

结果：Chromium 130.0.6723.31 (playwright build v1140) 安装成功，浏览器位于 `~/.cache/ms-playwright/chromium-1140/`

## 备选方案

如果不想降级 Playwright，可以手动下载浏览器二进制：

```bash
# 1. 用浏览器（挂代理）下载对应版本的 Chrome for Testing
# 下载地址格式: https://storage.googleapis.com/chrome-for-testing-public/{版本号}/linux64/chrome-linux64.zip

# 2. 解压到 Playwright 缓存目录
mkdir -p ~/.cache/ms-playwright/chromium-{build号}
unzip chrome-linux64.zip -d ~/.cache/ms-playwright/chromium-{build号}/
```

## 环境变量备忘

| 变量 | 用途 |
|------|------|
| `PLAYWRIGHT_DOWNLOAD_HOST` | 指定浏览器下载镜像源 |
| `PLAYWRIGHT_BROWSERS_PATH` | 指定浏览器安装目录（默认 `~/.cache/ms-playwright/`） |

## 日期

2026-05-19
