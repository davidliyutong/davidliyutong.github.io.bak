# 建立SSH隧道，使能PyCharm远程调试

## 背景
本人平时使用一台Macbook Pro进行代码编写。因为课题有机器学习相关的应用需要用到实验室的GPU服务器。一个行之有效的解决方案是使用Pycharm进行远程调试。

## 需求
- PyCharm 专业版（PyCharm Professional）。可以通过教育邮箱参加教育计划领取

## 环境
学校实验室提供了许多GPU服务器$S_1$，$S_2$，$S_3$ ... 。这些服务器架设在实验室的内网，运行了SSH服务，端口为22（记服务器 $S_i$ 的SSH端口为`<Port_Server_i_SSH>`)。但是它们被分配了形如`192.168.1.x`的内网IP地址（记该地址为`<IP_Server_i>`）。因此，这些服务器（目标机）无法通过SSH直接访问。

好在实验室配备了一台专用的跳板机 $G$ 供同学们连接。跳板机 $G$ 与目标机群 $S_i$ 在同一个子网下，被分配了形如`192.168.1.G`的内网IP地址（记该地址为`<IP_Gateway>`）。跳板机上运行了SSH服务，端口为22（记该端口为`<Port_Gateway_SSH>`)。实验室的路由器 $R$ 有公网IP地址（记其为`<IP_Router>`），并设置了端口转发（记该端口为`<Port_Router>`）。这样就有了如下转发规则

```
<IP_Router>:<Port_Router> <-- (TCP) --> <IP_Gateway>:<Port_Gateway_SSH>
```

原来的工作流程是这样的：
1. **使用SSH登陆跳板机**，命令为

```bash
<localhost>$ ssh <username_gateway>@<IP_Router> -p <Port_Router>
```
其中`<localhost>`为本机的主机名（hostname），`<username_gateway>` 为用户在跳板机上的账号

2. **在跳板机上用SSH登陆目标机**，命令为

```bash
<Gateway>$ ssh <username_server_i>@<IP_Server_i> -p <Port_Server_i_SSH>
```
其中`<Gateway>`为跳板机的主机名，`<username_server_i>`为用户在目标机上的账号

PyCharm的远程调试功能需要与远端主机（Remote Host）建立直接的SSH连接，其本身不支持通过跳板机进行远程调试。关于PyCharm远程调试为何物，请另行了解。大体上，PyCharm 会通过SSH发起一个形如`ssh <username>@<hostname> -p <port>`，然后利用其调试工具进行调试。

## 解决方案

即然PyCharm无法直接连接到GPU服务器，我们可以换个思路，通过某种手段将GPU服务器“挪到”一个PyCharm可以连接的地方。这就是**SSH正向代理技术**

所谓正向代理，就是在本地启动端口`<Port_localhost>`将其收到的数据转发到远端服务器。当前情景下，需要执行的命令为

```bash
<localhost>$ ssh -L <Port_localhost>:<IP_Server_i>:<Port_Server_i> \
                  <username_gateway>@<IP_Router> -p <Port_Router>
```
比如
```bash
<localhost>$ ssh -L 9999:192.168.1.233:22 user233@123.45.67.89 -p 2333
```
#### 解释
- `-L` 选项表示使用的是正向代理模式
- `-p` 选项表示指定端口
- 需要跳板机 $G$ 的凭据登陆


运行该命令后，访问`<localhost>:<Port_localhost>`就相当于访问`<IP_Server_i>:<Port_Server_i>`了。我们可以在另一个终端上运行：

```bash
<localhost>$ ssh <username_server_i>@<localhost>:<Port_localhost>
```
输入用户在目标机上的凭据后，就可以登陆了。

如果要用PyCharm调试，就在PyCharm里添加`<username_server_i>@<localhost>:<Port_localhost>`这一远程主机，并配置相应的凭据。

#### 示意图
![](https://davidliyutong.github.io/ssh-pycharm/15877058543068.jpg)

需要注意的是，在使用PyCharm调试期间，必须保持终端上的代理进程持续运行。一旦关闭终端或者登出，代理就会停止。

## 小结
为了方便理解，本文中用IP地址来代表服务器的网络位置。而在配置了解析服务的网络里，可以使用各服务器的主机名（hostname）进行访问。
