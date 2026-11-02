# Toonflow 定制版 · 一键部署

本仓库是 **HBAI-Ltd/Toonflow-app** 的 fork，包含以下定制（相对上游）：
- 移除首页顶部 banner 与侧栏推广/广告轮播及其加载逻辑。
- 移除账号面板的“充值”按钮与“钱包”页签及其组件挂载。
- 保留“探索、插件市场、账号登录”等非充值功能。
- 附带局域网纯 HTTP 下的 `crypto.randomUUID` 兼容补丁（不影响 HTTPS）。
- 通过 GitHub Actions 自动构建 Docker 镜像并推送到 GitHub Container Registry（ghcr.io）。

> 本定制保留上游 MIT 许可与版权声明。上游代码参见 https://github.com/HBAI-Ltd/Toonflow-app。

## 镜像

GitHub Actions 在推送 `master` 或手动触发时，会把镜像构建并推送到：

```
ghcr.io/qingwo1991-debug/toonflow-app:latest
```

镜像标签：`latest`（master）、`master`、`sha-<提交号>`。

> 注意：GitHub 容器仓库默认**私有**。若要在新机器免登录直接 `docker pull`，请先在网页端把该镜像设为公开：
> GitHub → 你的头像 → Packages → `toonflow-app` → Package settings → Danger Zone → Change visibility → **Public**。

## 在新机器上部署

### 1. 拉取镜像

```sh
docker pull ghcr.io/qingwo1991-debug/toonflow-app:latest
```

（若镜像未设为公开，需先 `docker login ghcr.io -u qingwo1991-debug`。）

### 2. 运行容器（推荐 docker compose）

新建目录并放入 `docker-compose.yml`：

```yaml
services:
  toonflow:
    image: ghcr.io/qingwo1991-debug/toonflow-app:latest
    container_name: toonflow
    restart: unless-stopped
    init: true
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - toonflowData:/app/data
    networks:
      - default

volumes:
  toonflowData:
```

启动：

```sh
docker compose up -d
```

访问：本机 `http://127.0.0.1:3000`（Docker 构建的镜像已内置 UUID 补丁与去广告改动）。

### 3. 局域网 / 远程访问（推荐带认证的反向代理）

业务页面与 API 默认无独立登录鉴权。局域网或公网使用请在反向代理（如 Nginx）层加 Basic Auth + HTTPS，并把 `3000` 只绑定到 `127.0.0.1`。参考本机现有 `/etc/nginx/conf.d/toonflow.conf`。

### 4. 数据迁移

配置、插件、模型 API Key、项目文件都在数据卷 `/app/data`。换机迁移时备份并完整复制该目录（或数据卷内容）到新机的同名卷。

- 不要执行 `docker compose down -v`（会删除数据卷）。
- 模型服务 API Key 需在新机页面内重新配置（或在迁移数据时一并带上）。

## 本机维护

```sh
# 构建最新 master 的镜像（云端完成，不占本地磁盘）
gh workflow run docker.yml --repo qingwo1991-debug/Toonflow-app

# 拉取新镜像并重建容器
docker compose pull && docker compose up -d
```