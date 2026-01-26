# ip_derper

> 无域名部署 Tailscale Derper

## 部署

复制下面配置写入到 `docker-compose.yaml`，运行 `docker compose up -d`

```yaml
services:
  derper:
    image: ghcr.io/flxxyz/ip_derper:latest # 官方镜像，仅移除"必须域名"代码
    #image: ghcr.nju.edu.cn/flxxyz/ip_derper:latest # 国内加速镜像
    restart: always
    ports:
      - 3478:3478/udp
      - 33333:33333
      - 3443:3443
    environment:
      - DERP_HOST=1.1.1.1 # 你的VPS IP
      - DERP_ADDR=:33333
      - DERP_HTTP_PORT=3443
      - DERP_VERIFY_CLIENTS=true # 如果不验证客户端，可以把 volumes 也注释掉
    volumes:
      - /var/run/tailscale/tailscaled.sock:/var/run/tailscale/tailscaled.sock
```

> 优先考虑在宿主机安装 **tailscale**，开启 `DERP_VERIFY_CLIENTS=true`
> 
> 如不安装**tailscale**，挂载 `volumes` 是**无效行为**

## Access Consoles

在 **Tailscale Admin Console** 修改，添加 **derpMap** 代码块

```json
{
	// ...
	"derpMap": {
		"OmitDefaultRegions": true, // 允许使用官方提供的derper服务器
		"Regions": {
			"900": {
				"RegionID":   900,
				"RegionCode": "CN-SH",          // 平时出现的简化标识
				"RegionName": "China ShangHai", // 在netcheck报告出现的显示名称，可以参考下面示例
				"Nodes": [
					{
						"Name":             "ip-127.0.0.1",
						"RegionID":         900,
						"IPv4":             "你的第一台服务器IP",
						"HostName":         "你的第一台服务器IP",
						"DERPPort":         33333,
						"STUNPort":         3478,
						"InsecureForTests": true
					},
					// 同区域存在多个derper，直接复制修改 "Name"、"IPv4"、"HostName"
					{
						"Name":             "ip-127.0.0.2",
						"RegionID":         900,
						"IPv4":             "你的第二台服务器IP",
						"HostName":         "你的第二台服务器IP",
						"DERPPort":         33333,
						"STUNPort":         3478,
						"InsecureForTests": true
					}
				]
			}
		}
	}
}
```

### netcheck示例

```bash
[~]$ tailscale netcheck
Report:
	* Time: 2025-11-20T03:18:37.792087Z
	* UDP: true
	* IPv4: yes, 1.1.1.1:34890
	* IPv6: no, but OS has support
	* MappingVariesByDestIP: true
	* PortMapping:
	* CaptivePortal: false
	* Nearest DERP: ShangHai - Mainland China
	* DERP latency:
		- CN-SH: 10.7ms  (ShangHai - Mainland China)
```

