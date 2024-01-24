# dns 解析的流程

## dns 解析的类型

- A:将域名指向一个 IPv4 地址，如 106.55.75.123。 A 记录归属地是埃文科技的付费接口，为了防止盗刷，每个用户有查询限制，请节省使用。

- AAAA: 将域名指向一个 IPv6 地址，如 2402:4e00:1013:e500:0:9671:f018:4947。同一个主机名可以同时解析到 IPv4(A 记录)地址 和 IPv6(AAAA 记录)地址上，当只有 IPv4 地址的用户会解析到 IPv4 地址，一般情况下有 IPv6 地址的用户会优先解析到 IPv6 地址。

- CNAME: 将域名指向另一个域名地址，与其保持相同解析，如 static.ipw.cn 别名到 static.ipw.cn.cdn.dnsv1.com

- MX: 用于邮件服务器，一般由邮件注册商提供，如 mxbiz1.qq.com。如果邮箱格式为 <test@ipw.cn> 则输入 ipw.cn 查询。如果邮箱格式为 <test@mail.ipw.cn>则输入 mail.ipw.cn 查询。推荐 2 个免费的企业邮箱：腾讯企业邮、网易> 免费企业邮。

- TXT: 附加文本信息，常用于域名所有权验证，如在申请 HTTPS 证书时需要增加记录

- PTR: IP 的反向解析记录，例如 159.75.190.197 反解析到 ipw.cn，一般用于提升自建域名邮件服务器的可信度，可提单找云服务商添加。

- NS: 域名的 DNS 服务器地址，例如 ns3.dnsv2.com，推荐 DNSPod.

<br>

**DNS 一般分为`本地 DNS`（一般由电信运营商提供） 和`权威 DNS`（ 由根 DNS、顶级 DNS、二级 DNS 组成）**

## 本地 DNS

本地 DNS 采用递归查询域名解析记录，比如指定一个电信 DNS（202.101.224.69）来解析 ipw.cn，在 ANSWER SECTION 中可以看到直接返回了 ipw.cn 的 A 记录。

本地 DNS 本身不会记录域名解析记录，而是把请求转发到权威 DNS，逐级递归查询，再把解析记录返回给客户端。当然它会缓存解析记录（解析记录中 TTL 属性控制缓存的时长。）

本地 DNS 解析示例：

```shell
~$ dig @202.101.224.69 ipw.cn
;; QUESTION SECTION:
;ipw.cn.				IN	A

;; ANSWER SECTION:
ipw.cn.			600	IN	A	106.55.75.123
```

## 权威 DNS

以查询`ipw.cn` 为例，了解下这 3 级 DNS 的作用。

- 根 DNS：比如 a.root-servers.net. , 负责返回待查询域名（ipw.cn) 所属顶级 DNS（如 a.dns.cn.）的解析记录 203.119.25.1；
- 顶级 DNS： 比如 a.dns.cn. ，负责返回待查询域名的 DNS 地址（如 ns3.dnsv2.com.，登记在顶级域名服务器中，通过 whois 命令可以查询，在域名管理平台中可以修改）的解析记录
- 一级 DNS：比如 ns3.dnsv2.com.， 负责返回具体域名的解析记录，比如 ipw.cn 的 A 记录为 106.55.75.123

以下为查询示例：

查看根 DNS 的 A 记录

