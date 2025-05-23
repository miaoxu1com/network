以下是一个完整的 Clash YAML 配置文件，包含 **中文注释**、**国内外流量分流** 和 **DNS 分流域名解析** 规则：

```yaml
# ------------------------------
# 基础配置
# ------------------------------
port: 7890                  # HTTP/Socks5 代理端口
socks-port: 7891           
allow-lan: false           # 禁止局域网连接
mode: rule                 # 规则模式（根据规则分流）
log-level: info            # 日志级别（info/warning/error）
external-controller: 0.0.0.0:9090  # 远程控制端口


# ------------------------------
# 代理节点配置 (示例，需替换为实际订阅信息)
# ------------------------------
proxies:
  # 手动添加节点示例（或使用订阅链接）
  - name: "🇺🇸 美国节点"    # 节点名称
    type: ss               # 协议类型（ss/vmess/trojan等）
    server: us.example.com
    port: 443
    cipher: aes-256-gcm
    password: "your_password"


# ------------------------------
# 代理组配置
# ------------------------------
proxy-groups:
  # 自动选择延迟最低节点
  - name: "🔁 自动选择"
    type: url-test
    proxies:
      - "🇺🇸 美国节点"      # 填入实际节点名称
    url: "http://www.gstatic.com/generate_204"
    interval: 300          # 每 300 秒测速

  # 手动切换节点
  - name: "📲 手动选择"
    type: select
    proxies:
      - "🔁 自动选择"       # 包含自动选择组
      - "🇺🇸 美国节点"

  # 国内直连组
  - name: "🎯 国内直连"
    type: select
    proxies:
      - DIRECT             # 直连不代理


# ------------------------------
# 流量分流规则 (从上到下匹配)
# ------------------------------
rules:
  # 强制直连规则
  - DOMAIN-KEYWORD,cn,DIRECT         # 包含 "cn" 的域名直连
  - DOMAIN-SUFFIX,edu.cn,DIRECT      # 教育网域名直连
  - DOMAIN-SUFFIX,gov.cn,DIRECT      # 政府网站直连

  # 国内域名/IP直连
  - GEOIP,CN,🎯 国内直连              # 国内 IP 直连
  - MATCH,🔁 自动选择                # 未匹配的流量走代理

  # (可选) 流媒体规则示例
  # - DOMAIN-SUFFIX,netflix.com,📲 手动选择
  # - DOMAIN-SUFFIX,disneyplus.com,📲 手动选择


# ------------------------------
# DNS 配置 (分国内外解析)
# ------------------------------
dns:
  enable: true             # 启用 DNS 解析
  ipv6: false              # 禁用 IPv6 解析
  enhanced-mode: fake-ip  # 使用 Fake-IP 提高响应速度
  fake-ip-range: 198.18.0.1/16

  # 国内域名使用国内 DNS
  nameserver:
    - 114.114.114.114      # 国内公共 DNS
    - 223.5.5.5            # 阿里 DNS
    - tls://dns.pub:853    # 腾讯 DoT

  # 国外域名使用加密 DNS
  fallback:
    - https://1.1.1.1/dns-query  # Cloudflare DoH
    - https://dns.google/dns-query  # Google DoH
    - tls://dns.adguard.com:853  # AdGuard DoT

  # 域名分流规则
  fallback-filter:
    geoip: true            # 国外 IP 使用 fallback DNS
    ipcidr:
      - 0.0.0.0/8
      - 10.0.0.0/8
      - 127.0.0.0/8
      - 172.16.0.0/12
      - 192.168.0.0/16
    domain:
      - "+.cn"             # 国内域名用 nameserver
      - "+.qq.com"         # 指定域名直连
      - "+.weibo.com"


# ------------------------------
# TUN 模式配置 (增强模式)
# ------------------------------
tun:
  enable: true             # 启用 TUN 模式
  stack: system            # 使用系统协议栈（或 gvisor/mixed）
  dns-hijack: ["any:53"]   # 劫持所有 53 端口 DNS 查询
  auto-route: true         # 自动设置系统路由
  auto-detect-interface: true  # 自动选择网络接口
```

------

### **配置文件说明**

1. **流量分流逻辑**
   - 国内域名/GeoIP 直连 (`DIRECT`)
   - 国外流量自动选择最优节点 (`🔁 自动选择`)
   - 内置 `GEOIP,CN` 数据库识别国内 IP
2. **DNS 分流机制**
   - `.cn` 域名及国内常见域名使用 `114.114.114.114` 等国内 DNS 解析
   - 其他域名使用加密 DNS (`DoH/DoT`) 防止污染
   - Fake-IP 模式减少 DNS 解析延迟
3. **TUN 模式增强**
   - 劫持所有 DNS 请求防止泄漏
   - 自动路由国内外流量

------

### **使用前注意事项**

1. **节点替换**：将 `proxies` 部分替换为您的实际订阅节点

2. **规则更新**：建议定期更新 [GEOIP 数据库](https://github.com/Loyalsoldier/geoip)

   ```yaml
   yaml复制代码# 在配置开头添加 geoip 更新参数
   geoip-auto-update: true
   geoip-update-interval: 48
   ```

3. **平台适配**：

   - **Windows**：建议使用 Clash for Windows 并启用 TUN 模式
   - **macOS**：安装 [ClashX Pro](https://install.appcenter.ms/users/clashx/apps/clashx-pro/distribution_groups/public)
   - **Linux**：需手动配置 TUN 设备权限

4. **分流验证**：

   - 访问 [IP.SB](https://ip.sb/) 确认国外 IP
   - 访问 [百度](https://baidu.com/) 确认直连无代理

------

### **扩展优化建议**

```yaml
yaml复制代码# 在 rules 部分添加常用国内服务直连
rules:
  # 国内常见服务直连
  - DOMAIN-SUFFIX,taobao.com,DIRECT
  - DOMAIN-SUFFIX,jd.com,DIRECT
  - DOMAIN-SUFFIX,weibo.com,DIRECT
  - DOMAIN-SUFFIX,zhihu.com,DIRECT

  # 广告屏蔽规则 (示例)
  - DOMAIN-KEYWORD,adservice,REJECT
  - DOMAIN-SUFFIX,doubleclick.net,REJECT
```

可根据需要自行调整规则顺序和 DNS 服务器地址，建议搭配 [Clash 规则生成器](https://acl4ssr-sub.github.io/) 生成更复杂规则。
