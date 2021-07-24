
工作需要用到WiFi Captive Portal热点做测试，而路由器不方便开启这个功能，遂想到自己搭建

WiFi Captive Portal 中文一般叫做无线认证门户

本文主要翻译自[ _How to Turn Your Raspberry Pi into a Captive Portal Wi‐Fi Access Point_ ](https://www.maketecheasier.com/turn-raspberry-pi-captive-portal-wi%E2%80%90fi-access-point/)

在树莓派4b上实验成功

## 安装步骤

### 安装raspap开源项目

```
curl -sL https://install.raspap.com | bash
```

装完reboot重启

装完后相关信息如下：

IP address: 10.3.141.1

Username: admin

Password: secret

DHCP range: 10.3.141.50 — 10.3.141.255

SSID: raspi-webgui

Password: ChangeMe

打开浏览器，输入10.3.141.1

修改WiFi密码，保存

### 用nodogsplash创建captive portal

nodogsplash需要自己编译安装

安装依赖库

```

sudo apt install git libmicrohttpd-dev

```

```

git clone https://github.com/nodogsplash/nodogsplash.git

```

```

cd ~/nodogsplash

make

sudo make install

```

### 配置captive portal

```
sudo vi /etc/nodogsplash/nodogsplash.conf
```

增加以下配置

```

GatewayInterface wlan0

GatewayAddress 10.3.141.1

MaxClients 250

AuthIdleTimeout 480

```

启动

```
sudo nodogsplash
```

**至此，树莓派接入有线网+WiFi热点建立Captive Portal基本功能搭建完成**

## 进阶

### 开机自动启动nodogsplash

sudo vi /etc/rc.local

在exit 0前加入

nodogsplash

这个方法似乎有问题，改用另外一个方法，从NDS [github issue](https://github.com/nodogsplash/nodogsplash/issues/533)上看到

```
sudo vi /lib/systemd/system/nodogsplash.service
```

```

[Unit]

Description=NoDogSplash Captive Portal

After=network.target

[Service]

Type=forking

ExecStartPre=sleep 10

ExecStart=/usr/bin/nodogsplash $OPTIONS

Restart=on-failure

[Install]

WantedBy=multi-user.target

```

之后运行

```
sudo systemctl enable nodogsplash
```

重启后生效

### 树莓派WiFi同时做WiFi接入点和AP热点

前面方法都是用有线做接入点，稍显不便，配置成无线做接入点，方便移动

时间关系，留到后续再搞

### 修改热点改成5G频段

TODO

### 关闭RaspAP

```

sudo systemctl disable nodogsplash.service

```

进入10.3.141.1管理界面，用户名admin, 密码secret

手动Stop hotspot

彻底关闭RaspAP自动启动，待研究

## 参考资料

- [How to Turn Your Raspberry Pi into a Captive Portal Wi‐Fi Access Point](https://www.maketecheasier.com/turn-raspberry-pi-captive-portal-wi%E2%80%90fi-access-point/)

- [Rasp AP Project](https://github.com/RaspAP/raspap-webgui)

- [Nodogsplash(NDS) Project](https://github.com/nodogsplash/nodogsplash)

- [Rasp AP Captive官方文档, 看到晚了，这篇讲得很详细](https://docs.raspap.com/captive.html)

- [自制简单Wi-Fi Captive Portal，支持树莓派](https://iedon.com/2017/10/18/623.html)

- [ndsctl命令手册](https://opennds.readthedocs.io/en/stable/ndsctl.html)
