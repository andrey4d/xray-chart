## XRAY-CORE helm chart

### K3s Lightweight Kubernetes
```url
https://k3s.io/
```
### Create package
```shell
helm package xray-core 
```

### Using the K3s Helm Controller
Content placed in /var/lib/rancher/k3s/server/static/ can be accessed anonymously via the Kubernetes APIServer from within the cluster. 
```shell
sudo cp xray-core-0.1.0.tgz /var/lib/rancher/k3s/server/static/charts
```
### K3s HelmChart example
```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: xray-vision
---
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: xray-vision
  namespace: kube-system
spec:
  chart: https://%{KUBERNETES_API}%/static/charts/xray-core-0.1.0.tgz
  targetNamespace: xray-vision
  valuesContent: |-
    image:
      repository: registry.example.com/xray-core
      tag: "v1.8.23"
    ingressRouteTCP:
      priority: 10
      match: HostSNIRegexp(`{subdomain:^[w]{3}}.example.com`)
    tlsSecret:
      annotations: 
        replicator.v1.mittwald.de/replicate-from: cert-manager/tls-www-example-com
      key: ""
      crt: ""      
    configJson: |
      {
        "log": {
            "loglevel": "warning"
        },
        "routing": {
            "domainStrategy": "IPIfNonMatch",
            "rules": [
                {
                    "type": "field",
                    "ip": [
                        "geoip:cn"
                    ],
                    "outboundTag": "block"
                }
            ]
        },
        "inbounds": [
            {
                "listen": "0.0.0.0",
                "port": 443,
                "protocol": "vless",
                "settings": {
                    "clients": [
                        {
                            "id": "",
                            "flow": "xtls-rprx-vision"
                        }
                    ],
                    "decryption": "none",
                    "fallbacks": [
                        {
                            "dest": "8001",
                            "xver": 1
                        },
                        {
                            "alpn": "h2",
                            "dest": "8002",
                            "xver": 1
                        }
                    ]
                },
                "streamSettings": {
                    "network": "tcp",
                    "security": "tls",
                    "tlsSettings": {
                        "rejectUnknownSni": true,
                        "minVersion": "1.2",
                        "certificates": [
                            {
                                "ocspStapling": 3600,
                                "certificateFile": "/etc/letsencrypt/certs/tls.crt",
                                "keyFile": "/etc/letsencrypt/private/tls.key"
                            }
                        ]
                    }
                },
                "sniffing": {
                    "enabled": true,
                    "destOverride": [
                        "http",
                        "tls"
                    ]
                }
            }
        ],
        "outbounds": [
            {
                "protocol": "freedom",
                "tag": "direct"
            },
            {
                "protocol": "blackhole",
                "tag": "block"
            }
        ],
        "policy": {
            "levels": {
                "0": {
                    "handshake": 2,
                    "connIdle": 120
                }
            }
        }
      }
```  


## Dependency 
### traefik ingress controller 
```url
https://traefik.io/traefik/
```
