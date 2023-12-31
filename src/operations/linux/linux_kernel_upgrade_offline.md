# 离线升级

## 一、获取离线升级包

到[elrepo](https://elrepo.org/linux/kernel/el7/x86_64/RPMS/)的网站下载想要的版本

```shell
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-ml-5.16.10-1.el7.elrepo.x86_64.rpm
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-ml-devel-5.16.10-1.el7.elrepo.x86_64.rpm
```

::: warning 注意
如果在目标服务器无法上网情况下也可在自己电脑下载 rpm 包，然后上传到服务器
:::

## 二、升级内核

### 2.1 安装 rpm 包

```shell
yum localinstall -y kernel-lt-4.4.206-1.el7.elrepo.x86_64.rpm \
kernel-lt-devel-4.4.206-1.el7.elrepo.x86_64.rpm
```

### 2.2 查看系统上的所有可用内核

```shell
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
```

```text
0 : CentOS Linux (5.16.10-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-862.11.6.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-514.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-063ec330caa04d4baae54c6902c62e54) 7 (Core)
```

设置新的内核为 grub2 的默认版本
服务器上存在 4 个内核，我们要使用 5.16 这个版本，可以通过 grub2-set-default 0 命令或编辑 /etc/default/grub 文件来设置

#### 2.2.1 通过 <font color='red'>**grub2-set-default 0**</font> 命令设置

其中 0 是上面查询出来的可用内核

```shell
grub2-set-default 0
```

#### 2.2.2 方法 2、编辑 <font color='red'>**/etc/default/grub**</font> 文件

设置 GRUB_DEFAULT=0，通过上面查询显示的编号为 0 的内核作为默认内核：

```shell
vim /etc/default/grub
```

```text
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=0
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=cl/root rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

### 2.3 生成 grub 配置文件并重启

```shell
grub2-mkconfig -o /boot/grub2/grub.cfg
```

```text
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.16.10-1.el7.elrepo.x86_64
Found initrd image: /boot/initramfs-5.16.10-1.el7.elrepo.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-862.11.6.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.11.6.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-514.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-514.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-063ec330caa04d4baae54c6902c62e54
Found initrd image: /boot/initramfs-0-rescue-063ec330caa04d4baae54c6902c62e54.img
done
```

```shell
reboot
```

## 2.4 验证

```shell
uname -r
```

```text
5.16.10-1.el7.elrepo.x86_64
```

## 2.5 删除旧的内核（可选）

### 2.5.1 查看系统中全部的内核

```shell
rpm -qa | grep kernel
```

```text
kernel-3.10.0-514.el7.x86_64
kernel-ml-5.16.10-1.el7.elrepo.x86_64
kernel-tools-libs-3.10.0-862.11.6.el7.x86_64
kernel-tools-3.10.0-862.11.6.el7.x86_64
kernel-3.10.0-862.11.6.el7.x86_64
```

#### 方法一：通过 <font color='red'>**yum remove**</font>删除

```shell
yum remove kernel-3.10.0-514.el7.x86_64 \
kernel-tools-libs-3.10.0-862.11.6.el7.x86_64 \
kernel-tools-3.10.0-862.11.6.el7.x86_64 \
kernel-3.10.0-862.11.6.el7.x86_64
```

#### 方法二：使用 <font color='red'>**yum-utils**</font>工具删除

如果安装的内核不多于 3 个，<font color='red'>**yum-utils**</font> 工具不会删除任何一个。只有在安装的内核大于 3 个时，才会自动删除旧内核。

安装<font color='red'>**yum-utils**</font>

```shell
yum install yum-utils
```

删除旧版本

```shell
package-cleanup --oldkernels
```
