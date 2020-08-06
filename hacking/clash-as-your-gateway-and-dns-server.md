# Clash 作为网关

家里网络下有不少平时不怎么用的设备，老旧的手机、iPad 和运行在 Proxomox 里面的 Windows 机器，因为有科学上网的需求但是又不想每个设备都安装代理应用和维护配置文件，用 HTTP 代理的方式在 Android 系统下是不能访问 Google Play Store 的，所以萌生了用 Clash 作为网关统一为局域网内的设备提供科学上网的能力。

对比过几个方案下来觉得 Clash 最合适，因为我平时在 Mac 下就一直在用它，规则配置也都熟悉，支持的协议也很多。刚开始我是在 Proxmox 下安装了一个定制的 OpenWrt，用它自带的 OpenClash 来上网的，这个模式对我来说有几个缺点：

1. 定制的 OpenWrt 包含了过多的无用的功能，我只需要一个 Clash 网关的功能，但是里面集成了一大堆对我来说无用的功能。
2. OpenClash web UI 的各种选项和设置看着太繁琐，不如直接编辑 Clash 配置文件来的简单方便。
3. 不知道为什么，虚拟机里的 OpenWrt 稳定性不行， 隔一段时间系统就会挂掉，我曾想过用 Proxmox 的 API 定期检查下虚拟机状态，停止的时候自动再把它拉起来，后来感觉索性就不要在 OpenWrt 上继续折腾了。

关于 Clash 网关的教程网上也有很多，我趁这个机会顺便看了下 Clash 的实现方式和不同模式（fake-ip 和 redir-host）的不同之处，也学到了不少新的东西。我的需求很简单，就是需要一个 Clash 的网关，如果局域网内的设备想要科学上网的话去网络里面更改下网关和 DNS 就好了。下面开始记录下我的操作记录：

软件环境：

- Debian 10
- Clash v1.0.0
- Supervisord
- Dnsmasq
- Iptables

虚拟机系统我用的 Debian，别的系统也是可以的，Debian 占用的资源相对较少而且我也熟悉。

这是我的 Clash 配置文件的一段配置：

```
# Full configurations https://github.com/Dreamacro/clash/wiki/configuration#all-configuration-options
port: 7890
socks-port: 7891
redir-port: 7892
allow-lan: true
bind-address: "*"
mode: Rule
log-level: debug
external-controller: 0.0.0.0:9090
secret: "xxx-xxx"
dns:
  enable: true
  listen: 0.0.0.0:5353
  ipv6: true
  enhanced-mode: redir-host
  nameserver:
    - 223.5.5.5
    - 223.6.6.6
  fallback:
    - tls://13800000000.rubyfish.cn:853
    - tls://1.0.0.1:853
    - tls://dns.google:853

fallback-filter:
  geoip: true
  ipcidr:
    - 240.0.0.0/4

proxies:
```

具体的解释可以参考[官方 wiki](https://github.com/Dreamacro/clash/wiki/configuration#all-configuration-options)，Clash 解决 DNS 污染的策略官方 wiki 也有详细的解释。简单来说就是 Clash 在 DNS 开启的情况下请求域名的时候会同时向 `nameserver` 和 `fallback` 里面指定的 DNS 服务器同时发送查询请求，如果 `nameserver` 返回的 IP 地址不是国内的地址 Clash 会使用 `fallback` 返回的 DNS 结果。

这里我选用了 redir-host 模式，另外一个是 fake-ip 模式，两者的优缺点可以参考这个文章：[代替 Surge 增强模式——使用 KoolClash 作为代理网关](https://blog.skk.moe/post/alternate-surge-koolclash-as-gateway)。

Clash 配置好后我们将 Clash 进程用 Supervisord 来进行托管，配置文件如下：

```
[program:clash]
command = /home/clash/.bin/clash
directory = /home/clash/
autostart = True
autorestart = True
user = clash
stderr_logfile = /var/log/supervisor/clash-stderr.log
stdout_logfile = /var/log/supervisor/clash-stdout.log
```

我们用 Dnsmasq 来搭建 DNS 服务器，其实目的就是把 53 端口的 DNS 请求转发到 Clash 监听的 5353  端口上，我看网上也有人用 root 来运行 Clash 并将端口指向 53 的，为了规范一些我没有这么做，不过这么做问题也不大，看自己选择了。Dnsmasq 配置如下：

```jsx
# https://computingforgeeks.com/install-and-configure-dnsmasq-on-ubuntu-18-04-lts/
port=53
domain-needed
bogus-priv
strict-order
expand-hosts
server=127.0.0.1#5353
```

最后一步就是配置 iptables 了，规则也非常简单：

```jsx
# Create CLASH chain https://cherysunzhang.com/2020/05/deploy-clash-as-transparent-proxy-on-raspberry-pi/
iptables -t nat -N CLASH

# Bypass private IP address ranges
iptables -t nat -A CLASH -d 10.0.0.0/8 -j RETURN
iptables -t nat -A CLASH -d 127.0.0.0/8 -j RETURN
iptables -t nat -A CLASH -d 169.254.0.0/16 -j RETURN
iptables -t nat -A CLASH -d 172.16.0.0/12 -j RETURN
iptables -t nat -A CLASH -d 192.168.0.0/16 -j RETURN
iptables -t nat -A CLASH -d 224.0.0.0/4 -j RETURN
iptables -t nat -A CLASH -d 240.0.0.0/4 -j RETURN

# Redirect all TCP traffic to 7892 port, where Clash listens
iptables -t nat -A CLASH -p tcp -j REDIRECT --to-ports 7892
iptables -t nat -A PREROUTING -p tcp -j CLASH
```

因为服务器每次重启后 iptables 的规则会被 reset 掉，这里我们用 iptables-persistent 来将 iptables 规则做持久化处理。

上面的简单几步操作完成后将局域网内的设备的网关和 DNS 设置为 Clash 网关的 IP 地址即可享受无障碍上网了。

## 参考链接

这篇总结的完成主要参考了很多网上现有的教程，自己再东拼西凑组成的，非常感谢下面的文章：

- [Clash 透明代理](https://blog.mrsheep.xyz/posts/clash-gateway/)
- [x86 软路由透明代理构建方案v2020.02](https://0x01.io/2020/02/16/x86-%E8%BD%AF%E8%B7%AF%E7%94%B1%E9%80%8F%E6%98%8E%E4%BB%A3%E7%90%86%E6%9E%84%E5%BB%BA%E6%96%B9%E6%A1%88v2020-02/)
- [在 Raspberry Pi 上运行 Clash 作为透明代理](https://cherysunzhang.com/2020/05/deploy-clash-as-transparent-proxy-on-raspberry-pi/)
- [使用 clash 和路由表实现透明代理](https://h-cheung.gitlab.io/post/%E4%BD%BF%E7%94%A8_clash_%E5%92%8C%E8%B7%AF%E7%94%B1%E8%A1%A8%E5%AE%9E%E7%8E%B0%E9%80%8F%E6%98%8E%E4%BB%A3%E7%90%86/)
- [DNS污染对Clash（for Windows）的影响](https://github.com/Fndroid/clash_for_windows_pkg/wiki/DNS%E6%B1%A1%E6%9F%93%E5%AF%B9Clash%EF%BC%88for-Windows%EF%BC%89%E7%9A%84%E5%BD%B1%E5%93%8D)
