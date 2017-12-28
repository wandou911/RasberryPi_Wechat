# RasberryPi_Wechat

一个以微信为终端的好玩的小东西

可以实现的功能

可以实现以手机微信端对树莓派终端进行实时监控、摄像头云台操纵、闯入报警、温度检测、灯光控制、自动光线控制等功能

设备图片

运行截图

# 需要用到的所有硬件

路由器

树莓派主板

树莓派电源（5V 2A）

至少 8g tf卡 （推荐class 10，8g足矣）

支持ouv的摄像头（罗技C170）


乐高积木（小颗粒）

两根网线

温度传感器（DHT11）

光线传感器（光敏电阻模块）

人体红外传感器（HC-SR501）

继电器（5V低电平触发）

步进电机（28BYJ-48-5V）

步进电机驱动板（UL2003型）

GPIO连接线若干

# 需要安装的所有软件

Windows或者Mac端

Raspberry Pi端

[RASPBIAN 系统](https://www.raspberrypi.org/downloads/)

webpy

python-lxml

python-memcache

apache2

mjpg-streamer

RPI.GPIO

# 此程序的全部源码

源码地址： https://github.com/wandou911/RasberryPi_Wechat

# 配置过程

## 1 初始化树莓派

### 更新缓存并升级软件
```
sudo apt-get update && apt-get upgrade
```
###安装及配置 安装所需要的所有软件，将必须的软件包安装完毕，并且调试成功 调试过程如果有问题可以参见本博客（附录）中的其他文章，或欢迎留言讨论 

####安装软件

webpy
```
git clone git://github.com/webpy/webpy.git
cd webpy
sudo python setup.py install
```
RPI.GPIO （安装RPI.GPIO 首先需要安装RPi.GPIO所需的Python Development Toolkit）
```
sudo apt-get install python-dev
sudo apt-get install python-pip
sudo pip install rpi.gpio
```
python-lxml
```
sudo apt-get install python-lxml
```
python-memcache
```
sudo apt-get install python-memcache
```
apache2
```
sudo apt-get install apache2
```
mjpg-streamer （安装mjpg-streamer 首先需要安装一下几个依赖包）
依赖包：
```
sudo apt-get install subversion
sudo apt-get install libv4l-dev
sudo apt-get install libjpeg8-dev
sudo apt-get install imagemagick
```
编译安装mjpg-steamer：
```
wget http://sourceforge.net/code-snapshots/svn/m/mj/mjpg-streamer/code/mjpg-streamer-code-182.zip
unzip mjpg-streamer-code-182.zip
cd mjpg-streamer-code-182/mjpg-streamer
make USE_LIBV4L2=true clean all
make DESTDIR=/usr install
```

## 2 配置内网穿透 frp 

### 2.1 安装frps 服务端（vps主机）

需要购买vps主机 [搬瓦工](https://bwh1.net/aff.php?aff=19935） 或者  [vultr](https://www.vultr.com/?ref=7236384)


下载一键安装包 安装
```
wget --no-check-certificate https://raw.githubusercontent.com/clangcn/onekey-install-shell/master/frps/install-frps.sh -O ./install-frps.sh
chmod 700 ./install-frps.sh
./install-frps.sh install
```
安装过程中 可以一直回车按默认选项 直到安装成功 安装安装成功截图：

![安装成功截图](https://images2017.cnblogs.com/blog/1044995/201712/1044995-20171226155743729-1396171587.png)

启动frps：
```
frps start
```
备注:frps 常用命令：启动（frps start），停止（frps stop）配置文件（frps config）

### 2.2 安装frpc 客户端（树莓派）

下载编译好的arm版本 

```
wget https://github.com/fatedier/frp/releases/download/v0.14.1/frp_0.14.1_linux_arm.tar.gz
tar zxvf frp_0.14.1_linux_arm.tar.gz #解压
cd frp_0.14.1_linux_arm
vi frpc.ini
```

修改 frpc.ini 文件，假设 frps 所在的服务器的 IP 为 x.x.x.x，

local_port:8080 为本地机器 web 服务对应的端口,绑定自定义域名 raspberry.yourdomain.com:

local_port:80   为本地机器 web 服务对应的端口,绑定自定义域名 weixin.yourdomain.com:
```
# frpc.ini
[common]
server_addr = x.x.x.x
server_port = 5443 #和服务端对应

[web_raspberry_web]#名称不能重复
type = http
local_port = 8080#端口号 对应本机web服务器端口
custom_domains = raspberry.yourdomain.com

[web_raspberry_weixin]#名称不能重复
type = http
local_port = 80#端口号 对应本机web服务器端口
custom_domains = weixin.yourdomain.com#可以配置多个子域名
```

## 3 申请域名并解析

将 yourdomain.com 的域名 A 记录解析到 IP x.x.x.x

![域名解析](https://images2017.cnblogs.com/blog/1044995/201712/1044995-20171228115824113-354951591.png)

## 4 配置 Apache 

#### 因为我的80端口需要被微信公众平台占用，所以我不能让网页走80端口，需要更改端口


修改Apache2端口号为：8080
```
cd /etc/apache2
sudo vi ports.conf
```

将 Listen 80 改为Listen 8080
```
:wq
```
保存退出

关闭Apache
```
/etc/init.d/apache2 stop
```
启动Apache
```
/etc/init.d/apache2 start
```

此时打开浏览器输入http://raspberry.yourdomain.com:8080 

如果看到it works 说明Apache配置成功


## 5 部署web页面

编辑Git包中的文件中的index.html，在你的树莓派ip处改为树莓派的ip地址或者raspberrypi.local
```
cd RaspberryWechatPi
vi index.html
```
将你的树莓派ip处改为raspberrypi.local
```
:wq
```

将index.html上传到/var/www目录下了，替换之前的index.html
```
sudo cp index.html /var/www/html
```

在浏览器中输入 http://raspberry.yourdomain.com:8080/ 尝试一下能否访问 如果成功出现页面，则web页面部署成功 

备注：apache2 网页存放路径 /var/www/html


## 6 调试摄像头 

在此Github中下载完整代码包，解压后进行编辑 （Git：https://github.com/wandou911/RasberryPi_Wechat）

运行树莓派的Git包目录中testcam文件夹中的“stream.sh”文件：

```
git clone https://github.com/wandou911/RasberryPi_Wechat
cd RasberryPi_Wechat/testcam
sudo chmod +x stream.sh（先增加执行权限，才能用./filename 来运行）
sudo ./stream.sh
```

在运行程序时，如果发生错误，可能是之前由于运行过，进程仍然在工作，导致没法再运行，可以先运行ps -A，查看运行中的进程和进程ID号，再使用kill id号杀掉进程

在pc上运行Git包中的testcam.html文件，右击编辑index.html，将树莓派ip换成树莓派的ip地址或者raspberrypi.local，保存，双击打开testcam.html

看到摄像头输出图像，说明摄像头工作正常。

## 7 申请及配置公众平台测试账号

打开页面 http://mp.weixin.qq.com/wiki/home/index.html 申请一个公共平台的测试账号

在左侧选择 测试号申请|在线调试选择接口测试号申请

申请成功后，进入管理界面

在接口配置信息的URL处输入你在步骤2.2 rpc.ini 配置的子域名：weixin.yourdomain.com，后面加上/weixin Token中填上你自己喜欢的一串字母，完成后不要点击提交 （此时可以用git代码包中的微信公众平台基础模板 testweixin 进行对接，可以对接成功后在进行接下来的工作，以测试网络环境是否配置完毕 

测试网络环境配置

```
cd ~/RasberryPi_Wechat/testweixin
chmod +x testweixin.py
python testweixin.py 80
```

此时在页面点击提交，如果显示配置成功，即可继续下面的操作，如果配置失败，请检查 步骤2 frp 是否配置成功

![微信公众号](https://images2018.cnblogs.com/blog/1044995/201712/1044995-20171225182153603-951671329.png)

## 8 配置主程序

进入RasberryPi_Wechat目录，修改index.py文件

```
cd ~/RasberryPi_Wechat
vi index.py
```

填入刚才自己设置的的Token以及测试号提供的appID和appsecret（yeekey稍后提到）

填入自己的所有传感器对应的GPIO接口 （传感器调试参考此博客（或附录）其他文章）

修改完成后保存退出

```
:wq

```

在刚在文件所在目录执行chmod +x start.sh 增加执行权限

执行 
```
chmod +x start.sh
./start.sh
```

如果出现如图所示信息，则程序正确运行


此时在微信公众平台测试账号的网页上点击提交，如果提示成功，则整套系统基本配置成功


## 9 设置微信公众账号菜单

在微信公众平台管理测试账号下方选择获取access token

在左侧菜单选择 ** 基础接口 获取access token**

在右侧最下方选择 使用网页调试工具调试该接口

首先获得access token 在appid和secret中填上之前管理测试账号页面提供的数据，点击检查问题

下方蓝色的access_token就是一会提交菜单要用到的access_token，复制此token

![获取access_token](https://images2017.cnblogs.com/blog/1044995/201712/1044995-20171227154052269-505421259.png)

分别在接口类型选择自定义菜单和在接口列表选择自定义菜单创建接口。进入如下界面，填入刚才的access_token（access_token具有一定的时效性，时间过长后需重新获取）
在body中填入Git包中的menu.txt内的内容，点击检查问题

若显示Request successful即为菜单创建成功。

![生成自定义菜单](https://images2017.cnblogs.com/blog/1044995/201712/1044995-20171227154408535-50975473.png)


Ps：取消关注微信号重新关注即可直接查看效果。否则受限于微信限制，需要24小时后缓存刷新方可查看。

Ps2：参数说明 

|参数	|是否必须	|说明| 
| ------------- |:-------------:|:-----| 
|button	|是|	一级菜单数组，个数应为13个| 
|sub_button|	否	|二级菜单数组，个数应为15个| 
|type	|是|	菜单的响应动作类型|
|name	|是|	菜单标题，不超过16个字节，子菜单不超过40个字节|
|key|	click等点击类型必须|	菜单KEY值，用于消息接口推送，不超过128字节| 
|url	|view类型必须|	网页链接，用户点击菜单可打开链接，不超过256字节|

## 10 申请Yeelink物联网服务

打开 http://www.yeelink.net/ 注册账号

登陆后在管理首页上，您的API Key 即为yeekey

添加一个新设备

添加完毕后记住自己的设备ID

在程序中填入自己的设备id以及yeekey，并将附近自己的yeelink页面改为自己的页面


## 附录

[参考链接：基于树莓派的智能家居控制平台 微信服务端](https://github.com/mcdona1d/RaspberryWechatPi)

[参考链接：配置微信公众号](http://www.cnblogs.com/mnstar/p/8110655.html)

[参考链接：域名解析](http://www.cnblogs.com/mnstar/p/8134994.html)

[参考链接：frps内网穿透](http://www.cnblogs.com/mnstar/p/8085113.html)


