### 使用 new-api

```sh
git clone https://github.com/QuantumNous/new-api.git

cd new-api

# 改一下数据库和换成的密码
vim docker-compose.yml

docker compose up -d
```

### 使用 Cloudflare Tunnel，全程免费，不需要证书
```
# 服务器上安装 cloudflared
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o /usr/local/bin/cloudflared
chmod +x /usr/local/bin/cloudflared

# 登录
cloudflared tunnel login

# 创建 tunnel
cloudflared tunnel create new-api

# 配置指向本地 3000 端口
cloudflared tunnel route dns new-api api.yourdomain.com

# 启动
cloudflared tunnel run --url http://localhost:3000 new-api
```

### 后台跑 Cloudflare Tunnel
```sh
cloudflared tunnel route dns new-api api.yourdomain.com

# 安装系统服务
cloudflared service install

# 启动
systemctl start cloudflared

systemctl status cloudflared
```