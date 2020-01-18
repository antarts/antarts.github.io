## 局域网搭建JupyterHub环境

### 概述

本文所使用的系统环境 Ubuntu 19.04 个人电脑，建立用户隔离的 JupyterLab Web 环境。这样可以在笔记本上使用台式电脑上的 NVIDIA GPU。

### 准备工作

1. 在电脑安装 Ubuntu 19.04 系统。

2. 更新电脑上的软件包，在终端执行：

```
sudo apt update
sudo apt upgrade --auto-remove
```

3. 使用 NVIDIA GPU ，需安装显卡驱动。

图形界面安装：

a. 打开 “软件和更新” 系统应用;

b. 在该应用的 “Ubuntu 软件” 选项卡中，选中 "设备的专有驱动(restricted)" 选项;

c. 在该应用的 “附加驱动” 选项卡中，待 “搜索可用驱动” 完毕后，选择 NVIDIA Corporation 下最新的长期支持版驱动版本。

### 全局安装 JupyterHub

```
sudo -H pip3 install -U jupyterhub
```

### 安装 configurable-http-proxy

JupterHub 在启动时，HTTP通信 需要 configurable-http-proxy 进行反向代理。

1. 先安装 node 版本管理器 npm：

```
sudo apt install nodejs npm
```

2. 用 npm 全局安装 configurable-http-proxy 。
```
sudo npm install -g configurable-http-proxy
```

### 配置 JupyterHub

1. 生成默认配置文件

生成默认配置文件 jupterhub_config.py ,将它移动到系统目录，如 /etc/jupyterhub ，并重命名为 config.py ：
```
jupterhub --generate-config
sudo mkdir -p /etc/jupterhub
sudo cp jupyterhub_config.py /etc/jupyterhub/config.py
```

2. 修改配置文件

jupyterhub_config.py ：

``` python
# 让 JupyterHub 以目标用户的身份登录 bash，然后运行 jupyterhub-singleuser，由这个进程此启动用户环境中的 JupyterLab。
c.LocalProcessSpawner.shell_cmd = ["bash", "-l", "-c"] 

# 使用 JupyterLab 而非 Jupyter notebook。
c.Spawner.default_url = "/lab" 

#设置 JupyterHub Web 服务的 HTTP 地址。
c.JupyterHub.ip = "0.0.0.0" 

# 不用确保运行 JupyterLab 的环境处于 Kenerl 在列表中。
c.Spawner.args = ["--KernelSpecManager.ensure_native_kernel=False"] 

# 白名单
c.Authenticator.whitelist = {'youracount'}  

# 管理员用户
c.Authenticator.admin_users = {'youracount'}  

# 白名单里的账户在系统里不存在的话就会自动在系统里新建这些用户
c.LocalAuthenticator.create_system_users = True  
```

### 后台运行 JupyterHub

编辑以下文本到 systemd 配置文件 jupyterhub.service
```
[Unit]
Description=Jupyterhub service
After=syslog.target network.target

[Service]
ExecStart=/usr/local/bin/jupyterhub -f /etc/jupyterhub/config.py

[Install]
WantedBy=multi-user.target
```

然后将它复制到 systemd 系统配置目录，并修改权限：
```
sudo cp jupyterhub.service /etc/systemd/system
sudo chmod 644 /etc/systemd/system/jupyterhub.service
```

最后，刷新服务列表并启动该服务：
```
sudo systemctl daemon-reload
sudo systemctl start jupyterhub
```

如果需要该服务自动运行，执行：
```
sudo systemctl enable jupyterhub
```

### 局域网访问 JupyterHub
用登录 Ubuntu 系统的用户名和密码登录即可。

---
### 参考文献

1. [单机多用户 JupyterLab 环境搭建](https://gist.github.com/tanbro/a94bfa4a552381f599e7e6b551ccadcf)
2. [配置 jupyterhub](https://www.brothereye.cn/ubuntu/431/)