# 戒烟 PWA · 部署说明

> 目标：`http://<服务器IP>/quit-smoking/`
> 服务器根路径 `/` 已有另一个项目，本方案只新增子目录，不改根 location。

## 当前状态

- MVP 已完成：`site/quit-smoking/`。
- 部署包：`deploy/quit-smoking.zip`（解压后根目录就是 `index.html`）。
- 本文档按“服务器根路径已有另一个项目，新增 `/quit-smoking/` 子目录”编写。

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

## 腾讯云服务器操作顺序（推荐 zip 上传）

假设服务器是 Linux + Nginx，已能 SSH 登录，根路径已有项目。

### 1. 本地 Windows 上传 zip

```powershell
scp "E:\开发\001\deploy\quit-smoking.zip" root@<服务器IP>:/tmp/
```

如果用户名不是 `root`，换成实际用户（如 `ubuntu`、`centos`）。

### 2. 登录服务器并解压

```bash
ssh root@<服务器IP>

sudo mkdir -p /var/www/quit-smoking
sudo unzip -o /tmp/quit-smoking.zip -d /var/www/quit-smoking

# Ubuntu / Debian
sudo chown -R www-data:www-data /var/www/quit-smoking

# CentOS / TencentOS 通常是 nginx 用户
# sudo chown -R nginx:nginx /var/www/quit-smoking

sudo chmod -R 755 /var/www/quit-smoking
ls -l /var/www/quit-smoking/index.html
```

### 3. 在现有站点的 server 块里加 location

找到当前项目使用的 Nginx 配置，例如：

- `/etc/nginx/conf.d/*.conf`
- `/etc/nginx/sites-available/*`
- 宝塔：`/www/server/panel/vhost/nginx/*.conf`

在 **已有的 `server { ... }` 里面**加：

```nginx
location = /quit-smoking {
    return 301 /quit-smoking/;
}

location /quit-smoking/ {
    root /var/www;
    index index.html;
    try_files $uri $uri/ /quit-smoking/index.html;
}
```

如果文件放在现有站点根目录下（例如宝塔的 `/www/wwwroot/example.com/quit-smoking/`），可以省略 `root`，只保留：

```nginx
location /quit-smoking/ {
    try_files $uri $uri/ /quit-smoking/index.html;
}
```

### 4. 检查并重载

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 5. 验证

```bash
curl -I http://<服务器IP>/quit-smoking/
```

浏览器访问：

```
http://<服务器IP>/quit-smoking/
```

根路径 `/` 的另一个项目应保持正常。

### 常见问题

- **404**：`root` 路径不对，或文件不在 `/var/www/quit-smoking/index.html`。
- **403**：目录权限不对，或 CentOS SELinux 限制；先确认 `chown` 用户与 Nginx 运行用户一致。
- **样式/图标 404**：确认 zip 解压后 `icons/` 和 `manifest.webmanifest` 与 `index.html` 同级。
- **刷新子路径 404**：确认 `try_files` 已加。
- **PWA 安装/离线不可用**：HTTP + IP 不支持 Service Worker，需要 HTTPS。

### 宝塔面板

1. 打开现有站点根目录（如 `/www/wwwroot/example.com/`）。
2. 上传 `quit-smoking.zip` 并解压，得到 `quit-smoking/` 文件夹。
3. 站点设置 → 配置文件，在 `server { }` 内加：
   ```nginx
   location /quit-smoking/ {
       try_files $uri $uri/ /quit-smoking/index.html;
   }
   ```
4. 保存 → 重载 Nginx。

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
