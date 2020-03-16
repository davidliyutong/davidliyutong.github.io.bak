# ESP8266 智能锁
2018-4-16 @davidliyutong
## 简介
一个可以用python控制的，基于HTTP请求的智能锁。  
使用了[Arduino](http://arduino.cc) IDE，[esp8266 / Arduino](https://github.com/esp8266/Arduino)

![image](https://github.com/davidliyutong/esp8266-smartlock/raw/master/images/ESP-01.png)
![image](https://github.com/davidliyutong/esp8266-smartlock/raw/master/images/USB-TTL.png)
## 实现原理
#### 服务器
脚本 ```server.py``` 在一台计算机上创建一个Server。该Sever共有三个页面：```\on``` ```\off```和 ```\status.html```，它的作用是存储锁的状态变量```status```（*1* 为开，*0* 为关）。

三个页面中，```\on``` 和```\off```页面为操作页面。服务器监听到对这两个页面的请求就改变存储的锁状态```status```（on-1，off-0）。当```\status.html```被请求的时候，服务器根据存储的```status```变量，返回字符“0”或“1”，发送其存储的锁的状态。
####ESP8266
ESP8266在启动后进入 *Soft_AP* 模式。 这个模式下ESP8266工作为一个热点。ESP8266的地址，即路由地址默认为```192.168.4.1```。

在这个模式下ESP8266等待客户端的连入，并接受客户端```setup.py```发送的Wi-Fi配置（ssid 和 password）、request_url请求地址。```setup.py```脚本以 HTTP GET 的形式发送这三个参数。这三个参数在请求的地址中体现，用“=”分割，如：

    http://192.168.4.1/ssid_custome=password_custome=request_url

其中，```request_url```为Server的域名（或IP地址）+ 返回状态的地址  
（比如 http://example.com/status.html）

ESP8266 收到配置后（若ssid_custome长度大于0）转为 *Station* 模式。这个模式下ESP8266工作为一个终端。它会尝试根据```setup.py```发送的ssid和password尝试连接到无线基站。

连接成功后，ESP8266依据给定的request_url以一秒的间隔向Server请求锁的状态。这里用LED模拟锁。若得到“1”则ESP8266将LED引脚置高，若得到”0”则ESP8266将引脚置低。
#### 客户端
客户端```command.py```获得用户的命令。根据命令```command.py```以 HTTP GET 的方式请求Server的```\on``` 和```\off```页面。从而达到改变Server中```status```值变量的意义。
## 物料
- ESP8266 这里使用ESP-01 模块
- USB转串口烧录器
## 硬件连接
按照提示将ESP8266与烧录器连接。将烧录器连接到电脑的USB接口


![image](https://github.com/davidliyutong/esp8266-smartlock/raw/master/images/Connection.png)
## 配置Arduino IDE
#### 安装驱动
- 烧录器Windows 下即插即用（Windows 10）
- Mac 下需要安装烧录器的驱动，不同型号的烧录器，驱动不同。以CH340 驱动为例，下载地址为  
 http://www.wch.cn/download/CH341SER_MAC_ZIP.html
#### 安装ESP8266 开发版
1. 在 Arduino IDE 的偏好设置中添加开发版管理网站  
 http://arduino.esp8266.com/stable/package_esp8266com_index.json  
 ![image](https://github.com/davidliyutong/esp8266-smartlock/raw/master/images/arduinoSettings.png)
2. 工具>开发版>开发版管理器 安装ESP8266定义  
![image](https://github.com/davidliyutong/esp8266-smartlock/raw/master/images/esp8266Platform.png)
3. 重启 Arduino IDE  
4. 工具>开发版 中选择 ```Generic ESP8266```
5. 在Arduino IDE 端口设置中选择相应的端口

#### ESP8266 开发配置

  - Arduino 里的编程器选择默认的 AVRISP mkII
  - *特别注意* 在Arduino 工具里为**Flash Mode** 选择 ```DOUT```模式  
   ![image](https://github.com/davidliyutong/esp8266-smartlock/raw/master/images/arduinoIDE.png)
## 烧录器的使用
**跳帽** 的作用是让相关端口短接，因为没有电表，因此也不了解短接之后端口的电位。  
使ESP8266工作需要用跳帽或导线短接```SW```*（对应CH_PD，从ESP8266工作特性来看应该被置高了）*，此外刷写ESP8266还需要短接```GPIO0```*（从ESP8266工作特性来看应该被置低了）*

#### 刷写ESP8266需要如此操作
1. 把ESP8266 插入烧写器
2. 将 ```SW``` 和 ```GPIO0``` 用跳帽或导线短接
3. 把烧写器插入电脑USB接口
4. 在Arduino中选择ESP8266对应的串口
5. Arduino 中编译上传，此时ESP8266 写入指示灯闪烁
6. 上传完成，拔掉 ```GPIO0``` 的跳帽或导线，**拔掉后再连接** ```SW``` 的（这是为了重置ESP8266，转换状态）
7. ESP8266 开始执行程序

#### 若要再次写入程序
1. 用跳帽或导线连接 ```GPIO0``` ，**拔掉后再连接** ```SW``` 的
2. Arduino 中编译上传，此时ESP8266 写入指示灯闪烁
3. 上传完成，拔掉 ```GPIO0``` 的跳帽或导线，**拔掉后再连接** ```SW``` 的

## 小技巧
- Arduino 事例有现成的Example可以实验，大部分只需要修改Wifi 的```SSID```和```PSK```  
- 可以用导线，开关和面包版快速复位，实现快速转换模式，从而免去插拔USB

## 改进
- 这个智能锁能够顺利的接入homebridge，实现用siri控制灯。[rudders/homebridge-http]() 这里提供了一个解决方案。homebrdige 配置文件里的```status_on```和```status_off```填写服务器的两个操作地址```\on``` 和 ```\off```
- 服务器可以加入记忆状态的功能
- 当前，ESP8266 在断电后会丢失配置信息。可以加入配置保存功能，利用提供的文件操作函数将配置保存在闪存上。
- 在程序中加入判断功能，如果为能连接到Wi-Fi或服务器提示重连。
- 加入硬件复位键，一键复位，清除存储的配置信息
- python编程直接发送请求效率低下，难以维护。考虑换用更高效的Web框架和语言，比如apache，php

- 每秒向服务器发送一次GET请求对服务器是很大的负担。考虑：
  - 与服务器握手发送自身IP地址，然后监听服务器发送的指令，困难在于如何穿过网关
  - P2P连接
  - 使用别的物联网技术，如MQTT
  - TCP长连接
  -   ...
- 服务器没有任何的认证措施，考虑加入一些验证
- HTTP连接很不安全，考虑换用HTTPS连接。需要配置证书
