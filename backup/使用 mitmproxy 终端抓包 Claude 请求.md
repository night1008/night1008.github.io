```sh
brew install mitmproxy

mitmproxy --listen-port 8080
mitmproxy --listen-port 8080 --mode upstream:http://127.0.0.1:1087 (通过代理)

export HTTPS_PROXY=http://127.0.0.1:8080
export HTTP_PROXY=http://127.0.0.1:8080
claude "hi"
```