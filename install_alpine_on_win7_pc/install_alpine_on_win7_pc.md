# install alpine on win7 pc(deprecated)

(**deprecated**) 原因: win 7 的 MBR 可能有问题. 看以后能不能转成UEFI.
---
安装 alpine linux 到 ASUS X450V, 并和win7共存.
这是一篇 step-by-step的教程(当然主要目的是记录过程而不是给别人看, 我自己看得懂就行).

# 参考
1. https://docs.alpinelinux.org/user-handbook/0.1a/Installing/manual.html
2. https://wiki.alpinelinux.org/wiki/Wi-Fi
3. https://wiki.alpinelinux.org/wiki/Setting_up_disks_manually
4. https://wiki.alpinelinux.org/wiki/Alpine_setup_scripts#setup-disk
5. https://wiki.archlinux.org/title/Fdisk
6. https://wiki.archlinux.org/title/Dual_boot_with_Windows#Windows_before_Linux

# 前置准备
1. 制作 usb引导盘.
2. 查看电脑是 uefi 还是 legacy 启动. (结果是MBR.)
# 步骤

## 进入 usb
1. 插好usb启动盘, 重启, 按住 esc 键3秒, 弹出启动引导选择蓝色 gtk 框, 选 `usb`.
2. 进如 linux usb, 先看到 grub 画面, 之后就是 alpine usb, 先 login:
`alpine login: `, 这里以 root 用户登录.

## alpine 引导盘操作
参考 上面 `# 参考 -1` 和 `# 参考 -3/4` 的步骤来.

### 网络
在配置系统的其余部分之前, 您应该设置网络. 这里我选择 "无线网络".

#### 无线网络
- `alpine:~# setup-interfaces` 来 使用*无线网络*继续安装. 提示:
```
Available interface are: eht0 wlan0.
Enter '?' for help on bridges, bonding and vlans.
Which one do you want to initialize? (or '?' or 'done') [eth0]
```
这里输入 `wlan0`. 之后提示:
```
Available wireless networks (scanning):

...
...
Type the wireless network name to connect to:
```
这里在上面一串名字里找到你的无线网络, 输入你要连接的*无线网络名*. 之后提示:
```
Type the "..." network Pre-shared Key (will not echo):
```
这里输入*密码*, 输入过程中是不会显示的. 完成之后弹出两行 OK 的提示. 
**tip**(重要):这里设置的无线网络跟重启之后的系统的无线网络**无关**! 所以必须之后记得输入: `rc-update add wpa_supplicant boot`, 建议弄个比较完整的解决方案, 比如`wpa_cli`. 详细情况查看 alpine wiki 的 wifi 一章: https://wiki.alpinelinux.org/wiki/Wi-Fi(在安装后再看.)

#### DHCP
**tip**: 下一节的 `dhcp` 在*无线网络*下**不要**选择.

#### Static IP
之后提示:
```
Ip address for wlan0? (or 'dhcp', 'none', '?') [dhcp]
```
这里输入 `?`, 查看输入示范, 参考示范, 输入这个设备interface(wlan0)的 *static ip* 即 `192.168.0.185`. 之后依次按照提示输入 *Netmask* 和 *Gateway*:
```
Netmask? [255.255.255.0]
Gateway? (or 'none') [none]
Configuration for wlan0:
  type=static
  address=192.168.0.185
  netmask=255.255.255.0
  gateway=192.168.0.1
Available interfaces are: eth0.
Enter '?' for help on bridges, bonding and vlans.
Which one do you want to initialize? (or '?' or 'done') [eth0]
```

#### 有线网络eth0
这里设置 `eth0`, 直接回车, 无需输入.
```
Which one do you want to initialize? (or '?' or 'done') [eth0]
Ip address for eth0? (or 'dhcp', 'none', '?') [dhcp]
```
之后提示:
```
Do you want do any manual network configuration? (y/n) [n]
```
输入 n, 告一段落.
之后可以检查 `/etc/network/interfaces` 文件.

#### 域名系统DNS
因为没有设置 DHCP, 所以 DNS 必须设置. 这是通过编辑`/etc/resolv.conf`文件来完成的. 这里使用 `setup-dns` 来设置. 
**note**: `setup-dns` 要求一个 domain name, 可以留空, 因为它是可选的.
```
alpine:~# setup-dns
DNS domain name? (e.g 'bar.com')
DNS nameserver(s)? 192.168.0.1
``` 
忽略 `DNS domain name`, `DNS nameserver` 后写 192.168.0.1, 之后再在/etc/resolv.conf 加几句
```
nameserver 192.168.31.1
# gonggong DNS
# aliyun DNS
nameserver 223.5.5.5
nameserver 223.6.6.6
```

