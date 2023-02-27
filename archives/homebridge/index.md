# A short instruction to Homebridge

###### 把 Apple 的 HomeKit 带入寝室

## 简介
#### 什么是Homekit

>HomeKit is a software framework by Apple that lets users set up their iPhone or other Apple device to configure, communicate with, and control smart-home appliances. By designing rooms, items, and actions in the HomeKit service, users can enable automatic actions in the house through a simple voice dictation to Siri or through apps.[1] HomeKit was first released with iOS 8 in September 2014. HomeKit support is not available in macOS, but it is available on all their other devices, including through Siri.

Homekit 是一个由 Apple 研发的软件框架。它允许用户使用iPhone或其他Apple设备配置、通讯、控制智能家庭设备。通过在Homekit服务中定义房间，设备，动作，用户可以通过Siri和手机应用在家中启用一系列的自动化操作。Homekit首次和iOS8亮相于2014年九月。Homekit 不支持macOS，但支持其他设备。

通过这篇教程，你将会对如何部署一个智能家庭系统有大致的概念

#### 效果

- Siri 自动开灯
- Siri 关闭空调
- 开门报警

#### 什么是[Homebridge](https://www.npmjs.com/package/homebridge)
Homekit 是使用了苹果公司的专有协议。Homebridge是一个轻量级的Node.js 服务器，用NodeJS模拟了一个Accessory Server，可以在局域网上模拟HomeKit API。  Homebridge 的作者KhaosT是个在美国留学的中国人，曾在Homekit 团队中工作。他逆向工程了Homekit协议并将其开源。目前Homebridge可以在GitHub上找到。  

Homebridge在iOS和智能硬件之间架起一座bridge。从而实现将非MFi认证的设备接入苹果的Homekit平台。
## 准备
#### 需要的材料

- 一部（类）Linux电脑（可以是树莓派，Linux虚拟机，macOS）
- 一部iOS设备（iPhone，iPad）
- 智能硬件（此处是小米飞利浦智睿台灯）
- 一个路由器
#### 可以提前下载的数据
- *VMware虚拟机*
- *Ubuntu 系统镜像*
- *Xcode*

#### 需要的知识技能
   - 虚拟机知识
   - Linux 基本操作（可以很快学会）

