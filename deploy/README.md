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

## 挂到现有项目子目录（无需新 Deploy Key）

如果服务器上已经有 `shangcheng` 项目的站点目录，可以直接把戒烟静态文件放进它的子目录，不需要为 `quit-smoking` 单独配 GitHub 密钥。

### 方式 1：上传 zip 解压（最简单）

本地：

```powershell
scp "E:\开发\001\deploy\quit-smoking.zip" root@<服务器IP>:/tmp/
```

服务器：

```bash
# 假设 shangcheng 的站点根目录是 /var/www/shangcheng
cd /var/www/shangcheng
sudo mkdir -p quit-smoking
sudo unzip -o /tmp/quit-smoking.zip -d quit-smoking
sudo chown -R www-data:www-data quit-smoking
sudo chmod -R 755 quit-smoking
```

访问地址取决于 shangcheng 挂在哪：

- shangcheng 在根路径 `/`：`http://<IP>/quit-smoking/`
- shangcheng 在 `/shangcheng/`：`http://<IP>/shangcheng/quit-smoking/`

### 方式 2：软链接（方便更新）

```bash
# 假设 quit-smoking 源码在 /var/www/quit-smoking-src
sudo ln -s /var/www/quit-smoking-src/site/quit-smoking /var/www/shangcheng/quit-smoking
```

要求：

- Nginx 没有开启 `disable_symlinks on`（默认是 off）。
- 符号链接目标对 Nginx 运行用户可读。
- 更新时只需在源码目录 `git pull`，软链接自动生效。

### 如果 shangcheng 是 Next.js / SPA（当前服务器就是）

症状：

- `/quit-smoking/` 返回 `308` 到 `/quit-smoking`
- 然后返回 Next.js 的 `404`
- 响应头里有 `X-Powered-By: Next.js`

原因：Nginx 把所有请求 `proxy_pass` 给 Next.js，静态目录没有被 Nginx 直接服务。

#### 方案 A：Nginx 直接服务静态文件（推荐）

在现有 `server { }` 里加（和 `proxy_pass` 的 location 同级）：

```nginx
location = /quit-smoking {
    return 301 /quit-smoking/;
}

location ^~ /quit-smoking/ {
    root /opt/shangcheng;   # quit-smoking 文件夹的父目录
    index index.html;
    try_files $uri $uri/ /quit-smoking/index.html;
}
```

`^~` 很重要：它让这个前缀匹配优先于 Next.js 的 `proxy_pass` / 正则 location。

路径对应关系：

- 文件在 `/opt/shangcheng/quit-smoking/` → `root /opt/shangcheng;`
- 文件在 `/opt/shangcheng/public/quit-smoking/` → `root /opt/shangcheng/public;`
- 文件在 `/var/www/quit-smoking/` → `root /var/www;`
- 文件在 `/var/www/shangcheng/quit-smoking/` → `root /var/www/shangcheng;`

先确认文件实际存在：

```bash
ls -l /opt/shangcheng/quit-smoking/index.html
# 或
ls -l /opt/shangcheng/public/quit-smoking/index.html
```

#### 当前服务器：文件在 `/var/www/quit-smoking/`

如果戒烟文件放在 `/var/www/quit-smoking/`，配置应为：

```nginx
location = /quit-smoking {
    return 301 /quit-smoking/;
}

location ^~ /quit-smoking/ {
    root /var/www;
    index index.html;
    try_files $uri $uri/ /quit-smoking/index.html;
}
```

这条 location 必须放在 shangcheng 的 `proxy_pass` 所在 `server { }` 内；`^~` 保证它优先于 Next.js 代理。

当前服务器（Ubuntu）实际配置：

- 文件：`/etc/nginx/sites-enabled/shangcheng`
- `server_name shangcheng.club www.shangcheng.club;`
- `location / { proxy_pass http://127.0.0.1:3000; }`
- 在同一个 `server { }` 里加上面的 `/quit-smoking/` location，然后 `nginx -t && systemctl reload nginx`。

### manifest.webmanifest 的 MIME

如果 `curl -I .../manifest.webmanifest` 返回 `application/octet-stream`，部分浏览器会忽略 manifest，影响 PWA 安装。加一个精确 location（优先级高于 `^~` 前缀）：

```nginx
location = /quit-smoking/manifest.webmanifest {
    root /var/www;
    default_type application/manifest+json;
    add_header Cache-Control "no-cache";
}
```

然后 `nginx -t && systemctl reload nginx`，再检查：

```bash
curl -I http://<IP>/quit-smoking/manifest.webmanifest
```

应为 `Content-Type: application/manifest+json`。

#### 方案 B：放进 Next.js 的 `public/`