```shell
$ dig a.root-servers.net
;; QUESTION SECTION:
;a.root-servers.net.		IN	A

;; ANSWER SECTION:
a.root-servers.net.	81077	IN	A	198.41.0.4

;; AUTHORITY SECTION:
root-servers.net.	125280	IN	NS	h.root-servers.net.
root-servers.net.	125280	IN	NS	c.root-servers.net.
root-servers.net.	125280	IN	NS	a.root-servers.net.
root-servers.net.	125280	IN	NS	e.root-servers.net.
root-servers.net.	125280	IN	NS	k.root-servers.net.
root-servers.net.	125280	IN	NS	b.root-servers.net.
root-servers.net.	125280	IN	NS	i.root-servers.net.
root-servers.net.	125280	IN	NS	d.root-servers.net.
root-servers.net.	125280	IN	NS	l.root-servers.net.
root-servers.net.	125280	IN	NS	g.root-servers.net.
root-servers.net.	125280	IN	NS	j.root-servers.net.
root-servers.net.	125280	IN	NS	f.root-servers.net.
root-servers.net.	125280	IN	NS	m.root-servers.net.

;; ADDITIONAL SECTION:
h.root-servers.net.	517037	IN	A	198.97.190.53
c.root-servers.net.	563686	IN	A	192.33.4.12
e.root-servers.net.	601355	IN	A	192.203.230.10
g.root-servers.net.	472031	IN	A	192.112.36.4
f.root-servers.net.	553894	IN	A	192.5.5.241
k.root-servers.net.	597325	IN	A	193.0.14.129
i.root-servers.net.	517037	IN	A	192.36.148.17
l.root-servers.net.	565009	IN	A	199.7.83.42
b.root-servers.net.	559683	IN	A	199.9.14.201
d.root-servers.net.	601468	IN	A	199.7.91.13
m.root-servers.net.	603681	IN	A	202.12.27.33
j.root-servers.net.	584819	IN	A	192.58.128.30
h.root-servers.net.	569462	IN	AAAA	2001:500:1::53
c.root-servers.net.	34585	IN	AAAA	2001:500:2::c
```

通过根 DNS 查询 ipw.cn ，返回了顶级 DNS a.dns.cn. 的地址为 203.119.25.1

```shell
~$ dig @198.41.0.4 ipw.cn
;; QUESTION SECTION:
;ipw.cn.				IN	A

;; AUTHORITY SECTION:
cn.			172800	IN	NS	a.dns.cn.
cn.			172800	IN	NS	b.dns.cn.
cn.			172800	IN	NS	c.dns.cn.
cn.			172800	IN	NS	d.dns.cn.
cn.			172800	IN	NS	e.dns.cn.
cn.			172800	IN	NS	f.dns.cn.
cn.			172800	IN	NS	g.dns.cn.
cn.			172800	IN	NS	ns.cernet.net.

;; ADDITIONAL SECTION:
a.dns.cn.		172800	IN	A	203.119.25.1
b.dns.cn.		172800	IN	A	203.119.26.1
c.dns.cn.		172800	IN	A	203.119.27.1
d.dns.cn.		172800	IN	A	203.119.28.1
e.dns.cn.		172800	IN	A	203.119.29.1
f.dns.cn.		172800	IN	A	195.219.8.90
g.dns.cn.		172800	IN	A	66.198.183.65
ns.cernet.net.		172800	IN	A	202.112.0.44
a.dns.cn.		172800	IN	AAAA	2001:dc7::1
d.dns.cn.		172800	IN	AAAA	2001:dc7:1000::1
```

通过 a.dns.cn 查询 ipw.cn，返回 一级 DNS 的 A 记录

```shell
~$ dig @a.dns.cn ipw.cn

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;ipw.cn.				IN	A

;; AUTHORITY SECTION:
ipw.cn.			86400	IN	NS	ns4.dnsv2.com.
ipw.cn.			86400	IN	NS	ns3.dnsv2.com.
```

通过 ns4.dnsv2.com 返回 ipw.cn 的 A 记录 106.55.75.123

```shell
~$ dig @ns4.dnsv2.com ipw.cn
;; QUESTION SECTION:
;ipw.cn.				IN	A

;; ANSWER SECTION:
ipw.cn.			600	IN	A	106.55.75.123

;; AUTHORITY SECTION:
ipw.cn.			86400	IN	NS	ns4.dnsv2.com.
ipw.cn.			86400	IN	NS	ns3.dnsv2.com.
```

#### 参考文档

- [ipw.cn](https://ipw.cn/dns/)
