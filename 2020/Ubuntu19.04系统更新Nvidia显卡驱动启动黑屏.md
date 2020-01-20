## Ubuntu19.04系统更新Nvidia显卡驱动启动黑屏

### 问题概述
Ubuntu 19.04 更新Nvidia显卡驱动，重启系统的时候出现黑屏。怎么办？

### 解题思路
想办法进入系统终端shell，然后删除 Nvidia 显卡驱动。

### 解决过程
1. 进入终端

启动系统 > 选“Ubuntu 高级设置”，进到下面的界面，选择 root shell 进入终端。

![shell](/images/IMG_20200118.jpeg)

2. 查看显卡

```
lspci -k | grep -A 2 -i "VGA"
```

3. 查看显卡驱动型号

```
sudo ubuntu-drivers devices
```

4. 删除显卡驱动
```
sudo apt-get autoremove --purge nvidia-*
``` 

5. 重启电脑
```
sudo reboot
```

6. 查看可可安装显卡驱动
```
ubuntu-drivers list
```

7. 安装显卡驱动
 ```
 sudo apt-get install nvidia-*
 ```