使用 [Xray](https://github.com/XTLS/Xray-install) 搭建节点。

```
bash <(curl -Ls https://raw.githubusercontent.com/XTLS/Xray-install/main/install-release.sh)

# 生成 UUID（用户ID）
xray uuid

# 生成 REALITY 密钥对
xray x25519
```

> vim /usr/local/etc/xray/config.json
```json
{
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "你的UUID",
            "flow": "xtls-rprx-vision"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "show": false,
          "dest": "www.google.com:443",
          "xver": 0,
          "serverNames": ["www.google.com"],
          "privateKey": "你的PrivateKey",
          "shortIds": [""]
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom"
    }
  ]
}
```

```sh
systemctl start xray

systemctl enable xray

systemctl status xray
```

Shadowrocket 配置
字段 | 内容
-- | --
类型 | VLESS
地址 | 你的服务器IP
端口 | 443
UUID | 你生成的UUID
Flow | xtls-rprx-vision
传输协议 | tcp
TLS | reality
SNI | www.google.com
Public Key | 你生成的PublicKey
Short ID | 留空


---

mac 上可以使用 [clash-party](https://github.com/mihomo-party-org/clash-party) 进行连接，

新建本地订阅，文件内容如下，
```yaml
mixed-port: 7890
allow-lan: false
mode: rule
log-level: warning

proxies:
  - name: "Xray REALITY"
    type: vless
    server: 你的服务器IP
    port: 443
    uuid: 你的UUID
    flow: xtls-rprx-vision
    tls: true
    udp: true
    network: tcp
    reality-opts:
      public-key: 你的PublicKey
      short-id: ""
    servername: www.google.com

proxy-groups:
  - name: "节点选择"
    type: select
    proxies:
      - "Xray REALITY"
      - DIRECT

rules:
  - GEOIP,CN,DIRECT
  - DOMAIN-SUFFIX,cn,DIRECT
  - MATCH,节点选择
```
