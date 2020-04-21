现在手游直播特别火，不过要用到OBS和手机投屏工具，直播网站提供了官方工具，可不包括linux系统。还好有大神相助，下面我们来实现看看。

### 需要用到的工具

1. scrcpy
2. adb

### 安装 Scrcpy

`snap install scrcpy`

### 安装 adb 服务

`sudo apt-get install android-tools-adb`

### 配置 adb

将手机用 USB 连接电脑，查看手机的 USB 识别号。
终端命令：`lsusb`

```shell
Alice@Alice-Aspire-E2-01314G:~$ lsusb
Bus 002 Device 020: ID 22d9:2773
Bus 002 Device 006: ID 1a81:1007 Holtek Semiconductor, Inc.
Bus 002 Device 007: ID 04d9:a096 Holtek Semiconductor, Inc.
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 003: ID 1bcf:2c18 Sunplus Innovation Technology Inc.
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

找到自己的手机识别号。（我的是：`22d9:2773`）

创建设备文件,改成自己的识别号(我的 `22d9` ), android.rules 文件名可自定义

`mkdir ~/.android`

`echo 0x04e8 > ~/.android/adb_usb.ini`

`sudo touch /etc/udev/rules.d/android.rules`

`sudo gedit /etc/udev/rules.d/android.rules`


在文件中输入:

>SUBSYSTEM=="usb", SYSFS{idVendor}=="22d9", MODE="0666"

保存后修改文件权限

`sudo chmod 777 /etc/udev/rules.d/android.rules`

注：如果出现下面的权限错误
``` shell
Alice@Alice-Aspire-E2-01314G:~$ sudo gedit /etc/udev/rules.d/android.rules

(gedit:21546): IBUS-WARNING **: 14:17:29.113: The owner of /home/Alice/.config/ibus/bus is not root!
...略...
```
可以输入 `sudo -i` 键入管理员帐户，再执行上面的命令。

### 启动adb服务

`service udev restart`

`adb start-server`

`adb devices`

``` shell
Alice@Alice-Aspire-E2-01314G:~$ adb devices
List of devices attached
I7EU7LSK5LINDY9D device
```

出现设备说明连接成功。如果没有看看自己手机的开发者模式有没有打开, 不同手机的开发者模式位置不同, 自行google。

### 启动 scrcpy

输入命令行：`scrcpy`

``` shell
Alice@Alice-Aspire-E2-01314G:~$ scrcpy
INFO: scrcpy 1.12 <https://github.com/Genymobile/scrcpy>;
/usr/local/share/scrcpy/scrcpy-server:...shed. 0.3 MB/s (26196 bytes in 0.082s)
INFO: Initial texture: 1080x1920
```

就会弹出手机界面。

但是用USB连接，总会有些不方便。

### 无线连接

手机连接到和电脑相同WiFi，然后查看手机的IP地址。

#### 打开手机5555端口

注意这时手机的USB数据线一定要和电脑连着。然后输入命令行：`adb tcpip 555`

``` shell
Alice@Alice-Aspire-E2-01314G:~$ adb tcpip 5555
restarting in TCP mode port: 5555
```
这样打开手机的5555端口。如果手机上有提示要点 `确定`。

#### 无线连接手机

用刚才打开的 5555 端口连接手机，执行命令行：`adb connect 你的手机IP地址:5555`
```
Alice@Alice-Aspire-E2-01314G:~$ adb connect 192.168.2.100:5555
connected to 192.168.2.100:5555
```

此时通过adb devices可以看到adb列表中已经出现了新的设备。
```
Alice@Alice-Aspire-E2-01314G:~$ adb devices
List of devices attached
192.168.2.100:5555 device
82a0a8cf9804 device
```

拔掉USB数据线。如果新连接的设备后面出现 `unauthorized` 的字样，说明设备未授权。这时执行命令：`adb reconnect offline`，这条命令将会强制未授权的设备重新链接。

这时查看连接设备列表：`adb devices`

```shell
Alice@Alice-Aspire-E2-01314G:~$ adb devices
List of devices attached
192.168.2.100:5555 device
```

输入命令行：`scrcpy`

```
iwhyer@iwhyer-Aspire-E1-571G:~$ scrcpy
INFO: scrcpy 1.12 <https://github.com/Genymobile/scrcpy>;
/usr/local/share/scrcpy/scrcpy-server:...shed. 0.6 MB/s (26196 bytes in 0.040s)
...略...
INFO: Initial texture: 1920x1080
INFO: New texture: 1080x1920

```

即可实现无线投屏。

### 参考文献 

1. [Ubuntu安装scrcpy完成手机投屏和控制(Ubuntu用QQ微信的另一种方法)](https://www.jb51.net/article/172057.htm)
2. [Ubuntu安卓手机投屏](https://blog.csdn.net/zekdot/article/details/94782904)
3. [scrcpy](https://github.com/Genymobile/scrcpy)