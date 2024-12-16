---
date:
  created: 2024-11-17
tags:
  - Router
authors: [Tenax]
description: >
  Xiaomi AX3000T Brush with ImmortalWrt and U-Boot
---

# 小米 AX3000T 刷机 ImmortalWrt 和 U-Boot

<!-- more -->

## 启用 SSH 权限

用网线连接你的路由器进入小米路由器后台，全新的路由器需要先配置设置，配置完后用后台地址中`stok=`后面的一串替换下面代码中的{}

```bash
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok={替换成你的stok}/api/xqsystem/start_binding -d "uid=1234&key=1234'%0Anvram%20set%20ssh_en%3D1'"
```

```bash
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok={替换成你的stok}/api/xqsystem/start_binding -d "uid=1234&key=1234'%0Anvram%20commit'"
```

```bash
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok={替换成你的stok}/api/xqsystem/start_binding -d "uid=1234&key=1234'%0Ased%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%22debug%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear'"
```

```bash
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok={替换成你的stok}/api/xqsystem/start_binding -d "uid=1234&key=1234'%0A%2Fetc%2Finit.d%2Fdropbear%20start'"
```

```bash
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok={替换成你的stok}/api/xqsystem/start_binding -d "uid=1234&key=1234'%0Apasswd%20-d%20root"'
```

## SSH 登录

使用 WinSCP 连接路由器（WinSCP 图形化界面更适合小白）

![1](../../assets/router/1.avif)

开启 SSH 后会生成一些原厂分区信息

![2](../../assets/router/2.avif)

## 刷入

将`mt7981_ax3000t-fip-fixed-parts-multi-layout.bin`放在 tmp 文件夹里

```bash
mtd write /tmp/mt7981_ax3000t-fip-fixed-parts-multi-layout.bin FIP
```

然后断开电源，用牙签或卡针捅住 reset 键，然后接通电源，指示灯变化后松开 reset 键。用网线将电脑和路由器连接起来，电脑网卡配置静态 IP：`192.168.1.100` ，网关: `192.168.1.1`

在浏览器中访问 [http://192.168.1.1](http://192.168.1.1/) 进入 U-Boot 的网页刷机界面，此处抛弃小米原有的 stock layout，直接选用 immortalwrt-112m layout，然后上传固件确认升级即可。

![3](../../assets/router/3.avif)

![4](../../assets/router/4.avif)

![5](../../assets/router/5.avif)

![6](../../assets/router/6.avif)

![7](../../assets/router/7.avif)

![8](../../assets/router/8.avif)

## ImmortalWrt 固件/U-Boot 下载下载

uboot 上传固件用的是`immortalwrt-mediatek-mt7981-xiaomi_mi-router-ax3000t-squashfs-factory.bin`

固件后台内更新用的是`immortalwrt-mediatek-mt7981-xiaomi_mi-router-ax3000t-squashfs-sysupgrade.bin`

下载地址：[xiaomi-ax3000t-immortalwrt-hanwckf-firmware-build](https://github.com/hkint/xiaomi-ax3000t-immortalwrt-hanwckf-firmware-build)

---

**参考：**

[小米 AX3000T 路由器刷入使用官方原版 OpenWrt / ImmortalWrt 固件](https://note.okhk.net/xiaomi-ax3000t-router-install-openwrt-immortalwrt)

[小米 AX3000T 路由器使用 hanwckf 版本的 ImmortalWrt 和 U-Boot](https://okhk.net/xiaomi-ax3000t-router-with-hanwckf-immortalwrt)

[[小米 AX3000T] 小米 AX3000T 新设备出厂固件版本 1.0.84 开启 ssh 及相关参考资料](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=8395187)

[[小米 AX3000T] 小米 AX3000T 1.0.84 固件不降级成功刷上 OpenWrt（一键 ssh）](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=8404780)
