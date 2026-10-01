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

## 域名解析（可选）

如果想用域名代替 IP 连接服务器，在 DNS 服务商（如 Cloudflare）添加一条 A 记录：

| 类型 | 名称 | 内容 | 代理状态 |
| --- | --- | --- | --- |
| A | `ts` | `<你的服务器 IP>` | **仅 DNS（灰色云）** |

以 Cloudflare 为例，代理状态必须选择 **DNS only（灰色云）**，而非 Proxied（橙色云）：

- TeamSpeak 语音走 UDP 9987、文件传输走 TCP 30033，均非 HTTP(S) 流量。
- Cloudflare 免费/Pro 版的橙色云代理仅支持 HTTP/HTTPS，套在 TS 端口上会导致无法连接。
- 灰色云只做域名解析，流量直连服务器，语音延迟最低。

> 注意：灰色云不隐藏源站 IP，`dig` 或客户端日志仍可看到真实 IP。若需真正隐藏 IP，需另置中转服务器。

添加后，客户端连接地址填写该域名即可（端口保持 9987 默认）。

## 更新

```bash
docker compose pull && docker compose up -d
```

## 重要：Beta 许可每 2 个月过期

TeamSpeak 6 服务端内置一个 32 槽位的 **Beta 默认许可**，该许可**内置于服务端版本中**，有效期约 2 个月。许可过期后，服务端会拒绝启动并不断重启（`Restarting`），日志中出现：

```
CRITICAL|Accounting | The default license has expired. Please use the latest server version.
```

此时**必须更新到最新版本**才能恢复（旧版本的许可无法续期，也不能购买更大的许可）：

```bash
docker compose pull && docker compose up -d
```

> 注意：`restart: unless-stopped` 只会重启同一个旧容器，**不会自动拉取新镜像**。因此许可过期后，容器会陷入无限重启（每 60 秒重试一次），需手动执行上面的更新命令。数据保存在 `teamspeak-data` 卷中，更新不会丢失配置。

可随时在日志中确认当前许可的有效期：

```bash
docker compose logs | grep -E 'starting date|ending date|max slots'
```

## 参考

- 官方仓库：[teamspeak/teamspeak6-server](https://github.com/teamspeak/teamspeak6-server)
- 配置说明：[CONFIG.md](https://github.com/teamspeak/teamspeak6-server/blob/main/CONFIG.md)
