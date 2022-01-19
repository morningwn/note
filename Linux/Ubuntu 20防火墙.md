# Ubuntu 20防火墙

默认安装了ufw，如果没有安装的话，使用如下命令安装：

```shell
sudo apt install ufw
```

常用的命令：

```sh
# 查看防火墙的状态
sudo ufw status verbose

# 开启/关闭防火墙
sudo ufw enable/disable
# 查看当前防火墙的规则
sudo ufw status
# 重置防火墙
sudo ufw reset

# 使用外来访问默认的拒绝/允许规则
sudo ufw default deny/allow

# 允许/拒绝 20端口，后面可以加上/tcp或/udp
sudo ufw allow/deny 20

# 批量打开7100到7200的端口，这个时候必须指定tcp或udp
sudo ufw allow 7100:7200/tcp

# 允许/拒绝 对应的协议
sudo ufw allow/deny http

# 删除对应的规则
sudo ufw delete allow http

# 允许/拒绝特定的ip
sudo ufw allow/deny from 192.168.254.254

# 允许指定ip地址访问本机对应的端口
sudo ufw allow from 64.63.62.61 to any port 22
```

[文档](https://manpages.ubuntu.com/manpages/focal/man8/ufw.8.html)
