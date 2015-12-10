date: 2015-5-14
title: PPTP VPN搭建
tags: PPTP, vpn
category: system
---

本文介绍在Linux下搭建VPN的过程
```bash
#!/bin/sh
if [ `id -u` -ne 0 ] 
then
  echo "please run it by root"
  exit 0
fi

apt-get -y update

apt-get -y install pptpd || {
  echo "could not install pptpd" 
  exit 1
}

cat >/etc/ppp/options.pptpd <<END
name pptpd
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128
ms-dns 8.8.8.8
ms-dns 8.8.4.4
proxyarp
lock
nobsdcomp 
novj
novjccomp
nologfd
END

cat >/etc/pptpd.conf <<END
option /etc/ppp/options.pptpd
logwtmp
localip 192.168.2.1
remoteip 192.168.2.10-100
END

cat >> /etc/sysctl.conf <<END
net.ipv4.ip_forward=1
END

sysctl -p

iptables-save > /etc/iptables.down.rules

iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -o eth0 -j MASQUERADE

iptables -I FORWARD -s 192.168.2.0/24 -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1300

iptables-save > /etc/iptables.up.rules

cat >>/etc/ppp/pptpd-options<<EOF
pre-up iptables-restore < /etc/iptables.up.rules
post-down iptables-restore < /etc/iptables.down.rules
EOF

cat >/etc/ppp/chap-secrets <<END
test pptpd test *
END

service pptpd restart

netstat -lntp

exit 0
```
实际使用中需要按照需要修改本地IP地址即可，一般情况自动安装和配置是成功的，但是在我的实际使用中使用手机网络连接的时候是没问题的，但是使用电脑连接的时候还是出现无法连接的情况，查看日志如下：

Starting negotiation on /dev/pts/1
sent [LCP ConfReq id=0x1 <asyncmap 0x0> <auth chap MS-v2> <magic 0xc93c30c6> <pcomp> <accomp> <mrru 1500> <endpoint [MAC:00:16:3e:d8:d7:39]>]
sent [LCP ConfReq id=0x1 <asyncmap 0x0> <auth chap MS-v2> <magic 0xc93c30c6> <pcomp> <accomp> <mrru 1500>
...
LCP: timeout sending Config-Requests
Connection terminated.
这个可能是学校路由器没有采用PPTP穿透吧