**最后**, 运行`rc-service networking start`来**开启网络配置**.
如果需要，您还可以将其设置为**在引导期间加载**: `rc-update add networking boot`.

### root password
输入`passwd` 设置 root 密码, 重复两次.

### ssh
运行`setup-sshd`, `Which SSH server? ('openssh', 'dropbear', or 'none') [openssh]`, 选 openssh,
然后修改sshd_config `vi /etc/ssh/sshd_config`, 在 `Authentication` 里的 `PermitRootLogin prohibit-password` 下面加一句 `PermitRootLogin yes`,这样可以 root 登录 ssh, 保存.
(**deprecated**)分别使用`rc-service openssh start`和`rc-update add openssh`开启 openssh, 自启动 openssh.
(两个命令当时运行时输出: `* rc-service: service 'openssh' dose not exist`,
所以把 openssh 替换为 sshd, 尝试成功)
为保险, 运行 `rc-service sshd restart`, `rc-update add sshd boot`.

发现win10 ssh 连 alpine 连不上:
```
$ ssh -i id_ecdsa.pub root@192.168.0.185 -p 22
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:..... .
Please contact your system administrator.
Add correct host key in /c/Users/ash_258/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /c/Users/ash_258/.ssh/known_hosts:16
Host key for 192.168.0.185 has changed and you have requested strict checking.
Host key verification failed.
```

 根据 https://dalanzg.github.io/tips-tutorials/posts/2016/06/03/how-to-fix-warning-about-ecdsa-host-key-when-ssh-connection/ 的说法, 需要删除ssh缓存: 
```
ssh-kengen -R 192.168.0.185
# Host 192.168.0.185 found: line 16
# Host 192.168.0.185 found: line 25
/c/Users/ash_258/.ssh/known_hosts updated.
Original contents retained as /c/Users/ash_258/.ssh/known_hosts.old
```
再次尝试发现ssh连上了, **之后开始以 ssh 来完成安装**.
在win10上命令行操作, 连接ssh: `ssh root@192.168.0.185 -p 22`

### 键盘布局和 hostname
- `alpine:~# setup-keymap us us` 设置*键盘布局*, 弹出两行 `ok` 的提示.
- `alpine:~# setup-hostname alpine` 设置 *hostname*, 没有弹出提示, 因为这个命令不会填充 `/etc/hosts`, 即本地的 dns 缓存. 
要*应用hostname*即主机名, 运行: 
`rc-service hostname restart` 或 `/etc/init.d/hostname restart`.

### 时区timezone
在 musl 上, 时区由 TZ 环境变量定义. `Area/SubArea` 是一个标准定义的样式.
可以通过安装 `tzdata` 包来获得 `/usr/share/zoneinfo` 目录. 查看这个目录来查看可用区域, 以及可用子区域(在你选择的可用区域内). 一旦选择, 就必须二选一: 保持 `tzdata` 包已安装, 或者复制你选择的文件到 `/etc/zoneinfo` 里.
**note**: 推荐保持包已安装.

安装 `tzdata` 包, 
```
alpine:~# apk add tzdata
(1/1) Installing tzdata (2021a-r0)
Executing busybox-1.32.1-r6.trigger
OK: 18 MiB in 28 packages
```
之后输入`setup-timezone`, 之后依次输入`Asia`、`Shanghai`.
之后输入命令:
```
install -Dm 0644 /usr/share/zoneinfo/Asia/Shanghai /etc/zoneinfo/Asia/Shanghai
```
最后, 把 TZ 环境变量添加到系统中.
`export TZ='Asia/Shanghai'` (此步骤主要用于将新设置传播到当前会话.)
`echo "export TZ='$TZ'" >> /etc/profile.d/timezone.sh`
**warning**: setup-timezone, 就像现在一样, 不会设置TZ环境变量. 相反, 它将假装时区数据是本地时间样式的文件. 这是一个技术差异, 您可能不需要担心, 但由于这种差异, 建议您**完全手动**执行此步骤.

