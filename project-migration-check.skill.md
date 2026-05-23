---
name: project-migration-check
description: Use when checking whether a Node/Vite/React/Vercel project still works after being moved, renamed, copied, restored, or migrated between directories or Windows machines.
---

# 项目迁移检查

## 原则

默认只检查，不修改代码、不删除文件、不安装依赖、不提交 Git、不覆盖配置。破坏性操作和修复动作必须先获得用户明确同意。

在 Windows PowerShell 中优先使用 `npm.cmd`，避免触发 `npm.ps1` 执行策略问题。输出密钥时只列环境变量键名，不显示值。

## 工作流

1. 先说明检查计划和检查强度。
2. 确认真实项目根目录；若当前目录无 `.git` 或 `package.json`，先定位项目，不急着判定失败。
3. 按强度执行检查，记录命令结果、失败证据和是否与迁移有关。
4. dev 服务只做短时启动验证，避免留下后台进程。
5. 最后只给状态、问题和建议，不自动修复。

## 检查强度

| 类型 | 适用场景 | 检查范围 |
| --- | --- | --- |
| 关键项目 | 生产、客户、即将部署 | Git、结构、依赖、dev、test、build、Vercel、环境变量键名、路径引用、编码、端口残留、工作区脏状态 |
| 普通项目 | 日常开发、迁移后继续开发 | Git、结构、依赖、dev、test、build、Vercel、路径引用 |
| 归档项目 | 暂不开发，只需可恢复 | Git、结构、`package.json`、锁文件、明显路径引用、README/配置可读性 |

未指定时按“普通项目”。

## 核心检查面

- Git：仓库是否可识别，分支/远程是否正常，是否有未提交改动、换行符风险。
- 项目结构：是否在真实项目根目录，关键目录和配置文件是否存在。
- Node 依赖：`package.json`、锁文件、`node_modules` 是否匹配；用 `npm.cmd ls --depth=0` 判断缺失或异常依赖。
- 运行脚本：优先检查 `dev`、`test`、`build`；Vitest 不要误用 Jest 参数如 `--runInBand`。
- TypeScript：必要时用无输出编译检查路径、类型和大小写问题。
- Vercel：检查 `vercel.json`、`.vercel/project.json`、SPA rewrite、API 路由、输出目录和环境变量键名；不做远程部署。
- 路径引用：搜索旧盘符、旧工作区名、绝对路径、Windows 反斜杠、大小写不一致导入。
- 编码：重点看 README、中文错误消息和替换字符；PowerShell 显示乱码不等于文件损坏，必要时按 UTF-8 验证。
- Windows 迁移风险：执行策略、端口占用、CRLF/LF、大小写敏感差异、本地缓存和 `.env.local` 差异。

## 常用命令策略

优先使用只读命令：`git status --short --branch`、`rg --files`、`Get-ChildItem`、`Get-Content`、`npm.cmd ls --depth=0`。  
验证运行能力时使用项目脚本：`npm.cmd run dev`、`npm.cmd test`、`npm.cmd run build`。  
如果 build 会改写产物且用户要求零写入，跳过 build 并说明，改用类型检查或结构检查。

## 判断级别

| 状态 | 含义 |
| --- | --- |
| 正常 | 核心命令通过，未发现迁移相关问题 |
| 有风险 | 可运行，但存在脏工作区、编码、环境变量、换行符或文档路径风险 |
| 需处理 | dev/test/build 失败、依赖异常、配置无效或硬编码旧路径 |
| 阻塞 | 无法定位项目根目录、无 `package.json`、Git/依赖基础不可用 |

## 报告模板

```markdown
**项目状态总结**
- 项目目录：
- 检查强度：
- 总体状态：正常 / 有风险 / 需处理 / 阻塞
- Git：
- 依赖：
- dev/test/build：
- Vercel：
- 路径与 Windows 迁移风险：
- 编码：

**存在的问题**
1. 问题：
   - 证据：
   - 影响：
   - 是否与迁移相关：

**建议下一步操作**
1. 先处理：
2. 再验证：
3. 可选：

**已执行 / 跳过的检查**
- 已执行：
- 跳过及原因：
```
