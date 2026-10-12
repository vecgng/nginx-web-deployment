# Nginx Web Deployment

## 项目简介

本项目基于 Rocky Linux 9 和 Nginx，完成静态网站的部署及基础运维实践。

通过 VMware 虚拟机搭建 Linux 服务器环境，实现 Windows 浏览器访问 Nginx 网站，并完成服务管理、日志查看和常见故障排查。

## 技术环境

- 操作系统：Rocky Linux 9.8
- Web 服务器：Nginx 1.20.1
- 虚拟化平台：VMware
- 服务管理：systemd
- 防火墙：firewalld
- 安全机制：SELinux

## 已实现功能

- 安装和配置 Nginx
- 部署自定义 HTML 网页
- 配置 HTTP 80 端口访问
- 设置 Nginx 开机自启
- 查看访问日志和错误日志
- 排查 HTTP 403 权限问题
- 模拟 Nginx 服务停止和配置错误，并完成恢复

## 项目文件

- `index.html`：静态网站首页
- `nginx-site.conf.example`：Nginx 站点配置示例
- `README.md`：项目说明文档

## 部署步骤

### 1. 安装 Nginx

```bash
sudo dnf install -y nginx
```

### 2. 部署网站文件

```bash
sudo mkdir -p /var/www/mywebsite
sudo cp index.html /var/www/mywebsite/
```

### 3. 配置 Nginx

编辑 `/etc/nginx/nginx.conf`，在现有的 HTTP 站点配置中设置：

```nginx
listen 80;
server_name _;
root /var/www/mywebsite;
index index.html;
```

修改完成后检查配置：

```bash
sudo nginx -t
```

### 4. 修复 SELinux 文件标签

```bash
sudo restorecon -Rv /var/www/mywebsite
```

### 5. 启动服务并设置开机自启

```bash
sudo systemctl enable --now nginx
```

### 6. 放行 HTTP 服务

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

### 7. 验证部署结果

检查 Nginx：

```bash
systemctl status nginx --no-pager
```

本机测试：

```bash
curl -I http://127.0.0.1
```

最后，在能够访问该服务器的浏览器中输入：

```text
http://服务器IP地址
```

成功显示 `Welcome to My Website!`，即表示静态网站部署成功。

## 故障排查记录

### 1. Windows 浏览器无法访问 Nginx

**故障现象：** Linux 本机使用 `curl` 访问正常，但 Windows 浏览器访问服务器 IP 时连接超时。

**排查过程：**
- 使用 `systemctl status nginx` 确认服务正在运行。
- 使用 `ss -lntp` 确认 Nginx 监听 TCP 80 端口。
- 检查 firewalld，发现未放行 HTTP 服务。

**解决方法：**

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

**排查结果：** Windows 浏览器可以正常访问网站。

### 2. 网站出现 403 Forbidden

**故障现象：** 修改网站根目录后，浏览器返回 HTTP 403。

**排查过程：**
- 查看 `/var/log/nginx/error.log`，发现 `Permission denied`。
- 检查文件的普通读写权限。
- 使用 `ls -Z` 和 `matchpathcon` 检查 SELinux 标签，发现实际标签与默认标签不一致。

**解决方法：**

```bash
sudo restorecon -Rv /var/www/mywebsite
```

**排查结果：** 恢复正确的 SELinux 标签后，网站正常显示。

### 3. Nginx 服务停止

**故障现象：** 模拟停止 Nginx 后，网站无法访问。

**解决方法：**

```bash
sudo systemctl start nginx
systemctl status nginx --no-pager
```

**排查结果：** Nginx 恢复运行，网站重新可以访问。

### 4. Nginx 配置语法错误

**故障现象：** 故意删除 `root` 指令末尾的分号，执行配置检查时报错。

**排查过程：** 使用 `nginx -t` 检查配置，根据错误信息定位问题。

**解决方法：** 恢复缺失的分号，再次执行：

```bash
sudo nginx -t
sudo systemctl reload nginx
```

**排查结果：** 配置检查通过，网站正常运行。

## 项目总结

通过本项目，熟悉了 Linux 环境下 Nginx 的部署流程，实践了 systemd 服务管理、HTTP 访问测试、firewalld 防火墙配置、SELinux 权限排查以及 Nginx 日志分析。

掌握了从服务状态、端口监听、网络访问、错误日志和配置文件等角度定位基础 Web 服务故障的方法。
