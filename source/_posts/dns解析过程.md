---
title: dns解析过程
date: 2023-12-23 23:30:18
tags:
---

<!--more-->

测试一下

<!--more-->

dns解析

​	1.首先查看浏览器缓存。缓存有时间会过期，几分钟或几小时

​	2.查看本地的c.:/windows/system32/drivers/etc/hosts

​						/etc/hosts

​	3.本机没有就会向本机配置的本地dns域名服务器（ldns）发起请求。如果是学校就是学校dns父亲，如果是小区连接互联网就是互联网提供商，联通电信的dns服务器。linux可以查看，/etc/resolv.conf查看

4.ldns不能解析，就直接到跟域名服务器。

5.根域名服务器会给本地域名服务器ldns一个所查询的主域名服务器gtld地址。gltd是国际顶级域名服务器，com,cn,org

6.然后ldns在想上一步返回域名gltd发送请求。

7.gtld返回域名对应的name server域名。然后注册域名服务商解析该域名

8.name server域名服务器会查询域名与ip关系表。将ip和ttl值返回dnsserver

9.ldns拿到ip与ttl会缓存。缓存有ttl控制

10.把解析结果返回给用户。用户更具ttl之在本地缓存种。
