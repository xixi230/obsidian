# Linux 电源管理：挂起与休眠指南

本文档基于 Arch Linux Wiki 整理，用于指导 AI 助手处理 Linux 系统的电源管理、挂起和休眠相关问题。

来源：https://wiki.archlinuxcn.org/wiki/电源管理/挂起与休眠

---

## 睡眠状态类型

### 1. Suspend to idle (S0ix / 现代待机)
- Intel 称为 S0ix，Microsoft 称为"现代待机"
- 内核称为 S2Idle
- 节能效果与 S3 相当，但唤醒时间更短
- Linux 不使用网络唤醒特性，不会像 Windows/macOS 那样有电池消耗问题

### 2. Suspend to RAM (挂起到内存 / S3)
- ACPI 定义的 S3 睡眠状态
- 仅保持 RAM 供电，其他部件断电
- 推荐笔记本在合盖或空闲时自动进入

### 3. Suspend to disk (休眠 / S4)
- ACPI 定义的 S4 睡眠状态
- 将内存状态保存到交换空间后完全关机
- 无待机功耗，恢复较慢

### 4. Hybrid suspend (混合挂起)
- 同时保存到 RAM 和磁盘
- 断电后仍可从磁盘恢复

---

## systemd 睡眠命令

```bash
# 挂起到内存
systemctl suspend

# 休眠到磁盘
systemctl hibernate

# 混合睡眠
systemctl hybrid-sleep

# 先挂起后休眠（推荐）
systemctl suspend-then-hibernate

# 自动选择最佳睡眠方式 (systemd v256+)
systemctl sleep
```

---

## 查看和更改挂起方法

### 查看支持的睡眠状态
```bash
cat /sys/power/mem_sleep
```

输出示例：`[s2idle] shallow deep`
- `s2idle` = suspend-to-idle
- `shallow` = standby
- `deep` = suspend-to-RAM (S3)

### 临时切换到 deep 睡眠
```bash
echo deep > /sys/power/mem_sleep
```

### 永久配置 (方法一：systemd)
创建 `/etc/systemd/sleep.conf.d/mem-deep.conf`:
```ini
[Sleep]
MemorySleepMode=deep
```

### 永久配置 (方法二：内核参数)
添加内核参数：`mem_sleep_default=deep`

### 强制使用 s2idle（固件有问题时）
创建 `/etc/systemd/sleep.conf.d/freeze.conf`:
```ini
[Sleep]
SuspendState=freeze
```

---

## 休眠配置步骤

### 前提条件
1. 创建足够大的 swap 分区或文件
2. 配置 initramfs
3. 指定交换空间位置

### 配置 initramfs (mkinitcpio)

编辑 `/etc/mkinitcpio.conf`，确保 `resume` 钩子在 `udev` 之后：
```bash
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems resume fsck)
```

重新生成 initramfs：
```bash
mkinitcpio -P
```

**注意**：使用 systemd 钩子时不需要额外添加 resume 钩子。

### 指定交换空间位置

#### UEFI 系统 (systemd v255+)
自动处理，无需手动配置。

#### 手动配置内核参数
```
resume=UUID=你的swap分区UUID
# 或
resume=/dev/你的swap设备
```

#### 使用交换文件时
需要额外指定偏移量：
```
resume=UUID=文件系统UUID resume_offset=偏移量值
```

### 获取交换文件偏移量

#### 普通文件系统
```bash
filefrag -v /swapfile | awk '$1=="0:" {print substr($4, 1, length($4)-2)}'
```

#### Btrfs 文件系统
```bash
btrfs inspect-internal map-swapfile -r /swapfile
```

### 立即生效（无需重启）
```bash
# 设置 resume 设备 (major:minor)
echo 8:3 > /sys/power/resume

# 设置偏移量
echo 38912 > /sys/power/resume_offset
```

---

## 休眠镜像大小优化

创建 `/etc/tmpfiles.d/hibernation_image_size.conf`:
```
w    /sys/power/image_size  -    -    -    -   0
```

设为 0 可使镜像尽可能小，适合小交换分区。

---

## 休眠压缩算法 (Linux 6.9+)

支持 `lzo`（默认）和 `lz4`。

### 内核参数方式
```
hibernate.compressor=lz4
```

