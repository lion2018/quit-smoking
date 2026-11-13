# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

MVP implemented as **mobile Web / PWA** (as of 2026-11-13). Source in `site/quit-smoking/`; docs in `docs/`; UI mockup in `design/`; deploy package in `deploy/`. Git repo initialized and pushed to GitHub (`main`). Before new features, update docs and get user approval; after changes, update `docs/开发文档.md` and `docs/开发日志.md`.

## 文档约定（必须遵守）

- 每次对话中产生有价值的信息（研究结论、需求、设计决策、技术选型、风险、待办），立即写入 `docs/开发文档.md`，不要只留在聊天里。
- 每次对仓库做任何更改（新增/修改/删除文件、调整方案），在 `docs/开发日志.md` 末尾追加一条记录，格式：日期 / 类型 / 内容 / 原因 / 影响文件 / 下一步。
- 文档用中文；结论区分「已有证据」与「设计推断」；医疗相关内容标注不构成医疗建议。
- 更新文档后同步更新文档顶部的版本号与日期。

## Project

戒烟 Web/PWA（移动端优先，可添加到手机主屏幕；2026-11-13 从微信小程序方案切换）。核心不是打卡，而是「触发场景识别 + 3–5 分钟渴求急救 + 医疗导流」。渴求急救 2.1 采用「事前 if-then + 事中分型/urge surfing/延迟阶梯（任务或小游戏）+ 事后学习」闭环；延迟结束记录时长并给具体、过程性称赞，默认语气为**鼓励型 + 积极预期**，**与上次对比默认显示**（更多庆祝，更少给上下文）。始终保留“我现在要抽”并一键记录，不做监控型产品。首页首屏固定 4 个数字：戒烟天数、少抽支数、省钱、延迟分钟/熬过波数；避免精确“彻底戒掉日期”。详见 `docs/开发文档.md` 第 16–20 节；平台方案见第 15 节，改动记录见 `docs/开发日志.md`。

## Working here

- 当前无构建/测试命令；脚手架落地后在此补充经过验证的命令。
- 不要编造未经验证的命令或架构。
- 截至 2026-11-12 下午：VS Code 已不在安装目录/注册表中（仅剩空目录），Geany/MSYS2 安装已取消并清理，**当前无专用编辑器**。需要安装编辑器时先与用户确认，不要自行安装。
- 低内存编辑器选型参见 `docs/开发文档.md` 第 14.3 节（当前暂缓）。
- 项目已改为 Web/PWA：零构建、单文件 HTML + 系统 Edge 即可开发，不依赖 Node；发布与 PWA 方案见第 15 节。
- UI 设计稿：`design/ui-mockup.html`（Edge 直接打开）；设计说明：`design/README.md`。开发前先对照设计稿，不要把设计板当业务代码直接复制。
- 数据研究：群体效果统计需要事件采集 + 同意 + 后端 + 分析 + 管理后台；MVP 先做本地事件日志。观察数据不能直接宣称因果，见开发文档第 22 节。
- 部署：目标子目录 `/quit-smoking/`，所有资源用相对路径，不碰根 `location /`；配置见 `deploy/README.md` 与 `deploy/quit-smoking.nginx.conf`。Service Worker 需要 HTTPS。
- MVP 代码：`site/quit-smoking/index.html`（单文件，原生 JS + localStorage）；PWA 文件在同目录；部署包 `deploy/quit-smoking.zip`。修改后重新压缩 zip。
- GitHub：`https://github.com/lion2018/quit-smoking.git`，分支 `main`，远端 `git@github.com:lion2018/quit-smoking.git`。本机 Git 在 `C:\Users\Administrator\Tools\MinGit\cmd\git.exe`。
- 推送使用 SSH key（`~/.ssh/id_ed25519_github`，config 走 `ssh.github.com:443`）；公钥已添加到 GitHub，`main` 已推送成功。推送命令：
  `$env:GIT_SSH_COMMAND='"C:\Users\Administrator\Tools\MinGit\usr\bin\ssh.exe" -F "C:\Users\Administrator\.ssh\config"'; & "C:\Users\Administrator\Tools\MinGit\cmd\git.exe" push`
- 绝不提交私钥、token 或服务器密码；GitHub 不再接受账号密码做 Git 操作，HTTPS 需用 PAT。
- 本机无法直连 Microsoft 下载域名与 GitHub；安装工具/依赖时优先使用 Gitee、清华 TUNA、华为云镜像、npmmirror，并校验哈希。详见 `docs/开发文档.md` 第 14 章。