理论上，Homebridge支持任何能运行Node.js的电脑，包括Windows，但Windows下尚无合适的安装手段，支持也不如Linux
##安装VMware虚拟机
1. 下载VMware虚拟机，关于软件版本，选择Workstation(Windows)或者Fusion(Mac)，雪多大学都有和VMware的合作，可以拿到免费正版的虚拟机软件。
2. 在VMware中新建一个虚拟机，虚拟机系统选择Ubuntu x64 (这里以Ubuntu x64 17.01 LTS为例子)。VMware的向导给出的默认设置无需更改，只要注意内存设定在1GB以上。  
创建虚拟机的系统需要Ubuntu x64 的安装镜像，可以在官网免费下载。若要载入镜像，在VMware设置向导弹出相关提示的时候选择下载的镜像即可。VMware自带安装向导。根据相应提示设置虚拟机中的用户名和密码，**密码需要牢记**。
3. 设置向导运行完毕之后，VMware会自动启动虚拟机，系统安装是全自动进行的。
    - **虚拟机启动出错**  
      部分电脑(惠普的某些型号)出厂默认屏蔽了Intel VT-x技术，导致虚拟机无法正常启动。因此需要进入电脑的BIOS打开Intel VT-x技术。不同电脑进入BIOS方法不同，请参照这篇文章进行解决。  
      [vmware安装ubuntu " Intel VT-x 处于禁用状态"](https://jingyan.baidu.com/article/fc07f98976710e12ffe519de.html)

## 安装Homebridge
#### 安装
1. 首先将电脑连接到局域网。这时应该可以打开网络与共享中心(Windows)或网络偏好设置(Mac)，查看电脑本地的iP地址，其形如```192.168.1.x```
2. 在VMware中将虚拟机的网络适配器设置为桥接模式。这样VMware中的虚拟机和物理机、智能硬件、homekit 支持的iOS设备处于同一个局域网下。虚拟机的IP地址可以在网络偏好设置中查看。请确保虚拟机分配了一个IP地址，其形如```192.168.1.x```


2. 安装homebridge插件
    ###### 虚拟机中
      homebridge插件是由npm进行管理的。npm是Nodejs的组件之一，可以将Nodejs代码的部署简化到一条命令。好比iOS系统中的App Store。  
      在Ubuntu中，Nodejs可以通过apt-get进行管理。apt-get之于linux，正如npm之于Nodejs。
      apt-get 命令需要在终端中进行操作。点击Ubuntu桌面左下角的菜单，在弹出的应用程序列表中找到终端并单机打开，列表可以滚动。

      apt-get 命令的基本用法是 apt-get [操作] [参数]
        > -   apt-get install xxx
        > -   apt-get uninstall xxx
        > -   apt-get update


    大部分的apt-get操作需要root权限，需要在命令前加上sudo来获取相关权限。执行带有sudo的命令后，Ubuntu会要求输入创建系统时的密码。密码输入时没有提示，没有回显，放心输入就好。

    - 首先,我们使用apt-get的update操作更新软件源。这样apt-get将会从软件源获取最新的软件列表，确保其安装的是软件的最新版本。

          sudo apt-get update
      >*该操作耗时取决于网络情况。由于国内网络环境的限制，Ubuntu的软件源可能需要更换为国内源。*  

    - 然后安装Node.js和npm，Nodejs是homebridge运行的基础

          sudo apt-get install nodejs
          sudo apt-get install npm
    - 安装Avahi依赖包，此包允许Ubuntu用苹果的协议通讯

          sudo apt-get install libavahi-compat-libdnssd-dev
    - 最后安装homebridge

          sudo npm install -g --unsafe-perm homebridge

    ###### macOS

      macOS 作为类Linux系统，安装较为简单：

      - 安装[Homebrew](https://brew.sh)，在终端中输入

          /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
          Homebrew 之于macOS，如同apt-get之于Ubuntu，也是管理软件的工具。

      - 在App Store中搜索安装Xcode

      - 安装 nodejs，在终端中输入，然后等待

            brew install Nodejs

      - 安装 homebridge

            npm install -g homebridge

## 为Homebridge设置小米飞利浦智睿台灯
需要下载[米家APP](http://home.mi.com/index.html)，按照提示将小米台灯连入寝室的WiFi
## 配置Homebridge
#### 1. 启动homebridge
  在Ubuntu系统中打开终端，输入

      homebridge

  即可启动homebridge，屏幕上会显示二维码和PIN码（三组共八位数字），表示Homebridge已经正常安装在系统中了  
  在终端中按下<kbd>Ctrl</kbd>+<kbd>C</kbd>退出homebridge  

  macOS系统的终端可以在launchpad中打开，默认在“其他”文件夹中，也可以使用spotlight搜索来找到
#### 2. 添加插件
1. 安装 [miio](https://github.com/aholstenson/miio) （Mi Input/Output）插件  

        sudo npm install miio@0.14.0

说明：这个插件的用途是发现、操控局域网中的小米设备。  
>截至目前，最新版本的插件有很多兼容问题。因此，这里我们使用它的0.14.0版本，需要在安装的时候附上版本号。

2. 安装[mi-philips-light](https://github.com/YinHangCode/homebridge-mi-philips-light)插件

        sudo npm install -g homebridge-mi-philips-light

说明：这个插件用于把飞利浦台灯接入homebridge平台，实现手机对台灯的控制。

#### 3. 配置config.json文件
  Homebridge 的配置文件保存在用户目录下的 ```\.Homebridge\config.json``` 中，格式是纯文本文件，因此应用纯文本文件编辑器编辑，而不应当使用Word等软件。对配置文件的更改在重启homebridge之后生效。
  编辑Vim有两种方法：

  ###### 1. 终端法 - 使用Vim  

  这里需要用到Linux下的一个应用程序：Vim。大体上，vim 是一个Linux下的文字编辑工具，正如同记事本是Windows下的文字编辑工具。如果没有Vim 可以通过执行命令：

    sudo apt-get install vim
  来安装。  
  首先，关闭homebridge，然后在终端中执行下列命令：

    cd ~
    cd .homebridge
    vim config.json
>######VIM 编辑器的使用:
  >- 使用方向键移动光标  
  >- 按下 <kbd>I</kbd> 键进入编辑模式
  >- 按下 <kbd>ESC</kbd> 退出编辑模式
  >- 退出编辑模式后，输入 ```:wq``` 保存（Write and Quit）

###### 2. 图形法 - 使用文本编辑程序

因为只需要更改config.json 这个纯文本文件，所以可以用更加简单的工具。Ubuntu自带的文字工具就很方便。你甚至可以在物理机中修改好配置文件，然后利用VMware的文件拖放功能传入虚拟机。无论用什么工具，保存的时候都应当选择UTF-8编码，尤其是中英文混合的情况。
>不要使用Windows的记事本或写字板，他们都不是正宗的纯文本编辑器，会导致错误。应当使用Ultraedit或Atom。

在Ubuntu中打开文件管理器，进入用户的个人目录。点击右上角的菜单-显示隐藏文件，就可以看到```.Homebridge``` 目录了。config.json文件存储在此目录下，可以访问并修改。

1. **基本设置**

  Homebridge配置文件的格式如下：
```
  {
      "bridge": {
          "name": "Home",
          "username": "FF:FF:FF:FF:FF:FF",
          "port": xxxxx,
          "pin": "xxx-xx-xxx"
      },

      "description": "your description",

      "accessories": [],
      "platforms": []
  }
```
* ```name```        随便填写。
* ```username```    填写十六进制数字，如 12:34:56:78:9A:BC 是homebridge的唯一名称。
* ```port```        填写1024-65536 之间的数字，是homebridge的通讯端口，可选择修改。
* ```pin```         是连接homebridge的密码，形如 123-45-678 默认已经设置好，可选择修改。
* ```description``` 随意填写
* ```accessories``` homebridge 里的概念，配件
* ```platforms```   homebridge 里的概念，平台

举个例子：

```
  {
      "bridge": {
          "name": "SJTU-Library",
          "username": "78:97:E8:2E:45:93",
          "port": 54321,
          "pin": "123-45-678"
      },

      "description": "homebridge-test",

      "accessories": [],
      "platforms": []
  }
```
2. **插件设置**  

mi-philips-light给出了如下的配置指导：
```
{
  "platform": "MiPhilipsLightPlatform",
  "deviceCfgs": [{
      "type": "MiPhilipsTableLamp2",
      "ip": "192.168.1.x",
      "token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "mainLightName": "mainLight",
      "secondLightName": "secondLight",
      "secondLightDisable": false,
      "eyecareSwitchName": "eyecare",
      "eyecareSwitchDisable": false
  }]
}
```
* ```ip``` 填写上一步获取的台灯的IP地址
* ```token```填写上一步获取的台灯的token
* ```platform```platform名称，不可更改
* ```secondLightDisable``` 设为true可使得副灯失效
* ```eyecareSwitchDisable```设为true可使得护眼模式失效

最终我们得到这样的配置文件
```
{
    "bridge": {
        "name": "SJTU-Library",
        "username": "78:97:E8:2E:45:93",
        "port": 54321,
        "pin": "123-45-678"
    },

    "description": "homebridge-test",

    "accessories": [],
    "platforms": [{
      "platform": "MiPhilipsLightPlatform",
      "deviceCfgs": [{
          "type": "MiPhilipsTableLamp2",
          "ip": "192.168.1.x",
          "token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
          "mainLightName": "mainLight",
          "secondLightName": "secondLight",
          "secondLightDisable": false,
          "eyecareSwitchName": "eyecare",
          "eyecareSwitchDisable": false
      }]
    }]
}
```
保存文件
# 将Homebridge与iPhone连接
#### 1. 启动homebridge
  在Ubuntu系统中打开终端，输入

      homebridge

  即可启动homebridge，屏幕上会显示二维码和PIN。
#### 2. 添加homebridge到Home
  打开“家庭”App（如果此前不慎删除可在App Store重新下载），点击右上角的加号选择添加配件。然后扫描Ubuntu终端上显示的二维码。iOS会提示该配件没有经过认证，大可忽略。

  如果无法扫描成功，可以选择“没有代码或无法扫描”，选择出现的homebridge，手动输入屏幕上显示的PIN码来添加

# 愉快地在寝室里使用
若要愉快地在寝室里使用，需要：
* 一台常开的电脑.推荐购买[树莓派](https://baike.baidu.com/item/树莓派/80427?fr=aladdin)等迷你电脑。树莓派运行与Ubuntu很像的Debian系统（先有Debian后有Ubuntu）。但是，树莓派默认没有屏幕，因此安装方法有些不同。
* 寝室的宽带允许路由器的存在，详情咨询运营商相关情况。在某些学校，比如上海财经大学，宿舍网络需要专门的客户端认证，因此路由器无法存在，或存在而无法连接互联网。Homebridge的控制要求iOS设备，运行homebridge的电脑，智能硬件在同一局域网下。因此这些寝室如果按照这个教程装设homebridge，将面临频繁切换网络的问题（联网和遥控不可兼得）。
* 购买/制作相关的智能硬件，需要钱和技术。
* 一台放在寝室里的iPad/Apple TV，这样以来，就可以从寝室外遥控操作。需要钱。如果没有iPad，就只能在寝室里控制智能硬件。

# 相关连接
* [apt-get 是什么](https://baike.baidu.com/item/apt-get/2360755?fr=aladdin)
* [sudo 是什么](https://baike.baidu.com/item/sudo/7337623?fr=aladdin)
* [终端是什么](https://www.kafan.cn/edu/45159581.html)
* [Windows 下虚拟机的安装](https://blog.csdn.net/u014180259/article/details/52040640)
* [交大福利-免费VMware](http://vmap.sjtu.edu.cn)
* [端口是什么](https://baike.baidu.com/item/端口/103505?fr=aladdin)
* [桥接](https://baike.baidu.com/item/桥接/3574841?fr=aladdin)
* [Windows 安装Nodejs](https://nodejs.org/en/)