### 运行时修改
```bash
echo lz4 > /sys/module/hibernate/parameters/compressor
```

---

## zram 与休眠共存

1. 创建普通交换文件用于休眠
2. 配置 zram 并设置更高优先级（如 `pri=100`）
3. systemd 会自动忽略 zram 进行休眠

---

## 禁用 zswap 写回

Linux 6.8+ 支持禁用 zswap 写回，使其像 zram 一样工作同时支持休眠。

安装 `zswap-disable-writeback` (AUR) 或手动配置 systemd 单元。

验证：
```bash
cat /sys/kernel/debug/zswap/written_back_pages
# 应该输出 0
```

---

## 睡眠钩子

### systemd 单元方式（推荐）

挂起前执行：
```ini
# /etc/systemd/system/my-suspend.service
[Unit]
Description=My suspend actions
Before=sleep.target

[Service]
Type=oneshot
ExecStart=/path/to/script.sh

[Install]
WantedBy=sleep.target
```

恢复后执行：
```ini
# /etc/systemd/system/my-resume.service
[Unit]
Description=My resume actions
After=suspend.target hibernate.target

[Service]
Type=simple
ExecStart=/path/to/script.sh

[Install]
WantedBy=suspend.target hibernate.target
```

### 脚本方式

创建可执行脚本 `/usr/lib/systemd/system-sleep/example.sh`:
```bash
#!/bin/sh
case $1/$2 in
  pre/*)
    echo "Going to $2..."
    ;;
  post/*)
    echo "Waking up from $2..."
    ;;
esac
```

---

## 禁用睡眠功能

创建 `/etc/systemd/sleep.conf.d/disable-sleep.conf`:
```ini
[Sleep]
AllowSuspend=no
AllowHibernation=no
AllowHybridSleep=no
AllowSuspendThenHibernate=no
```

---

## 常见问题排查

### 休眠后系统不断电
创建 `/etc/systemd/sleep.conf.d/hibernatemode.conf`:
```ini
[Sleep]
HibernateMode=shutdown
```

### 黑屏问题
- 移除 mkinitcpio MODULES 中的显卡模块
- 移除 `kms` 钩子
- 重新生成 initramfs

### NVIDIA 显卡问题
禁用 `nvidiafb` 模块：
```bash
# /etc/modprobe.d/blacklist.conf
blacklist nvidiafb
```

### Intel 触摸板导致内核恐慌
将 `intel_lpss_pci` 添加到 initramfs MODULES。

### USB 设备阻止挂起
检查日志中的 `failed to suspend` 错误，尝试断开相关 USB 设备。

### AMD A520I/B550I 主板无法唤醒
禁用 GPP0 唤醒：
```bash
echo GPP0 > /proc/acpi/wakeup
```

永久生效需创建 systemd 服务或 udev 规则。

### systemd v256+ 冻结问题
如果出现 `Failed to freeze unit 'user.slice'`，添加 drop-in：
```ini
[Service]
Environment=SYSTEMD_SLEEP_FREEZE_USER_SESSIONS=false
```

---

## 测量挂起功耗

使用 Batenergy 脚本（需要 `bc` 包）记录电池变化到系统日志。

---

## Intel 快速启动技术 (IRST)

- 基于固件的休眠方法
- 需要 SSD 上的特殊分区（大小等于 RAM）
- 分区类型 GUID: `D3BFE2DE-3DAF-11DF-BA40-E3A556D89593`
- 内核模块: `intel_rst`

**警告**：IRST 分区未加密，使用软件磁盘加密时建议禁用。

---

## 相关手册页

- `systemctl(1)`
- `systemd-sleep(8)`
- `systemd-sleep.conf(5)`
- `systemd.special(7)`
- `systemd-hibernate-resume(8)`

---

## 参考链接

