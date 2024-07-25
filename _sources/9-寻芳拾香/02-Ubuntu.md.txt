# Ubuntu

## 快速连接服务器

**起源：**
 本地开发服务器较多，Linux 也有类似 XShell 的工具，但仍然不如直接在终端使用 ssh 方便，但是直接使用 ssh 又需要记住服务器 IP、用户名和密码，根本记不住

**解决方案**

- 使用 `ssh` config 设置服务器别名
- 使用 `sh-copy-id` 复制本机公钥到远程服务器

**使用示例**

```bash
➜  ~ ssh had1                     
Last login: Wed Jan 17 17:23:54 2024 from 172.16.11.96
[root@had1 ~]#
```

**配置方式**

1. 在 `.ssh/config` 文件中添加如下内容：

```bash
Host had3
    HostName 172.16.11.41
    User root
```

2. 复制公钥到远程服务器

```shell
ssh-copy-id root@172.16.11.41
```

```{note}
有些服务器支持的ssh加密算法与本机的默认算法不匹配，会提示：

➜  ~ ssh-copy-id root@172.16.11.44

/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed<br>
/usr/bin/ssh-copy-id: ERROR: Unable to negotiate with 172.16.11.44 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss<br>

这种情况在`.ssh/config`中指定算法即可
```bash
Host had3
    HostName 172.16.11.44
    User root
    HostkeyAlgorithms +ssh-rsa
    PubkeyAcceptedAlgorithms +ssh-rsa
```

## IDEA 转换大小写失效

- 现象： 选中单词，使用`Ctrl-Shift-U`尝试转换大小写，结果单词变为u
- 原因： 快捷键被 ibus 占用，参考 [这里解决](https://youtrack.jetbrains.com/issue/IDEA-112533/Toggle-Case-Ctrl-Shift-U-not-working-under-Gnome-Linux)

> - 运行 `ibus-setup` 命令
> - 切换到 Emoji 页面
> - 修改或删除 `Ctrl-Shift-U` 快捷键


## 显卡驱动

```bash
# 1. 查看可选驱动
➜  ~ ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:03.1/0000:26:00.0 ==
modalias : pci:v000010DEd00001E84sv00001462sd0000C729bc03sc00i00
vendor   : NVIDIA Corporation
model    : TU104 [GeForce RTX 2070 SUPER]
driver   : nvidia-driver-535-server - distro non-free
driver   : nvidia-driver-535-open - distro non-free
driver   : nvidia-driver-545-open - distro non-free
driver   : nvidia-driver-545 - distro non-free
driver   : nvidia-driver-535-server-open - distro non-free
driver   : nvidia-driver-470-server - distro non-free
driver   : nvidia-driver-535 - distro non-free recommended
driver   : nvidia-driver-450-server - distro non-free
driver   : nvidia-driver-470 - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin

# 2. 安装推荐驱动
➜  ~ sudo apt-get install nvidia-driver-535

# 3. 重启
➜  ~ reboot
```

## ssh 端口转发

```
    -L [bind_address:]port:host:hostport
    -L [bind_address:]port:remote_socket
    -L local_socket:host:hostport
    -L local_socket:remote_socket
```
![alt text](img/image.png)