```bash
cp -r /path/to/site/quit-smoking/* /path/to/shangcheng/public/quit-smoking/
```

访问 `http://<IP>/quit-smoking/index.html` 可以打开。  
但想要干净的 `/quit-smoking/`，仍需要在 Next.js `next.config.js` 加 rewrite，或使用方案 A。优先方案 A。

### 注意

- 如果 shangcheng 本身是 Git 仓库，把 `quit-smoking/` 放进去会变成未跟踪文件；建议加到 shangcheng 的 `.gitignore`，或放在仓库外的静态目录再用软链接。
- 这种方式只是“部署静态文件”，**不等于服务器能拉取 quit-smoking 私有仓库**。如果想让服务器 `git pull` quit-smoking，仍然需要单独的 Deploy Key。
- 每次更新：重新上传 zip 解压覆盖，或 `git pull` + 软链接/复制。

## 如果仓库改为 public（无需 Deploy Key）

公开仓库的读操作不需要认证，服务器可以直接 HTTPS clone：

```bash
sudo rm -rf /var/www/quit-smoking-src
sudo git clone https://github.com/lion2018/quit-smoking.git /var/www/quit-smoking-src

sudo mkdir -p /var/www/quit-smoking
sudo rsync -a --delete /var/www/quit-smoking-src/site/quit-smoking/ /var/www/quit-smoking/
```

更新：

```bash
sudo git -C /var/www/quit-smoking-src pull --ff-only
sudo rsync -a --delete /var/www/quit-smoking-src/site/quit-smoking/ /var/www/quit-smoking/
```

注意：

- public 只免 **读** 认证；从服务器 `push` 仍然需要 key/token。
- GitHub HTTPS 在国内可能慢或中断；失败时改用 zip 上传或镜像。
- 公开前检查 docs 里的本机路径、SSH 公钥、个人健康描述；不要提交私钥、token、真实服务器密码。
- 仓库公开后，原来的 shangcheng Deploy Key 不用动；quit-smoking 不再需要新 key。

## 从 GitHub 拉取（私有仓库，部署密钥）

仓库是私有的，服务器直接 `git clone` 需要认证。推荐给服务器生成一把 **只读 Deploy Key**，添加到 GitHub 仓库的 Deploy keys，而不是用账号密码或 PAT。

### 1. 在服务器生成部署密钥

登录服务器后执行（root 或你的部署用户）：

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t ed25519 -C "tencent-quit-smoking-deploy" -f ~/.ssh/quit_smoking_deploy -N ""
cat ~/.ssh/quit_smoking_deploy.pub
```

复制输出的整行公钥。

### 2. 添加到 GitHub 仓库

打开仓库：

```
https://github.com/lion2018/quit-smoking/settings/keys
```

点击 **Add deploy key**：

- Title：`Tencent Cloud`
- Key：粘贴服务器公钥
- **Allow write access：不要勾选**（服务器只需要拉取）

### 3. 配置 SSH（GitHub 443 端口）

如果服务器访问 `github.com:22` 不稳定，用官方 443 端口：

```bash
cat >> ~/.ssh/config <<'EOF'
Host github.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/quit_smoking_deploy
  IdentitiesOnly yes
EOF
chmod 600 ~/.ssh/config ~/.ssh/quit_smoking_deploy
```

如果 `~/.ssh/config` 里已经有 `Host github.com`，请编辑原配置，不要重复追加。

> 追加 config 时，结尾的 `EOF` 必须顶格（行首不能有空格），否则 heredoc 不会结束。建议直接用 `nano ~/.ssh/config` 编辑。

### 4. 测试

```bash
ssh -T git@github.com
```

成功会显示类似：

```
Hi lion2018/quit-smoking! You've successfully authenticated, but GitHub does not provide shell access.
```

`ssh -T` 返回 1 是正常的，不代表失败。

### 5. 克隆（推荐 root 登录）

```bash
sudo mkdir -p /var/www
sudo git clone git@github.com:lion2018/quit-smoking.git /var/www/quit-smoking-src

# Nginx 实际服务的是仓库里的 site/quit-smoking/
sudo ln -sfn /var/www/quit-smoking-src/site/quit-smoking /var/www/quit-smoking
sudo chmod -R 755 /var/www/quit-smoking-src
```

如果 `sudo git clone` 找不到密钥，说明你用的是非 root 用户：
sudo 会读 `/root/.ssh`，不是你的 `~/.ssh`。改用下面的“非 root 用户”方式。

### 6. 非 root 用户克隆

```bash
cd ~
git clone git@github.com:lion2018/quit-smoking.git