### 存储库repo
存储库配置在`/etc/apk/repositories`, 有效的签名密钥在`/etc/apk/keys/`. usb引导介质应带有有效的预配置密钥, 但没有外部存储库(external repo). 目前, 您可以在[mirrors.alpinelinux.org](https://mirrors.alpinelinux.org/)查看可用镜像列表及其状态.
**note**: 不用担心镜像中缺少"https"——所有包都经过签名，所以只要你不添加任何不可信的密钥，你的包管理器将拒绝安装任何非法包.

这里因为网络配置好了, 采用 `setup-apkrepos`, 获取有效存储库的列表，并为您提供它们之间的选择（以及“随机”等选项）.
多添加几个, 这里我把 上交大的放最前, 还有 tuna, ustc, aliyun, dl-cdn.alpinelinux.org, dl-4.alpinelinux.org, dl-5.alpinelinux.org, 全部注释掉, 之后添加 latest-stable 源.
(**deprecated**)(或者手动添加: 示例
```
echo "http://mirrors.sjtug.sjtu.edu.cn/v3.13/main" >> /etc/apk/repositories
echo "http://mirrors.sjtug.sjtu.edu.cn/v3.13/community" >> /etc/apk/repositories
```
)
**tip**: 去 sjtug mirror repo 查看 3.13, 竟然没有 os-prober 的包, 于是在第一组repos下**添加** latest-stable 源.

因为我制作usb引导盘用的 alpine iso 文件版本比较旧, 所以之后运行
```
# apk update
# apk upgrade
```
如果中途中断过最好运行一次`# apk fix *`
### 提前安装packages
`# apk add vim os-prober sfdisk util-linux e2fsprogs e2fsprogs-extra grub grub-bios lsblk ntfs-3g ntfs-3g-progs`
(grub-bios 换成 grub-efi)
安装完之后 `# vim /etc/apk/world`看一下包的列表.
### NTP
确保您的时钟正确可能会很有用。这可以通过使用 NTP 守护程序来实现。一些常见的是chronyd和openntpd。

**warning**: 目前，chronyd被窃听。有问题的错误主要是装饰性的，但它可能会让新用户感到震惊。因此，暂时**建议您使用setup-ntp脚本并选择busybox**。
```
alpine:/media/usb/apks/x86_64# setup-ntp
Which NTP client to run? ('busybox', 'openntpd', 'chrony' or 'none') [chrony] busybox 
 * service ntpd added to runlevel default
  * Caching service dependencies ...                                          [ ok ]
  * Starting busybox ntpd ...                                                 [ ok ]
```
### 对磁盘进行分区
感觉 `setup-disk` 不太适合这种双系统, 还是手动吧.

#### (deprecated)Parted
用 Parted 手动分区. 先安装 parted.
**tip**: Parted 分区和fdisk不同, 会*即刻生效*! 请小心操作！
```
alpine:~# apk add parted
fetch http://mirrors.sjtug.sjtu.edu.cn/latest-stable/main/x86_64/APKINDEX.tar.gz
ERROR: http://mirrors.sjtug.sjtu.edu.cn/latest-stable/main: temporary error (try again later)
WARNING: Ignoring http://mirrors.sjtug.sjtu.edu.cn/latest-stable/main: No such file or directory
fetch http://mirrors.sjtug.sjtu.edu.cn/latest-stable/community/x86_64/APKINDEX.tar.gz
ERROR: http://mirrors.sjtug.sjtu.edu.cn/latest-stable/community: temporary 
error (try again later)
WARNING: Ignoring http://mirrors.sjtug.sjtu.edu.cn/latest-stable/community: No such file or directory
(1/5) Installing libblkid (2.36.1-r1)
(2/5) Installing device-mapper-libs (2.02.187-r1)
(3/5) Installing readline (8.1.0-r0)
(4/5) Installing libuuid (2.36.1-r1)
(5/5) Installing parted (3.3-r1)
Executing busybox-1.32.1-r6.trigger
OK: 25 MiB in 42 packages
```
通过检查`/sys/firmware/efi`文件夹是否存在来确定您当前是否使用 UEFI 引导。
或者可以使用以下代码段来获得直接答案： `test -d /sys/firmware/efi && echo UEFI || echo BIOS`, 输出 `UEFI`, 但是先后用 `fdisk -l` 和 `parted -l`测试, 后者输出显示是 msdos的分区:
```
alpine:/# parted -l
Model: ATA HGST HTS545050A7 (scsi)
Disk /dev/sda: 500GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type      File system  Flags
 1      32.3kB  85.9GB  85.9GB  primary   ntfs         boot
 2      85.9GB  225GB   140GB   primary   ntfs
 3      225GB   365GB   140GB   primary   ntfs
 4      365GB   500GB   135GB   extended
```
故详细分区操作步骤参考 [BIOS+ms-dos方案](https://docs.alpinelinux.org/user-handbook/0.1a/Installing/manual.html#_bios_ms_dos)
(后来发现这个教程是单磁盘单系统的, 不适合, 自己摸索吧)

##### 规划分区

采用 **BIOS+MS_DOS方案**:

| 分区号 | 开始 | 结束 | Mount | Flags |
--------|------|------|-------|-------|
| /dev/sda1 | - | - | /mnt/media/win7 | bootable |
| /dev/sda5 | - | +1G | /mnt/boot | - |
| /dev/sda6 | - | +4G | swap | - |
| /dev/sda7 | - | - | /mnt | - |

#### fdisk
参考 Arch Wiki: fdisk 一章: https://wiki.archlinux.org/title/Fdisk

1. 列出 sda的分区信息: `# fdisk -l /dev/sda`
2. 备份并转存分区表, 然后检查一下得到的分区表备份格式是否正确: 
`# apk add sfdisk`
`# sfdisk -d /dev/sda > sda.dump`
3. 进入 fdisk交互模式: 
```
alpine:~# fdisk /dev/sda

The number of cylinders for this disk is set to 60801.
There is nothing wrong with that, but this is larger than 1024,
and could in certain setups cause problems with:
1) software that runs at boot time (e.g., old versions of LILO)
2) booting and partitioning software from other OSs
   (e.g., DOS FDISK, OS/2 FDISK)

Command (m for help):
```
4. n 创建新分区(先创建 /boot , 之后 /swap, 最后 /): 

- /boot:
```
Command (m for help): n
First sector (713062461-976771071, default 713062461): 
Using default value 713062461
Last sector or +size{,K,M,G,T} (713062461-976771071, default 976771071): +1G
```

- /swap:
```
Command (m for help): n
First sector (715159676-976771071, default 715159676): 
Using default value 715159676
Last sector or +size{,K,M,G,T} (715159676-976771071, default 976771071): +4G
```
- /:
```
Command (m for help): n
First sector (723548347-976771071, default 723548347): 
Using default value 723548347
Last sector or +size{,K,M,G,T} (723548347-976771071, default 976771071): 
Using default value 976771071
```
5. 给创建好的分区修改一些参数, 如文件系统类型等:
- 先用 p 命令查看分区表:
```
Command (m for help): p
Disk /dev/sda: 466 GB, 500107862016 bytes, 976773168 sectors
60801 cylinders, 255 heads, 63 sectors/track
Units: sectors of 1 * 512 = 512 bytes

Device  Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
/dev/sda1 *  0,1,1       1023,254,63         63  167782859  167782797 80.0G  7 HPFS/NTFS
/dev/sda2    1023,254,63 1023,254,63  167782860  440419926  272637067  130G  7 HPFS/NTFS
/dev/sda3    1023,254,63 1023,254,63  440419927  713061089  272641163  130G  7 HPFS/NTFS
/dev/sda4    1023,254,63 1023,254,63  713062398  976771071  263708674  125G  5 Extended
/dev/sda5    1023,254,63 1023,254,63  713062461  715159612    2097152 1024M 83 Linux
/dev/sda6    1023,254,63 1023,254,63  715159676  723548283    8388608 4096M 83 Linux
/dev/sda7    1023,254,63 1023,254,63  723548347  976771071  253222725  120G 83 Linux
```
- 再用 t 命令修改指定分区类型:
```
Command (m for help): t
Partition number (1-7): 6
Hex code (type L to list codes): L

 0 Empty                  1c Hidden W95 FAT32 (LBA) a0 Thinkpad hibernation  
 1 FAT12                  1e Hidden W95 FAT16 (LBA) a5 FreeBSD
 4 FAT16 <32M             3c Part.Magic recovery    a6 OpenBSD
 5 Extended               41 PPC PReP Boot          a8 Darwin UFS
 6 FAT16                  42 SFS                    a9 NetBSD
 7 HPFS/NTFS              63 GNU HURD or SysV       ab Darwin boot
 a OS/2 Boot Manager      80 Old Minix              af HFS / HFS+
 b Win95 FAT32            81 Minix / old Linux      b7 BSDI fs
 c Win95 FAT32 (LBA)      82 Linux swap             b8 BSDI swap
 e Win95 FAT16 (LBA)      83 Linux                  be Solaris boot
 f Win95 Ext'd (LBA)      84 OS/2 hidden C: drive   eb BeOS fs
11 Hidden FAT12           85 Linux extended         ee EFI GPT
12 Compaq diagnostics     86 NTFS volume set        ef EFI (FAT-12/16/32)
14 Hidden FAT16 <32M      87 NTFS volume set        f0 Linux/PA-RISC boot
16 Hidden FAT16           8e Linux LVM              f2 DOS secondary
17 Hidden HPFS/NTFS       9f BSD/OS                 fd Linux raid autodetect
1b Hidden Win95 FAT32
Hex code (type L to list codes): 82
Changed system type of partition 6 to 82 (Linux swap)
```

6. p 检查分区表无误后, 输入 w 命令保存修改:
```
Command (m for help): p
Disk /dev/sda: 466 GB, 500107862016 bytes, 976773168 sectors
60801 cylinders, 255 heads, 63 sectors/track
Units: sectors of 1 * 512 = 512 bytes

Device  Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
/dev/sda1 *  0,1,1       1023,254,63         63  167782859  167782797 80.0G  7 HPFS/NTFS
/dev/sda2    1023,254,63 1023,254,63  167782860  440419926  272637067  130G  7 HPFS/NTFS
/dev/sda3    1023,254,63 1023,254,63  440419927  713061089  272641163  130G  7 HPFS/NTFS
/dev/sda4    1023,254,63 1023,254,63  713062398  976771071  263708674  125G  5 Extended
/dev/sda5    1023,254,63 1023,254,63  713062461  715159612    2097152 1024M 83 Linux
/dev/sda6    1023,254,63 1023,254,63  715159676  723548283    8388608 4096M 82 Linux swap
/dev/sda7    1023,254,63 1023,254,63  723548347  976771071  253222725  120G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table
```

#### 格式化分区
参考 Arch Wiki 上 Installation guide 一章 Format the partitions 一节: https://wiki.archlinux.org/title/Installation_guide#Format_the_partitions 以及 alpine wiki 上的 partition.

1. /dev/sda5(/boot)格式化为 ext4: 
`# apk add util-linux e2fsprogs`
```
alpine:~# mkfs.ext4 /dev/sda5
mke2fs 1.45.7 (28-Jan-2021)
/dev/sda5 alignment is offset by 1536 bytes.
This may result in very poor performance, (re)-partitioning suggested.
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: {UUID}
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```

2. /dev/sda6(/swap)格式化为 swap:
```
alpine:~# mkswap /dev/sda6
mkswap: warning: /dev/sda6 is misaligned
Setting up swapspace version 1, size = 4 GiB (4294963200 bytes)
no label, UUID={UUID}
```
3. /dev/sda7(/)格式化为 ext4:
```
alpine:~# mkfs.ext4 /dev/sda7
mke2fs 1.45.7 (28-Jan-2021)
/dev/sda7 alignment is offset by 2560 bytes.
This may result in very poor performance, (re)-partitioning suggested.
Creating filesystem with 31652840 4k blocks and 7913472 inodes
Filesystem UUID: {UUID}
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done
Writing inode tables: done
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done
```

#### 挂载文件系统
参考 Arch Wiki 上 Installation guide 一章 Mount the file systems 一节: https://wiki.archlinux.org/title/Installation_guide#Mount_the_file_systems
以及 (**important**)Alpinelinux Wiki 上 https://wiki.alpinelinux.org/wiki/Dualbooting#Installing_Alpine_on_an_HDD_partition.

1. 把 root volume 挂载到 /mnt: `# mount -t ext4 /dev/sda7 /mnt`
2. 把 boot volume 挂载到 /mnt/boot: 
`# mkdir /mnt/boot`
`# mount -t ext4 /dev/sda5 /mnt/boot`
3. 用 swapon 命令启用 swap volume:
`# swapon /dev/sda6`

#### setup-disk 并加上自定义参数安装到硬盘上
按照 alpine handbook(0.1b)的 安装 -> 半自动安装 一章, 硬盘分区 一节: https://docs.alpinelinux.org/user-handbook/0.1a/Installing/manual.html#_partitioning_your_disk
以及 alpinelinux wiki 的 Alpine_setup_scripts 一章,  setup-disk 一节: https://wiki.alpinelinux.org/wiki/Alpine_setup_scripts#setup-disk
以及(**important**) alpinelinux wiki 的 Setting up disks manually 一章: https://wiki.alpinelinux.org/wiki/Setting_up_disks_manually[^1]
[^1]:2022——01——25 01：59：00最后编辑地址

`setup-disk` 可以搭配一些 *参数* 和 *环境变量* 来达到自定义安装.
1. 查看现有环境变量
```
alpine:~# export
export CHARSET='UTF-8'
export HOME='/root'
export LANG='C.UTF-8'
export LC_COLLATE='C'
export LOGNAME='root'
export MAIL='/var/mail/root'
export OLDPWD='/lib/firmware'
export PAGER='less'
export PATH='/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin' 
export PS1='\h:\w\$ '
export PWD='/'
export SHELL='/bin/ash'
export SHLVL='1'
export SSH_CLIENT='192.168.0.101 61188 22'
export SSH_CONNECTION='192.168.0.101 61188 192.168.0.185 22'
export SSH_TTY='/dev/pts/0'
export TERM='xterm-256color'
export TZ='Asia/Shanghai'
export USER='root'
```
2. 添加环境变量
```
alpine:~# export BOOTLOADER='grub'

alpine:~# export BOOTFS='ext4'

alpine:~# BOOT_SIZE='1024'

alpine:~# export SYSROOT='/mnt'

alpine:~# export MBR='/usr/share/syslinux/mbr.bin'

alpine:~# export DISKLABEL='dos'
```

然后参考 ArchWiki 上的 *dual boot* 章节 https://wiki.archlinux.org/title/Dual_boot_with_Windows#Windows_before_Linux,

也需参考 alpinelinux wiki 上的 *手动设置磁盘* 一章的 *手动分区* 一节: https://wiki.alpinelinux.org/wiki/Setting_up_disks_manually#Manual-partitioning
以及 https://wiki.alpinelinux.org/wiki/Dualbooting#Install_Alpine, 因为安装的版本3.12, 比 2.2.3要新, 所以看 *In Alpine 2.2.3 or newer* 章节.

运行 `setup-disk -m sys /mnt`:
```
alpine:~# setup-disk -m sys /mnt
Installing system on /dev/sda7:
grub-install: error: /usr/lib/grub/i386-pc/modinfo.sh doesn't exist. Please specify --target or --directory.
100% ██████████████████████████████████████████████████████████████████████████████████████████= 
=> initramfs: creating /boot/initramfs-lts
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-lts
Found initrd image: /boot/initramfs-lts
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
done
You might need fix the MBR to be able to boot
```

**second**:
```
alpine:~# setup-disk -m sys /mnt
Installing system on /dev/sda7:
grub-install: error: /usr/lib/grub/x86_64-efi/modinfo.sh doesn't exist. Please specify --target or --directory.
install: can't stat '/mnt/boot/efi/EFI/alpine/grubx64.efi': No such file or directory
100% ██████████████████████████████████████████████████████████████████████████████████████████= 
=> initramfs: creating /boot/initramfs-lts
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-lts
Found initrd image: /boot/initramfs-lts
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
done
You might need fix the MBR to be able to boot
```
然后修复上述看到的错误:
1. `Warning: os-prober will not be executed to detect other bootable partitions.`
这表示 os-prober 没有探测到电脑的 windows 系统. 参考 arch wiki 的 https://wiki.archlinux.org/title/GRUB#Detecting_other_operating_systems 以及 https://wiki.archlinux.org/title/GRUB#Generate_the_main_configuration_file 
上面写的: 应该先挂载windows7的 /bootmgr, 他就相当于虚拟的系统盘. 然后以 root 身份运行 os-prober, 检查输出看是否探测到 windows7.

2. `grub-install: error: /usr/lib/grub/i386-pc/modinfo.sh doesn't exist. Please specify --target or --directory.`
运行(因为之后看 https://wiki.archlinux.org/title/GRUB/Tips_and_tricks#Generate_core.img_alone 引入了 `--grub-setup` 和 `--debug`参数, 所以输出信息很长, 没有贴出来):
```
(**deprecated**)alpine:~# grub-install --target=i386-pc --directory=/mnt/usr/lib/grub/i386-pc/ --boot-directory=/mnt/boot /dev/sda
Installing for i386-pc platform.
Installation finished. No error reported.
```
`alpine:~# grub-install --target=i386-pc --directory=/mnt/usr/lib/grub/i386-pc/ --boot-directory=/mnt/boot --grub-setup=/bin/true --debug /dev/sda`