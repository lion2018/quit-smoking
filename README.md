# 戒烟 PWA

移动端优先的戒烟 Web/PWA：在“想抽烟”的那一刻，先给自己几分钟。

> 本产品提供行为支持，不构成医疗建议。药物和戒断处理请咨询医生或戒烟门诊。

## 当前状态

- 平台：移动端 Web / PWA（可添加到手机主屏幕）
- 阶段：MVP 已实现，等待真机测试与服务器部署
- 数据：本地 `localStorage`，无后端、无账号
- 部署目标：`/quit-smoking/` 子目录，不影响服务器根路径的其他项目

## 功能

- 首次使用：知情同意（本地数据 + 匿名研究可选项）
- 计划设置：戒烟日、基线日均支数、烟价、每包支数、戒烟基金目标
- 首页：戒烟第 X 天、少抽支数、省钱、延迟分钟、熬过波数、下一里程碑、今日洞察
- 渴求急救：10 秒分型 + 强度 → 3/10 分钟延迟 / 任务 / 小游戏 → 延迟结束称赞与上次对比
- 记录：一键记录一支、触发/情绪标签、“下一支不抽”提示
- 进展：累计延迟、最长/平均、7 日柱状图、28 日无烟日历、触发画像
- 设置：称赞语气、上次对比开关、匿名研究开关、导出 JSON、清除数据

## 目录

```
site/quit-smoking/     可运行的静态应用（单文件 + PWA 资源）
design/                UI 设计稿与设计说明
docs/                  开发文档、开发日志
deploy/                部署包、Nginx 配置、部署说明
```

## 本地预览

直接用 Edge 打开：

```powershell
& "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" "site\quit-smoking\index.html"
```

或双击 `site/quit-smoking/index.html`。

## 部署

1. 上传 `deploy/quit-smoking.zip` 到服务器
2. 解压到 `/var/www/quit-smoking/`
3. 按 `deploy/quit-smoking.nginx.conf` 增加子目录 location
4. `nginx -t && systemctl reload nginx`
5. 访问 `http://<服务器IP>/quit-smoking/`

完整 PWA（离线/安装）需要 HTTPS；纯 HTTP 下按普通网页使用。

## 文档

- [开发文档](docs/开发文档.md)
- [开发日志](docs/开发日志.md)
- [UI 设计稿](design/ui-mockup.html)
- [部署说明](deploy/README.md)

## 数据与研究

当前版本只做本地统计。匿名研究开关只保存偏好，不会上传数据。群体效果统计需要后端、同意管理、分析管线和管理后台，属于后续阶段。