sudo mkdir -p /var/www/quit-smoking
sudo rsync -a --delete ~/quit-smoking/site/quit-smoking/ /var/www/quit-smoking/
sudo chown -R www-data:www-data /var/www/quit-smoking
```

没有 `rsync` 时：

```bash
sudo cp -r ~/quit-smoking/site/quit-smoking/* /var/www/quit-smoking/
```

### 常见报错：`Repository not found`

GitHub 对“私有仓库 + 无权限”统一返回 `Repository not found`，不一定是仓库不存在。按顺序排查：

```bash
whoami
echo $HOME
ls -l ~/.ssh/quit_smoking_deploy*
ssh-keygen -lf ~/.ssh/quit_smoking_deploy.pub
ssh -T git@github.com
ssh -G github.com | grep -Ei 'hostname|port|identityfile|user'
```

判断：

- `ssh -T` 显示 `Hi lion2018/quit-smoking!`：密钥已被该仓库识别；仍报 not found 时，检查 clone 的仓库地址/大小写，或密钥是否加到了另一个仓库。
- `ssh -T` 显示 `Hi lion2018/shangcheng!`：密钥绑在 `shangcheng` 仓库，不是 `quit-smoking`。Deploy Key 是仓库级的，不能同时给两个仓库用：
  - 去 `https://github.com/lion2018/shangcheng/settings/keys` 删除该 key，再添加到 `https://github.com/lion2018/quit-smoking/settings/keys`；或
  - 为 `quit-smoking` 新生成一把 key，并在 SSH config 里用 Host 别名或切换 `IdentityFile`。
- `ssh -T` 显示 `Hi 其他用户名!`：服务器用错了密钥（可能是账号级 key），需要把 `quit_smoking_deploy.pub` 加到 `lion2018/quit-smoking` 的 Deploy keys。
- `Permission denied (publickey)`：密钥没被使用；看 `ssh -G` 的 `identityfile` 是否指向 `/root/.ssh/quit_smoking_deploy`。
- `ssh -G` 的 `hostname` 不是 `ssh.github.com` 或 `port` 不是 `443`：`~/.ssh/config` 没生效，检查是否有更早的 `Host github.com` 配置覆盖。
- 指纹与 GitHub 仓库 Deploy keys 页面显示的不一致：加到 GitHub 的公钥不是这把密钥。

如果 `sudo git clone` 找不到密钥，但普通用户 `git clone` 可以，说明密钥在 `/home/<user>/.ssh`，而 sudo 读的是 `/root/.ssh`。用普通用户 clone + rsync，或把密钥复制到 root 的 `.ssh`（权限 600）。

显式指定密钥测试：

```bash
GIT_SSH_COMMAND='ssh -i ~/.ssh/quit_smoking_deploy -o IdentitiesOnly=yes -o HostName=ssh.github.com -o Port=443' \
  git clone git@github.com:lion2018/quit-smoking.git /tmp/quit-test
```

### 7. 更新代码

root + 软链接方式：

```bash
sudo git -C /var/www/quit-smoking-src pull --ff-only
```

非 root + rsync 方式：

```bash
cd ~/quit-smoking
git pull --ff-only
sudo rsync -a --delete ~/quit-smoking/site/quit-smoking/ /var/www/quit-smoking/
sudo chown -R www-data:www-data /var/www/quit-smoking
```

> 不建议把 GitHub 账号密码或 PAT 直接写进 `git clone https://...` 命令；  
> GitHub 已不接受账号密码做 Git 操作，HTTPS 只能用 PAT。Deploy Key 更安全。

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
- 如果 `/var/www/quit-smoking` 是指向仓库 `site/quit-smoking/` 的软链接，Nginx 默认会跟随；若配置了 `disable_symlinks on`，请改用 rsync 复制。
- 不要修改 `location /`，根项目不受影响。
- `try_files` 让子路径刷新不 404。
- `index.html` 不强缓存，静态资源可长缓存。

## Nginx 权限：`ps` 显示 root 是正常的

`ps -o user= -C nginx | head -1` 通常取到 **master** 进程，master 以 root 运行；真正读取文件的是 worker，通常是 `www-data` 或 `nginx`。

查看所有 Nginx 进程和用户：

```bash
ps -eo user,pid,args | grep '[n]ginx'
```

静态文件不需要 chown 给 root。推荐直接让所有用户可读：

```bash
sudo chmod -R a+rX /var/www/quit-smoking
```

或者 chown 给 worker 用户：

```bash
sudo chown -R www-data:www-data /var/www/quit-smoking   # Ubuntu
# sudo chown -R nginx:nginx /var/www/quit-smoking       # CentOS
```

403 排查：

```bash
namei -l /var/www/quit-smoking/index.html
sudo tail -n 30 /var/log/nginx/error.log
```

`namei` 会显示每一级目录的权限；如果 `/var/www` 对 others 没有 `x`，Nginx 也进不去。

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
