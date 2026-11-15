# NapCat Docker 部署指南（ECS 服务器）

> 基于高带宽 ECS + Docker 部署 NapCat，对接 Koishi 的完整流程记录。

## 1. 配置 Docker 镜像加速器

```bash
sudo tee /etc/docker/daemon.json > /dev/null <<'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerhub.icu",
    "https://docker.chenby.cn",
    "https://hub.rat.dev"
  ]
}
EOF
sudo systemctl restart docker
```

验证：`sudo docker info | grep -A5 "Registry Mirrors"`

## 2. 拉取 NapCat 官方镜像

```bash
sudo docker pull mlikiowa/napcat-docker:latest
```

## 3. 启动容器

```bash
mkdir -p /home/admin/napcat/config

sudo docker run -d \
  --name napcat \
  -p 3000:3000 \
  -p 3001:3001 \
  -p 6099:6099 \
  -v /home/admin/napcat/config:/app/.config/QQ \
  -e ACCOUNT=你的QQ号 \
  --restart unless-stopped \
  mlikiowa/napcat-docker:latest
```

### 端口说明

| 端口 | 用途 |
|------|------|
| 3000 | HTTP API |
| 3001 | 正向 WebSocket（OneBot） |
| 6099 | WebUI 管理面板 |

> 所有端口需在阿里云安全组入方向放行 TCP。

## 4. 扫码登录

```bash
sudo docker logs napcat 2>&1 | grep -i "二维码"
```

三种方式任选：
1. 拷贝二维码到本地：`sudo docker cp napcat:/app/napcat/cache/qrcode.png ./qrcode.png`
2. 复制日志中的解码 URL 到二维码生成网站
3. 公网访问 `http://公网IP:6099/napcat/cache/qrcode.png`

登录凭据保存在挂载目录，容器重建无需重新扫码。

## 5. 获取 WebUI Token

```bash
sudo docker logs napcat 2>&1 | grep "WebUi Token"
```

访问：`http://公网IP:6099/webui?token=你的token`

> 每次容器重建 token 会更新。

## 6. 配置 OneBot 对接 Koishi

### 方案 A：反向 WebSocket（推荐）

NapCat 主动连接 Koishi，适合同服务器部署。

在 WebUI → OneBot11 → WebSocket 反向服务中填写：
```
ws://127.0.0.1:Koishi端口/onebot
```

例如 Koishi 监听 5140：`ws://127.0.0.1:5140/onebot`

> 路径必须是 `/onebot`，不是 `/ws`。
> 同服务器用 `127.0.0.1`，不要用公网 IP。

### 方案 B：正向 WebSocket

Koishi 主动连接 NapCat。在 WebUI 中将 WebSocket 服务 host 改为 `0.0.0.0`。

Koishi adapter-onebot 配置：
```yaml
protocol: ws
endpoint: ws://127.0.0.1:3001
token: <从WebUI或配置文件获取>
```

## 7. 常用运维命令

```bash
sudo docker logs napcat --tail 50       # 查看日志
sudo docker logs napcat -f              # 实时跟踪
sudo docker restart napcat              # 重启
sudo docker exec -it napcat bash        # 进入容器
```

## 8. 常见问题

- **公网无法访问**：检查安全组是否放行对应端口
- **反向 WS 连接意外关闭**：路径应为 `/onebot`，地址用 `127.0.0.1`
- **反向 WS 连接被拒绝**：目标端口无服务监听，检查 Koishi 是否启动
- **debug 日志太多**：不影响功能，配置文件中 `"debug": true` 改为 `false`
- **容器重建后需重新登录吗**：不需要，挂载了配置目录则凭据自动保留
