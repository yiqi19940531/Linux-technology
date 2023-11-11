## Centos7 脚本实现内核版本升级
```bash
#!/bin/bash

# 确保脚本以root权限运行
if [ "$EUID" -ne 0 ]
  then echo "---请以root用户运行此脚本---"
  exit
fi

# 安装ELRepo公钥
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

# 安装ELRepo仓库
rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm

# 启用ELRepo中的Kernel仓库
yum --enablerepo=elrepo-kernel install kernel-lt -y # 安装最新稳定版本内核
# yum --enablerepo=elrepo-kernel install kernel-lt -y # 安装长期支持版本内核

# 查看可用的内核
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg

# 更新GRUB配置，设置默认启动项为新内核
# 这里假设新内核是第一个条目，即0。如果不是，需要手动修改数字
grub2-set-default 0

# 重新生成GRUB配置
grub2-mkconfig -o /boot/grub2/grub.cfg

# 查看默认启动内核的索引
echo
echo "---默认启动内核的索引: ---"
grub2-editenv list

# 列出所有可用的内核排序
echo
echo "---列出所有可用的内核排序: ---"
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg

# 输出提示重启
echo
echo "---内核升级完成。请重启系统。---"

# 包含自动重启机器步骤
# reboot
```
