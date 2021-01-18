因为学校是按照流量收费的,但是校内的资源不消耗流量,而电脑插上网线后访问外网是不消耗流量的,所以可以将电脑作为代理服务器,所有流量都走校内的电脑保证上网不耗流量。

# ssr配置

首先准备好服务器，配好静态ip或者不关机，并查看服务器的ipv6地址，因为路由器设置了nat转换，需要查看ipv6地址来进行服务器连接。

```
#执行以下命令，最好用root权限执行
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh

chmod +x shadowsocksR.sh

./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```

中间可能出现无法连接raw.githubusercontent.com的情况，这是由于dns无法解析的问题。登录www.ipaddress.com查询raw.githubusercontent.com的实际ip地址，并加入`/etc/hosts`文件中，我查到的是199.232.96.133.

```
sudo vim /etc/hosts
#并加入这一行
199.232.96.133 raw.githubusercontent.com
```

执行完成后按照提示要求设置ssr的配置。配置文件保存在`/etc/shadowsocks.json`文件中，可以通过如下命令更改ssr的状态.

```
/etc/init.d/shadowsocks  start|stop|restart|status
```

至此手机端和电脑端的网页可以使用ssr代理，服务器地址使用ipv6地址。

# BBR加速

这时候上网的速度只能说还算可以，但是看直播还是有些卡顿的，需要对服务器的网络进行加速保证能够正常看直播。

BBR需要linux内核达到4.10以上，查看命令`uname -r`，这里是ubuntu 20.04所以不需要担心内核的版本。

开启bbr

```
echo 'net.core.default_qdisc=fq' >> /etc/sysctl.conf 
echo 'net.ipv4.tcp_congestion_control=bbr' >> /etc/sysctl.conf  
sysctl -p
```

安装bbr脚本，可能出现文件无法执行的情况，执行`chmod +x bbr.sh`

```
curl -O https://raw.githubusercontent.com/teddysun/across/master/bbr.sh && sh bbr.sh
```

查看可以使用的拥塞控制算法

```
sysctl net.ipv4.tcp_available_congestion_control
```

查看现在正在使用的拥塞控制算法

```
sysctl net.ipv4.tcp_congestion_control
```

检查bbr是否正常运行

```
lsmod | grep tcp_bbr
```

# 电脑端全局代理

需要一个软件使电脑端能够全局代理socks 5，来使电脑的流量全部都走代理。使用软件Proxifier（sstap不支持ipv6），官网直接下载的是Standard版本的，输入激活码`5EZ8G-C3WL5-B56YG-SCXM9-6QZAP`

之后需要三个步骤

1. **Proxy Server**

   ssr通常使用1080端口，ip为本地的环回ip，协议是socks 5

   ![image](https://github.com/KuangThreeSAMA/-/blob/main/images/ProxyServer.png)

   

2. **Proxification Rules**

   代理规则需要设置两步，首先在localhost里面的target hosts加入服务器ip地址，并且动作为direct，保证到服务器的流量是直接走的，不需要经过代理，防止循环代理。

   ![image](https://github.com/KuangThreeSAMA/-/blob/main/images/ProxificationRules1.png)

   之后需要新建一个代理规则，放在localhost后面，保证所有应用的流量都是经过ssr，动作为socks 5代理。

   ![image](https://github.com/KuangThreeSAMA/-/blob/main/images/ProxificationRules2.png)

3. **Name Resolution**

   dns让服务器进行识别。

   ![image](https://github.com/KuangThreeSAMA/-/blob/main/images/NameResolution.png)

至此电脑的所有流量也可以经过ssr代理，上网不花费一丁点流量。
