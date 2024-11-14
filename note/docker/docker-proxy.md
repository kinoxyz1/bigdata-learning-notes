
docker 
```bash
mkdir -p /etc/systemd/system/docker.service.d
tee /etc/systemd/system/docker.service.d/http-proxy.conf << EOF
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,127.0.0.1"
EOF

## 重载并重启服务
systemctl daemon-reload && systemctl restart docker
```

containerd
```bash
mkdir -p /etc/systemd/system/containerd.service.d/
tee /etc/systemd/system/containerd.service.d/http-proxy.conf <<EOF
[Service]
Environment="HTTP_PROXY=http://your_proxy_ip:your_proxy_port"
Environment="HTTPS_PROXY=http://your_proxy_ip:your_proxy_port"
EOF

## 重启生效
systemctl daemon-reload && systemctl restart containerd
```