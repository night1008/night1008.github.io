使用 [Hysteria2](https://github.com/apernet/hysteria) 搭建节点。

```sh
sudo su

bash <(curl -fsSL https://get.hy2.sh/)

# 生成证书
openssl req -x509 -nodes -newkey ec -pkeyopt ec_paramgen_curve:P-256 \
  -keyout /etc/hysteria/server.key \
  -out /etc/hysteria/server.crt \
  -subj "/CN=bing.com" -days 36500

# 修复权限
chmod 644 /etc/hysteria/server.key
chmod 644 /etc/hysteria/server.crt
```

> vim /etc/hysteria/config.yaml
```yaml
listen: :443

# 自签证书（最简单）
tls:
  cert: /etc/hysteria/server.crt
  key: /etc/hysteria/server.key

auth:
  type: password
  password: 你的密码123  # 改成你自己的密码

masquerade:
  type: proxy
  proxy:
    url: https://bing.com  # 伪装成 Bing
    rewriteHost: true
```

```
systemctl start hysteria-server

systemctl enable hysteria-server

systemctl status hysteria-server
```

在 Shadowrocket 上配置连接

字段 | 应该填写
-- | --
类型 | Hysteria2（不是 Hysteria）
地址 | 47.251.901.901
端口 | 443
密码 | 和 config.yaml 完全一致
SNI | bing.com
跳过证书验证 | ✅ 必须开启

测试下来在相同美国节点上速度会比 Shadowsocks 快快一倍。

---

mac 上可以使用 [clash-party](https://github.com/mihomo-party-org/clash-party) 进行连接，

新建本地订阅，文件内容如下，
```yaml 
mixed-port: 7890
allow-lan: false
mode: rule
log-level: info

proxies:
  - name: "我的Hysteria2"
    type: hysteria2
    server: 47.251.901.901    # 你的服务器IP
    port: 443
    password: 你的密码
    sni: bing.com
    skip-cert-verify: true

proxy-groups:
  - name: "节点选择"
    type: select
    proxies:
      - "我的Hysteria2"
      - DIRECT

rules:
  - GEOIP,CN,DIRECT
  - MATCH,节点选择
```