- [内核睡眠状态文档](https://docs.kernel.org/admin-guide/pm/sleep-states.html)
- [Arch Wiki 原文](https://wiki.archlinuxcn.org/wiki/电源管理/挂起与休眠)
  额外参考：

  022-12-11 UPDATE: There is a new rather severe caveat to this article. If Secure Boot is enabled and the kernel boots in lockdown mode, hibernation does not work as long as the kernel does not support signed hibernation images.
2022-12-11 更新：这篇文章有一个新的比较严重的注意事项。如果 Secure Boot 已启用且内核以锁定模式启动，只要内核不支持签名挂起镜像，挂起功能就无法正常工作。

Goal and Rationale  目标和理由
Hibernation stores the current runtime state of your machine – effectively the contents of your RAM, onto disk and does a clean shutdown. Upon next boot this state is restored from disk to memory such that everything, including open programs, is how you left it.
挂起功能将您机器当前的运行状态存储到磁盘上——实际上就是 RAM 的内容——然后执行干净关闭。下次启动时，此状态会从磁盘恢复到内存中，以便所有内容（包括打开的程序）都保持您离开时的状态。

Fedora Workstation uses ZRAM. This is a sophisticated approach to swap using compression inside a portion of your RAM to avoid the slower on-disk swap files. Unfortunately this means you don’t have persistent space to move your RAM upon hibernation when powering off your machine.
Fedora Workstation 使用 ZRAM。这是一种在 RAM 的一部分中进行压缩的交换技术，以避免较慢的磁盘交换文件。不幸的是，这意味着在关闭机器进行挂起时，您没有持久空间来移动您的 RAM。

How it works  如何工作
The technique configures systemd and dracut to store and restore the contents of your RAM in a temporary swap file on disk. The swap file is created just before and removed right after hibernation to avoid trouble with ZRAM. A persistent swap file is not recommended in conjunction with ZRAM, as it creates some confusing problems compromising your systems stability.
该技术配置 systemd 和 dracut，将您的 RAM 内容存储并在磁盘上的临时交换文件中恢复。交换文件在挂起前后创建和删除，以避免与 ZRAM 的麻烦。不建议与 ZRAM 一起使用持久交换文件，因为它会引发一些令人困惑的问题，从而影响系统的稳定性。

A word on compatibility and expectations
关于兼容性和预期
Hibernation following this guide might not work flawless on your particular machine(s). Due to possible shortcomings of certain drivers you might experience glitches like non-working wifi or display after resuming from hibernation. In that case feel free to reach out to the comment section of the gist on github, or try the tips from the troubleshooting section at the bottom of this article.
按照本指南进行的挂起可能无法在您的特定机器上完美运行。由于某些驱动程序的潜在缺陷，您可能会在挂起后恢复时遇到无线网络或显示器无法工作等问题。在这种情况下，请随时在 GitHub gist 的评论部分留言，或尝试本文底部的故障排除部分提供的建议。

The changes introduced in this article are linked to the systemd hibernation.service and hibernation.target units and hence won’t execute on their own nor interfere with your system if you don’t initiate a hibernation. That being said, if it does not work it still adds some small bloat which you might want to remove.
本文介绍的变化与 systemd 的 hibernation.service 和 hibernation.target 单元相关，因此如果未经你启动休眠，它们不会自行执行也不会干扰你的系统。话虽如此，如果它不起作用，仍然会带来一些小的冗余，你可能想要移除它。

Hibernation in Fedora Workstation
Fedora Workstation 中的休眠
The first step is to create a btrfs sub volume to contain the swap file.
第一步是创建一个 btrfs 子卷来包含交换文件。

$ btrfs subvolume create /swap
In order to calculate the size of your swap file use swapon to get the size of your zram device.
为了计算交换文件的大小，使用 swapon 获取你的 zram 设备的大小。

$ swapon
NAME       TYPE      SIZE USED PRIO
/dev/zram0 partition   8G   0B  100
In this example the machine has 16G of RAM and a 8G zram device. ZRAM stores roughly double the amount of system RAM compressed in a portion of your RAM. Let that sink in for a moment. This means that in total the memory of this machine can hold 8G * 2 + 8G of RAM which equals 24G uncompressed data. Create and configure the swapfile using the following commands.
在这个例子中，这台机器有 16G 的 RAM 和一个 8G 的 zram 设备。ZRAM 将大约双倍于系统 RAM 的数据压缩存储在你的一部分 RAM 中。请稍作思考。这意味着这台机器的总内存可以容纳 8G * 2 + 8G 的 RAM，即 24G 未压缩的数据。使用以下命令创建和配置交换文件。

$ touch /swap/swapfile
# Disable Copy On Write on the file
$ chattr +C /swap/swapfile
$ fallocate --length 24G /swap/swapfile
$ chmod 600 /swap/swapfile 
$ mkswap /swap/swapfile
Modify the dracut configuration and rebuild your initramfs to include the
修改 dracut 配置并重新构建你的 initramfs 以包含

resume  恢复
module, so it can later restore the state at boot.
作为模块，以便它可以在启动时恢复状态。
$ cat <<-EOF | sudo tee /etc/dracut.conf.d/resume.conf
add_dracutmodules+=" resume "
EOF
$ dracut -f
In order to configure grub to tell the kernel to resume from hibernation using the swapfile, you need the UUID and the physical offset.
为了配置 grub 以使用交换文件让内核从挂起状态恢复，你需要 UUID 和物理偏移量。

Use the following command to determine the UUID of the swap file and take note of it.
使用以下命令确定交换文件的 UUID，并记下它。

$ findmnt -no UUID -T /swap/swapfile
dbb0f71f-8fe9-491e-bce7-4e0e3125ecb8
Calculate the correct offset. In order to do this you’ll unfortunately need gcc and the source of the btrfs_map_physical tool, which computes the physical offset of the swapfile on disk. Invoke gcc in the directory you placed the source in and run the tool.
计算正确的偏移量。为此，你将不幸需要 gcc 和 btrfs_map_physical 工具的源代码，该工具用于计算磁盘上交换文件的物理偏移量。在放置源代码的目录中调用 gcc，并运行该工具。

$ gcc -O2 -o btrfs_map_physical btrfs_map_physical.c
$ ./btrfs_map_physical /path/to/swapfile

FILE OFFSET  EXTENT TYPE  LOGICAL SIZE  LOGICAL OFFSET  PHYSICAL SIZE  DEVID  PHYSICAL OFFSET
0            regular      4096          2927632384      268435456      1      <4009762816>
4096         prealloc     268431360     2927636480      268431360      1      4009766912
268435456    prealloc     268435456     3251634176      268435456      1      4333764608
536870912    prealloc     268435456     3520069632      268435456      1      4602200064
805306368    prealloc     268435456     3788505088      268435456      1      4870635520
1073741824   prealloc     268435456     4056940544      268435456      1      5139070976
1342177280   prealloc     268435456     4325376000      268435456      1      5407506432
1610612736   prealloc     268435456     4593811456      268435456      1      5675941888
The first value in the PHYSICAL OFFSET column is the relevant one. In the above example it is 4009762816.
PHYSICAL OFFSET 列中的第一个值是相关的值。在上述示例中，它是 4009762816。

Take note of the pagesize you get from getconf PAGESIZE.
注意从 getconf PAGESIZE 获取的页大小。

Calculate the kernel resume_offset through division of physical offset by the pagesize. In this example that is 4009762816 / 4096 = 978946.
通过物理偏移除以页大小来计算内核 resume_offset。在这个例子中是 4009762816 / 4096 = 978946。

Update your grub configuration file and add the resume and resume_offset kernel cmdline parameters.
更新您的 grub 配置文件，并添加 resume 和 resume_offset 内核命令行参数。

grubby --args="resume=UUID=dbb0f71f-8fe9-491e-bce7-4e0e3125ecb8 resume_offset=2459934" --update-kernel=ALL
The created swapfile is only used in the hibernation stage of system shutdown and boot hence not configured in fstab. Systemd units control this behavior, so create the two units hibernate-preparation.service and hibernate-resume.service.
创建的交换文件仅在系统关机和启动的休眠阶段使用，因此不在 fstab 中配置。Systemd 单元控制这种行为，因此创建两个单元 hibernate-preparation.service 和 hibernate-resume.service。

$ cat <<-EOF | sudo tee /etc/systemd/system/hibernate-preparation.service
[Unit]
Description=Enable swap file and disable zram before hibernate
Before=systemd-hibernate.service

[Service]
User=root
Type=oneshot
ExecStart=/bin/bash -c "/usr/sbin/swapon /swap/swapfile && /usr/sbin/swapoff /dev/zram0"

[Install]
WantedBy=systemd-hibernate.service
EOF
$ systemctl enable hibernate-preparation.service
$ cat <<-EOF | sudo tee /etc/systemd/system/hibernate-resume.service
[Unit]
Description=Disable swap after resuming from hibernation
After=hibernate.target

[Service]
User=root
Type=oneshot
ExecStart=/usr/sbin/swapoff /swap/swapfile

[Install]
WantedBy=hibernate.target
EOF
$ systemctl enable hibernate-resume.service
Systemd does memory checks on login and hibernation. In order to avoid issues when moving the memory back and forth between swapfile and zram disable some of them.
Systemd 会在登录和休眠时进行内存检查。为了在内存在不同交换文件和 zram 之间切换时避免问题，可以禁用其中一些检查。

$ mkdir -p /etc/systemd/system/systemd-logind.service.d/
$ cat <<-EOF | sudo tee /etc/systemd/system/systemd-logind.service.d/override.conf
[Service]
Environment=SYSTEMD_BYPASS_HIBERNATION_MEMORY_CHECK=1
EOF
$ mkdir -p /etc/systemd/system/systemd-hibernate.service.d/
$ cat <<-EOF | sudo tee /etc/systemd/system/systemd-hibernate.service.d/override.conf
[Service]
Environment=SYSTEMD_BYPASS_HIBERNATION_MEMORY_CHECK=1
EOF
Reboot your machine for the changes to take effect. The following SELinux configuration won’t work if you don’t reboot first.
重启你的机器以使更改生效。如果你不先重启，以下 SELinux 配置将无法工作。

SELinux won’t like hibernation attempts just yet. Change that with a new policy. An easy although “brute” approach is to initiate hibernation and use the audit log of this failed attempt via audit2allow. The following command will fail, returning you to a login prompt.
SELinux 目前还不喜欢休眠尝试。通过新的策略来改变这一点。一个简单但“粗暴”的方法是启动休眠，并通过 audit2allow 使用这次失败的尝试的审计日志。以下命令将失败，并返回登录提示。

systemctl hibernate
After you’ve logged in again check the audit log, compile a policy and install it. The -b option filters for audit log entries from last boot. The -M option compiles all filtered rules into a module, which is then installed using semodule -i.
重新登录后，检查审计日志，编译策略并安装它。-b 选项用于过滤最后一次启动的审计日志条目。-M 选项将所有过滤后的规则编译成一个模块，然后使用 semodule -i 进行安装。

$ audit2allow -b
#============= systemd_sleep_t ==============
allow systemd_sleep_t unlabeled_t:dir search;
$ cd /tmp
$ audit2allow -b -M systemd_sleep
$ semodule -i systemd_sleep.pp
Check that hibernation is working via systemctl hibernate again. After resume check that ZRAM is indeed the only active swap device.
再次通过 systemctl hibernate 检查休眠是否正常工作。恢复后，确认 ZRAM 确实是唯一活动的交换设备。

$ swapon
NAME       TYPE      SIZE USED PRIO
/dev/zram0 partition   8G   0B  100
You now have hibernation configured.
现在休眠已配置完成。

GNOME Shell hibernation integration
GNOME Shell 休眠集成
You might want to add a hibernation button to the GNOME Shell “Power Off / Logout” section. Check out the extension Hibernate Status Button to do so.
你可能想在 GNOME Shell 的“关机/注销”部分添加一个休眠按钮。可以查看 Hibernate Status Button 扩展来实现这一点。

Troubleshooting  故障排除
A first place to troubleshoot any problems is through journalctl -b. Have a look around the end of the log, after trying to hibernate, to pin-point log entries that tell you what might be wrong.
解决任何问题的首要方法是使用 journalctl -b。尝试休眠后，查看日志的末尾，以确定可能出错的日志条目。

Another source of information on errors is the Problem Reporting tool. Especially problems, that are not common but more specific to your hardware configuration. Have a look at it before and after attempting hibernation and see if something comes up. Follow up on any issues via BugZilla and see if others experience similar problems.
另一个关于错误的来源是问题报告工具。特别是那些不常见但更具体于你的硬件配置的问题。在尝试挂起前后查看它，看看是否出现了一些问题。通过 BugZilla 跟进任何问题，看看其他人是否也遇到类似的问题。