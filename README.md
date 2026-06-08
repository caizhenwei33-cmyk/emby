# 🎬 Emby Media Server — 完整部署与使用指南 / Complete Deployment & Usage Guide

> 一个功能强大的家庭媒体服务器解决方案，助你打造私人影音中心。
> A powerful home media server solution to build your personal streaming center.

[![Docker](https://img.shields.io/badge/Docker-Supported-2496ED?logo=docker)](https://docker.com)
[![Linux](https://img.shields.io/badge/Linux-Supported-FCC624?logo=linux)](https://kernel.org)
[![Emby](https://img.shields.io/badge/Emby-4.8%2B-52B54B)](https://emby.media)

---

## 📖 目录 / Table of Contents

- [1. 什么是 Emby？ / What is Emby?](#1-什么是-emby--what-is-emby)
- [2. 环境准备 / Prerequisites](#2-环境准备--prerequisites)
- [3. Docker 部署 / Docker Deployment](#3-docker-部署--docker-deployment)
- [4. 硬件转码 / Hardware Transcoding](#4-硬件转码--hardware-transcoding)
- [5. 媒体库设置 / Library Setup](#5-媒体库设置--library-setup)
- [6. 用户管理与分享 / User Management & Sharing](#6-用户管理与分享--user-management--sharing)
- [7. 插件 / Plugins](#7-插件--plugins)
- [8. Nginx 反代 + SSL / Reverse Proxy with Nginx + SSL](#8-nginx-反代--ssl--reverse-proxy-with-nginx--ssl)
- [9. 常见问题与排错 / Common Issues & Troubleshooting](#9-常见问题与排错--common-issues--troubleshooting)
- [10. 进阶技巧 / Advanced Tips](#10-进阶技巧--advanced-tips)
- [☕ 支持这个项目 / Support This Project](#-支持这个项目--support-this-project)

---

## 1. 什么是 Emby？ / What is Emby?

**中文**

Emby 是一款开源的媒体服务器软件，能将你收藏的电影、电视剧、音乐、照片集中管理，并通过网页、手机 App、电视、游戏机等设备串流播放。Emby 支持：

- **自动刮削元数据**：从 TMDB、TVDB 等自动获取封面、简介、演员信息。
- **实时转码**：根据客户端能力自动转换视频格式和分辨率。
- **多用户管理**：为家人或朋友创建独立账户，控制访问权限。
- **跨平台客户端**：支持 iOS、Android、Web、Apple TV、Android TV、Roku、Fire TV 等。
- **硬件加速转码**：利用 Intel QuickSync、NVIDIA NVENC、VAAPI 等提升转码性能。

**English**

Emby is an open-source media server that centralizes your movie, TV show, music, and photo collections, streaming them to web browsers, mobile apps, smart TVs, game consoles, and more. Key features include:

- **Automatic metadata scraping**: Fetches posters, descriptions, cast info from TMDB, TVDB, etc.
- **Live transcoding**: Converts video formats and resolutions on-the-fly based on client capabilities.
- **Multi-user management**: Create separate accounts for family or friends with controlled access.
- **Cross-platform clients**: iOS, Android, Web, Apple TV, Android TV, Roku, Fire TV, and more.
- **Hardware-accelerated transcoding**: Leverages Intel QuickSync, NVIDIA NVENC, VAAPI for performance.

---

## 2. 环境准备 / Prerequisites

**中文**

开始前，请确保你满足以下条件：

| 项目 | 要求 |
|-------|--------|
| 操作系统 | Linux (推荐 Ubuntu 22.04+ / Debian 12+ / CentOS Stream 9) |
| Docker | 24.0+（含 Docker Compose v2） |
| CPU | x86_64 或 ARM64 — 建议 4 核以上 |
| 内存 | 至少 4GB（推荐 8GB+） |
| 存储 | 媒体文件存储 + 元数据 SSD（推荐） |

**English**

Before you begin, ensure you have the following:

| Item | Requirement |
|------|-------------|
| OS | Linux (recommended Ubuntu 22.04+ / Debian 12+ / CentOS Stream 9) |
| Docker | 24.0+ (with Docker Compose v2) |
| CPU | x86_64 or ARM64 — 4+ cores recommended |
| RAM | 4GB minimum (8GB+ recommended) |
| Storage | Media files storage + SSD for metadata (recommended) |

### 安装 Docker / Install Docker

**中文**

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com | sudo bash
sudo usermod -aG docker $USER
newgrp docker

# 验证安装
docker --version
docker compose version
```

**English**

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com | sudo bash
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
docker compose version
```

---

## 3. Docker 部署 / Docker Deployment

**中文**

使用 Docker Compose 是部署 Emby 最推荐的方式。创建以下 `docker-compose.yml` 文件：

**English**

Using Docker Compose is the most recommended way to deploy Emby. Create the following `docker-compose.yml` file:

```yaml
version: "3.8"

services:
  emby:
    image: emby/embyserver:latest
    container_name: emby
    restart: unless-stopped
    ports:
      - "8096:8096"   # HTTP Web UI
      - "8920:8920"   # HTTPS (optional)
    volumes:
      # 配置文件 / Config
      - ./config:/config
      # 媒体库挂载 / Media libraries
      - /path/to/movies:/data/movies:ro
      - /path/to/tvshows:/data/tvshows:ro
      - /path/to/music:/data/music:ro
      # 转码缓存 / Transcode cache
      - ./transcodes:/config/transcodes
    environment:
      - TZ=Asia/Shanghai   # 时区 / Timezone
      - PUID=1000          # 用户 ID / User ID
      - PGID=1000          # 组 ID / Group ID
    devices:
      # 硬件转码设备（按需启用） / HW transcoding (enable as needed)
      - /dev/dri:/dev/dri                    # Intel QuickSync / VAAPI
      - /dev/nvidia0:/dev/nvidia0            # NVIDIA GPU
      - /dev/nvidiactl:/dev/nvidiactl
      - /dev/nvidia-modeset:/dev/nvidia-modeset
```

### 启动服务 / Start the Service

**中文**

```bash
# 创建配置目录
mkdir -p emby && cd emby

# 保存上面的 docker-compose.yml

# 启动
docker compose up -d

# 查看日志
docker compose logs -f

# 访问 http://YOUR_SERVER_IP:8096 完成初始化设置
```

**English**

```bash
# Create config directory
mkdir -p emby && cd emby

# Save the docker-compose.yml above

# Start service
docker compose up -d

# Check logs
docker compose logs -f

# Visit http://YOUR_SERVER_IP:8096 to complete initial setup
```

### 更新 Emby / Update Emby

**中文**

```bash
cd /path/to/emby
docker compose pull
docker compose up -d
docker image prune
```

**English**

```bash
cd /path/to/emby
docker compose pull
docker compose up -d
docker image prune
```

---

## 4. 硬件转码 / Hardware Transcoding

**中文**

硬件转码能充分利用 GPU 加速视频编码/解码，显著降低 CPU 占用。Emby 需要 **Premiere 订阅** 才能启用硬件转码。

**English**

Hardware transcoding leverages your GPU to accelerate video encoding/decoding, significantly reducing CPU load. Emby requires a **Premiere subscription** to enable hardware transcoding.

### Intel QuickSync (QSV) — 第 6 代以上 / 6th Gen+

**中文**

| 步骤 | 命令 |
|------|-------|
| 检查 Intel GPU | `ls /dev/dri` — 应看到 `renderD128` |
| 安装驱动 | Ubuntu: `sudo apt install intel-media-va-driver-non-free` |
| Docker 配置 | 在 compose 中添加 `devices: - /dev/dri:/dev/dri` |
| Emby 设置 | 管理后台 → 转码 → Intel QuickSync → 勾选所有支持的编码器 |

**English**

| Step | Command |
|------|---------|
| Check Intel GPU | `ls /dev/dri` — should show `renderD128` |
| Install driver | Ubuntu: `sudo apt install intel-media-va-driver-non-free` |
| Docker config | Add `devices: - /dev/dri:/dev/dri` to compose |
| Emby setting | Dashboard → Transcoding → Intel QuickSync → enable all supported encoders |

### NVIDIA GPU (NVENC/NVDEC)

**中文**

| 步骤 | 命令 |
|------|-------|
| 安装驱动 | `sudo apt install nvidia-driver-550` 或使用官方 runfile |
| 安装 Container Toolkit | `sudo apt install nvidia-container-toolkit && sudo systemctl restart docker` |
| Docker 配置 | 添加 `deploy.resources.reservations.devices` 一节（见下方） |
| Emby 设置 | 管理后台 → 转码 → NVIDIA NVENC → 勾选 H.264 / HEVC |

**English**

| Step | Command |
|------|---------|
| Install driver | `sudo apt install nvidia-driver-550` or use official runfile |
| Install Container Toolkit | `sudo apt install nvidia-container-toolkit && sudo systemctl restart docker` |
| Docker config | Add `deploy.resources.reservations.devices` section (see below) |
| Emby setting | Dashboard → Transcoding → NVIDIA NVENC → enable H.264 / HEVC |

```yaml
# docker-compose.yml 中的 NVIDIA 配置
services:
  emby:
    image: emby/embyserver:latest
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu, video, compute]
```

### VAAPI (AMD / 其它 GPU / Other GPUs)

**中文**

VAAPI 是通用的 Linux 视频加速 API，支持 AMD 和部分 Intel GPU。

```yaml
services:
  emby:
    devices:
      - /dev/dri:/dev/dri
    group_add:
      - "44"   # video 组
      - "107"  # render 组
```

Emby 设置：管理后台 → 转码 → VAAPI → 选择硬件加速。

**English**

VAAPI is a generic video acceleration API for Linux, supporting AMD and some Intel GPUs.

```yaml
services:
  emby:
    devices:
      - /dev/dri:/dev/dri
    group_add:
      - "44"   # video group
      - "107"  # render group
```

Emby setting: Dashboard → Transcoding → VAAPI → select hardware acceleration.

---

## 5. 媒体库设置 / Library Setup

**中文**

### 推荐目录结构 / Recommended Directory Structure

```
/media/
├── movies/              # 电影 / Movies
│   ├── Avatar (2009)/
│   │   ├── Avatar.2009.2160p.BluRay.mkv
│   │   └── subtitles/
│   └── Inception (2010)/
│       └── Inception.2010.1080p.mkv
├── tvshows/             # 电视剧 / TV Shows
│   └── Breaking Bad/
│       ├── Season 01/
│       │   ├── Breaking.Bad.S01E01.mkv
│       │   └── Breaking.Bad.S01E02.mkv
│       └── Season 02/
└── music/               # 音乐 / Music
    ├── Artist1/
    │   └── Album1/
    │       └── track1.flac
    └── Artist2/
```

### 命名规范 / Naming Conventions

**中文**

Emby 自动识别媒体文件依赖规范的命名。以下是最佳实践：

| 媒体类型 | 推荐格式 | 示例 |
|----------|----------|------|
| 电影 | `电影名 (年份).扩展名` | `The Matrix (1999).mkv` |
| 电视剧 | `剧名 SxxExx.扩展名` | `Game.of.Thrones.S01E01.mkv` |
| 多碟版 | 包含 `-cd1`, `-cd2` | `Movie (2000)-cd1.mkv` |
| 字幕 | 与视频同名 + `.chs.srt` | `Movie (1999).chs.srt` |

**English**

Emby relies on proper naming conventions for automatic media identification. Best practices:

| Media Type | Format | Example |
|------------|--------|---------|
| Movie | `Movie Name (Year).ext` | `The Matrix (1999).mkv` |
| TV Show | `Show Name SxxExx.ext` | `Game.of.Thrones.S01E01.mkv` |
| Multi-disc | Include `-cd1`, `-cd2` | `Movie (2000)-cd1.mkv` |
| Subtitles | Video filename + `.chs.srt` | `Movie (1999).chs.srt` |

### 添加媒体库 / Adding a Library

**中文**

1. 打开 Emby Web UI（http://YOUR_SERVER_IP:8096）
2. 管理后台 → 媒体库 → 新增媒体库
3. 选择内容类型（电影 / 电视剧 / 音乐 / 照片）
4. 填写显示名称（如「我的电影」）
5. 添加文件夹路径（对应 docker volume 挂载点，如 `/data/movies`）
6. 选择元数据刮削器（建议勾选 TMDB、TVDB、The Open Movie Database）
7. 设置语言偏好（首选中文，回退英文）
8. 点击确定 → Emby 开始扫描和刮削

**English**

1. Open Emby Web UI (http://YOUR_SERVER_IP:8096)
2. Dashboard → Libraries → Add Media Library
3. Select content type (Movies / TV Shows / Music / Photos)
4. Enter display name (e.g., "My Movies")
5. Add folder path (corresponds to docker volume mount, e.g., `/data/movies`)
6. Select metadata scrapers (recommended: TMDB, TVDB, The Open Movie Database)
7. Set language preference (prefer Chinese, fallback English)
8. Click OK → Emby starts scanning and scraping

---

## 6. 用户管理与分享 / User Management & Sharing

**中文**

### 创建用户 / Creating a User

1. 管理后台 → 用户 → 新增用户
2. 设置用户名和密码
3. 配置媒体库访问权限：勾选该用户能访问的库
4. 设置家长控制：限制特定分级的内容
5. 启用/禁用设备流媒体传输

### 用户权限说明 / Permission Levels

| 角色 | 说明 | 典型用途 |
|------|------|----------|
| 管理员 | 完全控制 | 服务器所有者 |
| 普通用户 | 按库限制访问 | 家庭成员 |
| 受限制用户 | 受控内容 + 家长控制 | 儿童账户 |
| 访客 | 仅观看，无设置权限 | 临时分享 |

### 外网分享 / Remote Sharing

**中文**

1. 启用远程访问：管理后台 → 网络 → 允许远程连接
2. 设置端口转发（默认 8096）或使用反向代理（推荐）
3. 创建用户账户 → 分享给家人/朋友
4. 或使用 Emby Connect 服务简化连接

**English**

1. Enable remote access: Dashboard → Network → Allow remote connections
2. Set up port forwarding (default 8096) or use reverse proxy (recommended)
3. Create user accounts → share with family/friends
4. Or use Emby Connect service for simplified connectivity

---

## 7. 插件 / Plugins

**中文**

Emby 插件系统可以扩展功能。以下是最常用的插件：

**English**

The Emby plugin system extends functionality. Here are the most popular plugins:

### 推荐插件列表 / Recommended Plugins

| 插件名称 | 功能 | 安装方式 |
|-----------|------|----------|
| **Open Subtitles** | 自动下载字幕 | Catalog → Subtitles |
| **Fanart** | 获取高质量背景图 | Catalog → Metadata |
| **Trailers** | 自动获取预告片 | Catalog → Other |
| **Skin Manager** | 更换 UI 主题 | Catalog → General |
| **AudioDB** | 音乐元数据增强 | Catalog → Metadata |
| **Subtitle Edit** | 字幕调整（延迟/偏移） | Catalog → Subtitles |

### 安装插件 / Installing Plugins

**中文**

管理后台 → 插件 → 目录 → 搜索插件名称 → 点击安装 → 重启 Emby 服务。

**English**

Dashboard → Plugins → Catalog → Search plugin name → Install → Restart Emby service.

### 手动安装插件 / Manual Plugin Installation

**中文**

```bash
# 1. 下载 .dll 文件到 config/plugins 目录
cd /path/to/emby/config/plugins
wget https://example.com/SomePlugin.dll

# 2. 设置正确权限
chmod 644 SomePlugin.dll

# 3. 重启 Emby
docker restart emby
```

**English**

```bash
# 1. Download .dll file to config/plugins directory
cd /path/to/emby/config/plugins
wget https://example.com/SomePlugin.dll

# 2. Set proper permissions
chmod 644 SomePlugin.dll

# 3. Restart Emby
docker restart emby
```

---

## 8. Nginx 反代 + SSL / Reverse Proxy with Nginx + SSL

**中文**

使用 Nginx 反向代理可以安全地暴露 Emby 到公网，并自动管理 SSL 证书。

**English**

Using Nginx as a reverse proxy securely exposes Emby to the internet with automatic SSL certificate management.

### Nginx 配置 / Nginx Configuration

```nginx
server {
    listen 80;
    server_name emby.yourdomain.com;

    # 自动跳转 HTTPS / Auto HTTPS redirect
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name emby.yourdomain.com;

    # SSL 证书路径（用 certbot 生成）
    ssl_certificate     /etc/letsencrypt/live/emby.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/emby.yourdomain.com/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    # 请求体大小（用于上传字幕等）
    client_max_body_size 100M;

    location / {
        proxy_pass http://127.0.0.1:8096;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket 支持（用于实时播放进度）
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 缓存优化（减少带宽）
        proxy_buffering off;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Certbot SSL 证书 / SSL Certificate with Certbot

**中文**

```bash
# 安装 Certbot
sudo apt install certbot python3-certbot-nginx

# 申请证书
sudo certbot --nginx -d emby.yourdomain.com

# 自动续期（证书有效期 90 天）
sudo certbot renew --dry-run
```

**English**

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Request certificate
sudo certbot --nginx -d emby.yourdomain.com

# Auto-renewal (certificates valid for 90 days)
sudo certbot renew --dry-run
```

### Docker Compose 集成 Nginx + Certbot

**中文**

```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./ssl:/etc/letsencrypt:ro
    depends_on:
      - emby

  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - ./ssl:/etc/letsencrypt
      - ./webroot:/var/www/certbot
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h; done'"
```

**English**

```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./ssl:/etc/letsencrypt:ro
    depends_on:
      - emby

  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - ./ssl:/etc/letsencrypt
      - ./webroot:/var/www/certbot
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h; done'"
```

---

## 9. 常见问题与排错 / Common Issues & Troubleshooting

**中文**

### 1. 无法访问 Emby Web UI

| 可能原因 | 解决方法 |
|----------|----------|
| Docker 未运行 | `systemctl status docker && docker compose ps` |
| 端口冲突 | `ss -tlnp | grep 8096` 检查端口占用 |
| 防火墙阻止 | `sudo ufw allow 8096` 或检查云服务商安全组 |
| SELinux | `sudo setenforce 0` 临时关闭测试 |

### 2. 硬件转码不工作

```bash
# 检查 GPU 设备是否存在
ls -la /dev/dri*
ls -la /dev/nvidia*

# 检查 Docker 容器是否能访问设备
docker exec emby ls /dev/dri/

# 检查 Emby 日志中的转码错误
docker logs emby | grep -i "transcode\|ffmpeg\|error"
```

### 3. 元数据刮削失败

- **检查网络**：Emby 需要访问 `api.themoviedb.org`、`thetvdb.com` 等
- **设置 DNS**：在 `docker-compose.yml` 中添加 `dns: 8.8.8.8`
- **手动刷新**：管理后台 → 媒体库 → 扫描媒体文件
- **API Key**：确保 TMDB API Key 有效（管理后台 → 元数据）

### 4. 播放卡顿/缓冲

| 原因 | 解决方案 |
|------|----------|
| 带宽不足 | 降低客户端播放码率 |
| 转码性能不足 | 启用硬件转码，降低转码质量 |
| 磁盘 I/O 瓶颈 | 使用 SSD 存放元数据和转码缓存 |
| 网络延迟 | 使用有线连接，避免 WiFi |

### 5. 字幕显示异常 / Subtitle Issues

- 确保字幕文件编码为 UTF-8
- 在播放器设置中选择正确的字幕轨道
- 安装「Subtitle Edit」插件进行延迟调整
- 中文乱码：将 `.srt` 文件转为 UTF-8 编码

**English**

### 1. Cannot Access Emby Web UI

| Possible Cause | Solution |
|----------------|----------|
| Docker not running | `systemctl status docker && docker compose ps` |
| Port conflict | `ss -tlnp | grep 8096` check port usage |
| Firewall blocking | `sudo ufw allow 8096` or check cloud firewall rules |
| SELinux | `sudo setenforce 0` temporarily for testing |

### 2. Hardware Transcoding Not Working

```bash
# Check if GPU devices exist
ls -la /dev/dri*
ls -la /dev/nvidia*

# Verify container can access devices
docker exec emby ls /dev/dri/

# Check Emby logs for transcoding errors
docker logs emby | grep -i "transcode\|ffmpeg\|error"
```

### 3. Metadata Scraping Fails

- **Check network**: Emby needs access to `api.themoviedb.org`, `thetvdb.com`, etc.
- **Set DNS**: Add `dns: 8.8.8.8` to `docker-compose.yml`
- **Manual refresh**: Dashboard → Libraries → Scan media files
- **API Key**: Ensure TMDB API Key is valid (Dashboard → Metadata)

### 4. Playback Stuttering/Buffering

| Cause | Solution |
|-------|----------|
| Insufficient bandwidth | Lower client playback bitrate |
| Weak transcoding performance | Enable hardware transcoding, lower quality |
| Disk I/O bottleneck | Use SSD for metadata and transcode cache |
| Network latency | Use wired connection, avoid WiFi |

### 5. Subtitle Display Issues

- Ensure subtitle file encoding is UTF-8
- Select the correct subtitle track in player settings
- Install "Subtitle Edit" plugin for delay adjustment
- Chinese garbled text: Convert `.srt` files to UTF-8 encoding

---

## 10. 进阶技巧 / Advanced Tips

**中文**

### 定时扫描媒体库 / Scheduled Library Scan

```bash
# 使用 curl 触发扫描（需要 API Key）
curl -X POST "http://localhost:8096/Library/Media/Updated" \
  -H "X-MediaBrowser-Token: YOUR_API_KEY"
```

### 限制转码分辨率 / Limit Transcoding Resolution

管理后台 → 转码 → 播放器转码限制 → 勾选「限制转码至此分辨率以下」→ 选择 1080p 或 4K。

### 使用 RAM 作为转码缓存 / RAM Transcode Cache

**中文**

将转码缓存移到内存中，大幅提升性能。

```yaml
# docker-compose.yml
services:
  emby:
    volumes:
      - /dev/shm:/transcodes
```

然后在 Emby 设置中将转码路径改为 `/transcodes`。

**English**

Move transcode cache to RAM for significant performance improvement.

```yaml
# docker-compose.yml
services:
  emby:
    volumes:
      - /dev/shm:/transcodes
```

Then set the transcode path to `/transcodes` in Emby settings.

### 备份与恢复 / Backup & Restore

**中文**

```bash
# 备份
tar -czf emby-backup-$(date +%Y%m%d).tar.gz \
  -C /path/to/emby/config .

# 恢复
tar -xzf emby-backup-20250101.tar.gz \
  -C /path/to/emby/config
```

备份文件包含：用户数据、播放记录、元数据缓存、插件配置。

**English**

```bash
# Backup
tar -czf emby-backup-$(date +%Y%m%d).tar.gz \
  -C /path/to/emby/config .

# Restore
tar -xzf emby-backup-20250101.tar.gz \
  -C /path/to/emby/config
```

Backup includes: user data, play history, metadata cache, plugin configs.

### 日志管理与轮转 / Log Management & Rotation

**中文**

```yaml
# docker-compose.yml 中添加日志限制
services:
  emby:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"
```

**English**

```yaml
# Add log limits in docker-compose.yml
services:
  emby:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"
```

---

## ☕ 支持这个项目 / Support This Project

如果你觉得这个教程对你有帮助，欢迎请我喝杯咖啡 ☕  
你的支持是我持续更新的最大动力！非常感谢！🙏

If you find this tutorial helpful, feel free to buy me a coffee ☕  
Your support is my biggest motivation to keep improving! Thank you so much! 🙏

**USDT (TRC20)**
```
TVbQerV1SF4MXB1JCcAzQxarewHwEPYTKm
```
