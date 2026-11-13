# 戒烟 PWA · 部署说明

> 目标：`http://<服务器IP>/quit-smoking/`
> 服务器根路径 `/` 已有另一个项目，本方案只新增子目录，不改根 location。

## 当前状态

- 目前只有 UI 设计稿：`design/ui-mockup.html`。
- **还没有功能版应用代码**。
- 本文档先准备部署结构；功能版 MVP 完成后，把构建产物放进 `site/quit-smoking/` 即可。

## 最终目录结构

```
/var/www/quit-smoking/
├── index.html
├── manifest.webmanifest
├── sw.js
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
└── design/                 # 可选：设计稿预览
    └── ui-mockup.html
```

## Nginx

复制 `deploy/quit-smoking.nginx.conf` 中的 location 到你的站点配置，然后：

```bash
sudo nginx -t
sudo systemctl reload nginx
```

要点：

- 文件放在 `/var/www/quit-smoking/`。
- 不要修改 `location /`，根项目不受影响。
- `try_files` 让子路径刷新不 404。
- `index.html` 不强缓存，静态资源可长缓存。

## Apache（备选）

```apache
Alias /quit-smoking /var/www/quit-smoking
<Directory /var/www/quit-smoking>
    Require all granted
    FallbackResource /quit-smoking/index.html
</Directory>
```

## 上传

```bash
scp -r site/quit-smoking user@<服务器IP>:/var/www/
```

或在服务器上：

```bash
unzip quit-smoking.zip -d /var/www/
sudo chown -R www-data:www-data /var/www/quit-smoking
sudo chmod -R 755 /var/www/quit-smoking
```

宝塔/腾讯云面板：把 `quit-smoking` 文件夹上传到站点根目录即可。

## PWA 与 HTTPS

- Service Worker 只在 **HTTPS** 或 `localhost` 下可用。
- `http://<IP>/quit-smoking/` 可以正常浏览网页，但**离线缓存/PWA 安装不完整**。
- 需要完整 PWA 时：给服务器配域名 + HTTPS 证书（腾讯云免费证书即可）。
- 没有域名时，先按普通移动网页使用，PWA 能力后置。

## 路径规则（开发时必须遵守）

- 所有资源使用相对路径：`./assets/...`，不要用 `/assets/...`。
- manifest：`start_url: "./"`、`scope: "./"`。
- Service Worker：`register("./sw.js", { scope: "./" })`。
- 不要占用根路径的 `manifest`、`sw.js`、`favicon`。

## 自检

1. `curl -I http://<IP>/quit-smoking/` → 200
2. 手机打开，CSS/JS/图标无 404
3. 根路径 `/` 项目正常
4. 刷新子路径、直接访问深层 URL 不 404
5. HTTPS 下检查 manifest、Service Worker、离线刷新

## 当前建议

先完成单文件 MVP，再打包部署。这样部署的是可用工具，而不是只能看的设计板。若只想先在手机看设计，可单独打一个设计预览包。
