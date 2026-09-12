# TeamSpeak 6 服务器（自托管）

基于官方镜像 [teamspeaksystems/teamspeak6-server](https://hub.docker.com/r/teamspeaksystems/teamspeak6-server) 的轻量自托管部署配置（SQLite 后端，单容器）。

> 注意：TeamSpeak 6 服务端目前仍为 **Beta** 版本，功能可能变动。

## 端口

| 功能 | 端口 | 协议 | 必需 |
| --- | --- | --- | --- |
| 语音 | 9987 | UDP | 是 |
| 文件传输 | 30033 | TCP | 是 |

- 屏幕共享走 WebRTC P2P（客户端间直连），服务器**无需额外端口**。
- ServerQuery / WebQuery 等管理端口默认未开放，如需启用请参考官方文档。

## 快速开始

前置：安装 Docker 与 Docker Compose。

```bash
docker compose up -d
```

查看首次启动输出的管理员权限密钥（privilege key）与 serveradmin 凭据：

```bash
docker compose logs -f
```

首次用客户端连接服务器后，输入 privilege key 领取管理员权限。

## 防火墙

```bash
sudo ufw allow 9987/udp  comment 'TS6 Voice'
sudo ufw allow 30033/tcp comment 'TS6 File Transfer'
```

若使用云服务商，还需在控制台安全组放行 9987/UDP 与 30033/TCP。

## 更新

```bash
docker compose pull && docker compose up -d
```

## 参考

- 官方仓库：[teamspeak/teamspeak6-server](https://github.com/teamspeak/teamspeak6-server)
- 配置说明：[CONFIG.md](https://github.com/teamspeak/teamspeak6-server/blob/main/CONFIG.md)
