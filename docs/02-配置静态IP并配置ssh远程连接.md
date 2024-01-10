## 配置静态IP并配置ssh远程连接

### 1，修改使用root用户操作

```shell
# 切换root用户
sudo su root
# 设置root用户密码
sudo passwd root
```

### 2，配置静态IP

```shell
# 配置文件位置
cd /etc/netplan
# 00-installer-config.yaml、01-network-manager-all.yaml两个文件，开头的数字00或01等, 有优先级的作用
00-installer-config.yaml
01-network-manager-all.yaml
```

00-installer-config.yaml原始内容（注释中的 `subiquity` 是服务安装程序的名称）

```shell
# This is the network config written by 'subiquity'
network:
  ethernets: {}
  version: 2
```

01-network-manager-all.yaml原始内容

```shell
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
```

网络工具命令

```shell
# Server版用netpla配置systemd-networkd
systemctl status systemd-networkd

# Desktop版用netplan配置NetworkManager
systemctl status NetworkManager
```



Desktop版本配置（修改命令：sudo vi 01-network-manager-all.yaml）

```shell
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens33:  # 网口名称
      dhcp4: false    # 关闭dhcp
      dhcp6: false
      addresses: [192.168.8.113/24]  #静态ip地址
      # gateway4: 192.168.8.1     # 网关
      routes:  # 在ubuntu22.04中，gateway4已弃用，替代的参数为：routes，另外search也不再使用
      - to: default
        via: 192.168.8.1
      nameservers:
        addresses: [8.8.8.8,114.114.114.114]  # DNS地址
```



### 3，修改完毕配置生效命令

```shell
sudo netplan apply
```

执行完毕之后重启系统！



### 4，配置ssh远程连接

```shell
# 更新软件源
sudo apt update && apt-get upgrade -y
# 安装ssh服务
sudo apt-get install ssh
# 启动ssh服务
sudo systemctl start ssh
# 查看ssh状态
sudo systemctl status ssh
# 设备开机自启ssh
sudo systemctl enable ssh
```

tips：配置使用root用户远程登陆

/etc/ssh/sshd_config找到如下代码，下面增加一行配置

```shell
#LoginGraceTime 2m
#PermitRootLogin prohibit-password
PermitRootLogin yes    # 增加的配置
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

修改完毕之后重启ssh服务

```shell
# 重启ssh服务
sudo systemctl restart ssh
# 查看ssh状态
sudo systemctl status ssh
```



### 附录1：netplan命令

```shell
sudo netplan generate # 生成与后端管理工具对应的配置；
sudo netplan apply # 应用配置，必要时重启管理工具；
sudo netplan --debug apply # 调试,返回错误信息；
sudo netplan try # 在配置得到确认之后才应用，如果配置存在错误，则回滚，类似test；
sudo netplan get # 获取当前 netplan 配置；
sudo netplan set # 修改当前 netplan 的配置。
```

