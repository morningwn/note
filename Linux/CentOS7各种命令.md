# CentOS7各种命令

## 常用

```shell
# 查看本机IP
ifconfig

# 开启服务
systemctl start 服务名
# 关闭服务
systemctl stop 服务名
# 重启服务
systemctl restart 服务名

# 设置开启自启动
systemctl enable 服务名
# 关闭开机自启动
systemctl disable 服务名
```

## 防火墙相关

以下仅针对firewalld

```sh
# 重启
systemctl restart firewalld.service

# 查看开启的端口

# 开启端口，仅临时有效
# 注意，在设置完成之后一定要重启使用这个端口的服务，不然不生效
firewall-cmd --zone=public --add-port=端口/协议

# 开启端口，永久有效
firewall-cmd --zone=public --add-port=端口/协议 --permanent

# 查询对应端口是否开启
firewall-cmd --query-port=端口/协议

# 列出所有的端口
firewall-cmd --list-ports

# 刷新
firewall-cmd --reload

# 添加允许的协议 
firewall-cmd --add-service=协议 --permanent

# 列出所有的zone
firewall-cmd --list-all-zones 

# 查询默认的zone
firewall-cmd --get-default-zone 

# 列出所有活跃的zone
firewall-cmd --get-active-zones
```

