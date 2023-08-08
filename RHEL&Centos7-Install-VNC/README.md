## VNC 介绍
VNC是一种远程桌面软件，允许图形软件应用程序的显示被发送给另外一个显示屏，而用户键盘和鼠标操作则发送到另外一个计算机，就像用户坐在另一台机器前面操作一样。这使得用户可以在某个位置访问并控制另一台计算机，可能位于一个完全不同的地理位置。

VNC的核心部分是由一个服务器和一个客户端（或多个客户端）组成。服务器安装并运行在您希望访问的计算机上，而客户端软件则在您实际使用的计算机上运行。

在Linux中，VNC的应用场景举例如下：

1. **远程访问桌面**：例如，您可以在家中访问办公室的机器，反之亦然。
2. **提供技术支持**：帮助其他用户解决其机器上的问题，而不需要实际到现场。
3. **教育和演示**：在多个计算机上显示相同的内容。

常见的Linux VNC服务器有`TightVNC`, `RealVNC`, `TigerVNC`等。
### 1. Centos/RHEL7 安装 VNC
会分别为root用户，以及普通用户安装VNC桌面。
#### 1.1 创建Linux普通用户
```bash
# 创建名为oracle的用户
sudo useradd oracle

# 为新用户设置密码
sudo passwd oracle
```
#### 1.2 安装 VNC 所需的 Package
```bash
# 安装 GUI 桌面软件包以通过 GUI 访问 VNC 服务器
yum groupinstall "Server with GUI"

# 在VNC服务器上，安装TigerVNC服务器包
yum install tigervnc-server
```
#### 1.3 设置 VNC 服务
将配置文件 /lib/systemd/system/vncserver@.service 复制到目录 /etc/systemd/system/，命名为“vncserver_[username]@:[port].service”。 例如：
```bash
cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver_root@:2.service

cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver_oracle@:3.service
```
修改 vncserver_root@:2.service & vncserver_oracle@:3.service 配置文件，确保其内容如下（可以将原本[Unit]、[Service]、[Install]下的内容替换）：
vncserver_root@:2.service
```bash
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=forking
User=root

# Clean any existing files in /tmp/.X11-unix environment
ExecStartPre=-/usr/bin/vncserver -kill %i
ExecStart=/sbin/runuser -l root -c "/usr/bin/vncserver %i -geometry 800x800"
PIDFile=/home/root/.vnc/%H%i.pid
ExecStop=-/usr/bin/vncserver -kill %i

[Install]
WantedBy=multi-user.target
```
vncserver_oracle@:3.service
```bash
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=forking
User=oracle

# Clean any existing files in /tmp/.X11-unix environment
ExecStartPre=-/usr/bin/vncserver -kill %i
ExecStart=/sbin/runuser -l oracle -c "/usr/bin/vncserver %i -geometry 800x800"
PIDFile=/home/oracle/.vnc/%H%i.pid
ExecStop=-/usr/bin/vncserver -kill %i

[Install]
WantedBy=multi-user.target
```
当使用 systemctl start 启动服务器时，会调用 ExecStart 条目中指定的命令； 它使用 runuser 在用户帐户下运行 TigerVNC。 -l 参数提供用户名，-c 指定 runuser 将执行的命令及其参数。 PIDFile 条目指定正在运行的进程将在其中跟踪其进程 ID 的目录。
> 注意：从 RHEL7.4 开始，支持在调用时传递给 vncserver 的服务器选项已移至 ~/.vnc/
> 目录中名为“config”的新文件中。 因此无需在 ExecStart 行中添加这些选项。
#### 1.4 设置防火墙
防火墙应允许显示器相应端口的流量。 显示器 0 使用端口 5900，显示器 1 使用端口 5901，显示器 2 使用端口 5902，依此类推。 如果您使用 FirewallD，预定义的 vnc-server 服务将打开端口 5900-5903：
```bash
firewall-cmd --zone=public --permanent --add-service=vnc-server

firewall-cmd  --reload

systemctl daemon-reload
```
#### 1.5 开启 VNC 服务并设置登录密码
为所选端口上的每个用户启用 vncserver 服务，这还将启用系统启动时的自动启动，使用以下命令：
```bash
systemctl enable vncserver_root@:2.service

systemctl enable vncserver_oracle@:3.service

# 重新加载 systemd 的配置以使其识别新的单元文件
systemctl daemon-reload

# 为每个使用 vncserver 的用户配置密码
$ vncpasswd root
Password:
Verify:
Would you like to enter a view-only password (y/n)? n

$ vncpasswd oracle
Password:
Verify:
Would you like to enter a view-only password (y/n)? n

# 您需要以用户身份登录时在命令行中执行“vncserver”。 这将自动要求您为用户创建一个新密码。（分别在root，oracle用户下执行）
$ vncserver -geometry 1920x1080
You will require a password to access your desktops.

Password:
Verify:
Would you like to enter a view-only password (y/n)? n
xauth:  file /root/.Xauthority does not exist

New 'geeklab:1 (root)' desktop is geeklab:1

Creating default startup script /root/.vnc/xstartup
Creating default config /root/.vnc/config
Starting applications specified in /root/.vnc/xstartup
Log file is /root/.vnc/geeklab:1.log
```

> 注意：可以通过 $vncserver -help 查看命令帮助 usage: vncserver [:<number>] [-name
> <desktop-name>] [-depth <depth>]
>                  [-geometry <width>x<height>]
>                  [-pixelformat rgbNNN|bgrNNN]
>                  [-fp <font-path>]
>                  [-cc <visual>]
>                  [-fg]
>                  [-autokill]
>                  [-noxstartup]
>                  [-xstartup <file>]
>                  [-fallbacktofreeport]
>                  <Xvnc-options>...
> 
>        vncserver -kill <X-display>
> 
>        vncserver -list

#### 1.6 VNC 运行检验
```bash
# vncserver -list 命令
vncserver -list

TigerVNC server sessions:

X DISPLAY # 	PROCESS ID
:1          	25853

# netstat 命令
netstat -antp | grep Xvnc
tcp    	0  	0 0.0.0.0:5901        	0.0.0.0:*           	LISTEN  	25853/Xvnc     	 
tcp    	0  	0 0.0.0.0:6001        	0.0.0.0:*           	LISTEN  	25853/Xvnc     	 
tcp    	0  	0 9.30.215.215:5901   	9.197.228.91:57827  	ESTABLISHED 25853/Xvnc     	 
tcp6   	0  	0 :::5901             	:::*                	LISTEN  	25853/Xvnc     	 
tcp6   	0  	0 :::6001             	:::*                	LISTEN  	25853/Xvnc
```
#### 1.7 VNC 远程登录
您可以在客户端计算机上安装任何 VNC 查看器软件来访问 VNC 服务器。 我在 MAC 上使用 realVNC 软件来访问 VNC 服务器。 您可以根据您使用的操作系统使用以下任意 VNC 查看器软件。
1.TigerVNC：http://tigervnc.org
2.TightVNC：https://www.tightvnc.com/download.php
3.RealVNC：https://www.realvnc.com/en/connect/download/viewer

我使用的是 RealVNC：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f00612426b8c43ceb9e99f1a9bac570f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/902c891091a54032828bd89ed9a745f0.png)
