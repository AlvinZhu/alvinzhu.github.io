---
layout: post
mathjax: true
category: Linux
title: Ubuntu16.04 KVM PCI-E Passthrough 配置
tags: Ubuntu KVM Linux PCI-E_Passthrough
author: Alvin Zhu
date: 2017-11-06
---

* content
{:toc}
一直以来我都有一个想法，那就是在虚拟机中使用显卡...

VMware暂时就不要指望了，Workstation目前是不支持的。ESXi可以，Windows Guest用显卡也没啥毛病，但是Ubuntu Guest就悲剧了，而且要把ESXi作为主系统也不符合我的需求。KVM更灵活，可以办到，不过还有些小毛病。



## 安装KVM

```sh
sudo apt install virt-manager qemu-kvm ovmf
```
就这一句就够了，然后是复杂的配置...

## 配置KVM

1. apparmor权限配置：

   ```sh
   sudo ln -s /etc/apparmor.d/usr.sbin.libvirtd  /etc/apparmor.d/disable/
   sudo ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper  /etc/apparmor.d/disable/
   sudo apparmor_parser -R  /etc/apparmor.d/usr.sbin.libvirtd
   sudo apparmor_parser -R  /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
   ```

2. 内核模块加载：

   `/etc/initramfs-tools/hooks/vfio`：

   ```sh
   #!/bin/sh

   PREREQ=""

   prereqs()
   {
           echo "$PREREQ"
   }

   case $1 in
   # get pre-requisites
   prereqs)
           prereqs
           exit 0
           ;;
   esac

   . /usr/share/initramfs-tools/hook-functions

   cp -pL --remove-destination "/sbin/vfio-pci-override.sh" "${DESTDIR}/sbin/vfio-pci-override.sh"
   ```

   `/etc/initramfs-tools/modules`：

   ```sh
   # List of modules that you want to include in your initramfs.
   # They will be loaded at boot time in the order below.
   #
   # Syntax:  module_name [args ...]
   #
   # You must run update-initramfs(8) to effect this change.
   #
   # Examples:
   #
   # raid1
   # sd_mod
   vfio
   vfio_iommu_type1
   vfio_pci
   vfio_virqfd
   vhost-net
   ```

   `/etc/modprobe.d/kvm.conf`：

   ```sh
   options kvm ignore_msrs=1
   ```

   `/etc/modprobe.d/nvidia.conf`：

   ```sh
   blacklist nouveau
   softdep nvidia pre: vfio-pci
   ```

   `/etc/modprobe.d/vfio.conf`：

   ```sh
   install vfio-pci /sbin/vfio-pci-override.sh
   ```

   `/sbin/vfio-pci-override.sh`：

   ```sh
   #!/bin/sh

   DEVS=""

   #GPU
   #DEVS="$DEVS 0000:05:00.0 0000:05:00.1"
   DEVS="$DEVS 0000:06:00.0 0000:06:00.1"
   #DEVS="$DEVS 0000:09:00.0 0000:09:00.1"

   #SATA
   DEVS="$DEVS 0000:00:11.0 0000:00:11.4"

   #USB
   #DEVS="$DEVS 0000:00:14.0"

   #USB
   DEVS="$DEVS 0000:11:00.0"

   #Audio
   DEVS="$DEVS 0000:00:1b.0"

   #CMD=`cat /proc/cmdline`
   #IS_IOMMU_ON=0

   #for P in $CMD; do
   #    if [ $P = "intel_iommu=on" ]; then
   #        IS_IOMMU_ON=1
   #    fi
   #done

   IS_IOMMU_ON=`find /sys/kernel/iommu_groups/ -maxdepth 1 -mindepth 1 -type d | wc -l`

   if [ $IS_IOMMU_ON -ne 0 ]; then
   for DEV in $DEVS; do
       if [ -e /sys/bus/pci/devices/$DEV/driver ]; then
       echo $DEV > /sys/bus/pci/devices/$DEV/driver/unbind
       fi
       echo "vfio-pci" > /sys/bus/pci/devices/$DEV/driver_override
   done
   modprobe -i vfio-pci
   fi
   ```

3. 更新grub和initramfs：

   编辑`/etc/default/grub`文件，修改`GRUB_CMDLINE_LINUX_DEFAULT`，加上`intel_iommu=on`，并在BIOS中开启`VT-d`。

   ```sh
   sudo update-grub
   sudo update-initramfs -u
   ```

4. 修复账户现实问题：

   ```sh
   sudo echo -e "[User]\nSystemAccount=true" > /var/lib/AccountsService/users/libvirt-qemu
   ```

5. 如果使用了Btrfs，最好加上：

   ```sh
   sudo chattr +C -R /var/lib/libvirt/images
   ```

   以提高性能和稳定性。


## 配置Windows Guest

完整配置文件如下：

```xml
<!--
WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
OVERWRITTEN AND LOST. Changes to this xml configuration should be made using:
  virsh edit Windows
or other application using the libvirt API.
-->

<domain type='kvm' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>
  <name>Windows</name>
  <uuid>1d196c70-df3c-42e5-8494-9519c53b0671</uuid>
  <memory unit='KiB'>16777216</memory>
  <currentMemory unit='KiB'>16777216</currentMemory>
  <memoryBacking>
    <nosharepages/>
  </memoryBacking>
  <vcpu placement='static'>8</vcpu>
  <cputune>
    <vcpupin vcpu='0' cpuset='4'/>
    <vcpupin vcpu='1' cpuset='5'/>
    <vcpupin vcpu='2' cpuset='6'/>
    <vcpupin vcpu='3' cpuset='7'/>
    <vcpupin vcpu='4' cpuset='12'/>
    <vcpupin vcpu='5' cpuset='13'/>
    <vcpupin vcpu='6' cpuset='14'/>
    <vcpupin vcpu='7' cpuset='15'/>
  </cputune>
  <sysinfo type='smbios'>
    <bios>
      <entry name='vendor'>American Megatrends Inc.</entry>
      <entry name='version'>3402</entry>
      <entry name='date'>11/14/2016</entry>
      <entry name='release'>5.11</entry>
    </bios>
    <system>
      <entry name='manufacturer'>ASUS</entry>
      <entry name='product'>All Series</entry>
      <entry name='version'>System Version</entry>
      <entry name='serial'>System Serial Number</entry>
      <entry name='sku'>All</entry>
      <entry name='family'>ASUS MB</entry>
    </system>
    <baseBoard>
      <entry name='manufacturer'>ASUSTeK COMPUTER INC.</entry>
      <entry name='product'>X99-E WS/USB 3.1</entry>
      <entry name='version'>Rev 1.xx</entry>
      <entry name='serial'>160470801200061</entry>
    </baseBoard>
  </sysinfo>
  <os>
    <type arch='x86_64' machine='pc-q35-2.5'>hvm</type>
    <loader readonly='yes' type='pflash'>/usr/share/OVMF/OVMF_CODE.fd</loader>
    <nvram>/var/lib/libvirt/qemu/nvram/Windows_VARS.fd</nvram>
    <bootmenu enable='yes'/>
    <smbios mode='sysinfo'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <hyperv>
      <relaxed state='on'/>
      <spinlocks state='on' retries='8191'/>
    </hyperv>
    <kvm>
      <hidden state='on'/>
    </kvm>
  </features>
  <cpu mode='host-passthrough'>
    <topology sockets='1' cores='4' threads='2'/>
  </cpu>
  <clock offset='localtime'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
    <timer name='hypervclock' present='yes'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <on_lockfailure>poweroff</on_lockfailure>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <target dev='sda' bus='sata'/>
      <readonly/>
      <serial>9a6a0e40-85e0-4091-9ba0-a1fc8d9637cb</serial>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
    </disk>
    <controller type='sata' index='0'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x1f' function='0x2'/>
    </controller>
    <controller type='pci' index='0' model='pcie-root'/>
    <controller type='pci' index='1' model='dmi-to-pci-bridge'>
      <model name='i82801b11-bridge'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x1e' function='0x0'/>
    </controller>
    <controller type='pci' index='2' model='pci-bridge'>
      <model name='pci-bridge'/>
      <target chassisNr='2'/>
      <address type='pci' domain='0x0000' bus='0x01' slot='0x01' function='0x0'/>
    </controller>
    <controller type='usb' index='0'>
      <address type='pci' domain='0x0000' bus='0x02' slot='0x02' function='0x0'/>
    </controller>
    <interface type='bridge'>
      <mac address='00:50:56:aa:bb:29'/>
      <source bridge='bridge0'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x02' slot='0x01' function='0x0'/>
    </interface>
    <hostdev mode='subsystem' type='pci' managed='yes'>
      <source>
        <address domain='0x0000' bus='0x11' slot='0x00' function='0x0'/>
      </source>
      <address type='pci' domain='0x0000' bus='0x02' slot='0x04' function='0x0'/>
    </hostdev>
    <hostdev mode='subsystem' type='pci' managed='yes'>
      <source>
        <address domain='0x0000' bus='0x00' slot='0x1b' function='0x0'/>
      </source>
      <address type='pci' domain='0x0000' bus='0x02' slot='0x05' function='0x0'/>
    </hostdev>
    <hostdev mode='subsystem' type='pci' managed='yes'>
      <source>
        <address domain='0x0000' bus='0x06' slot='0x00' function='0x0'/>
      </source>
      <address type='pci' domain='0x0000' bus='0x02' slot='0x06' function='0x0' multifunction='on'/>
    </hostdev>
    <hostdev mode='subsystem' type='pci' managed='yes'>
      <source>
        <address domain='0x0000' bus='0x06' slot='0x00' function='0x1'/>
      </source>
      <address type='pci' domain='0x0000' bus='0x02' slot='0x07' function='0x0'/>
    </hostdev>
    <hostdev mode='subsystem' type='pci' managed='yes'>
      <source>
        <address domain='0x0000' bus='0x00' slot='0x11' function='0x0'/>
      </source>
      <boot order='1'/>
      <address type='pci' domain='0x0000' bus='0x02' slot='0x08' function='0x0'/>
    </hostdev>
    <hostdev mode='subsystem' type='pci' managed='yes'>
      <source>
        <address domain='0x0000' bus='0x00' slot='0x11' function='0x4'/>
      </source>
      <boot order='2'/>
      <address type='pci' domain='0x0000' bus='0x02' slot='0x09' function='0x0'/>
    </hostdev>
    <memballoon model='none'/>
  </devices>
  <qemu:commandline>
    <qemu:arg value='-cpu'/>
    <qemu:arg value='host,kvm=off,hv_vendor_id=alvin,hv_time,hv_relaxed,hv_spinlocks=0x1fff'/>
  </qemu:commandline>
</domain>
```

我使用的主板是ASUS X99-E WS/USB 3.1，把板载的声卡、1个USB3.1控制器、1个SATA控制器、1个网卡、1个显卡都给Passthrough了（主板上东西多，即使这样，Host系统还剩下1个USB控制器、1个SATA控制器、1个网卡、3个显卡，主系统Ubuntu，声卡也没啥用）。这样的好处是Guest性能爆棚，和Host基本没区别，无论是USB、硬盘、网络。

Host与Guest之间共享文件主要通过samba服务。

Guest独占一个或多个显示器（因为是GPU Passthrough）。

Guest的鼠标键盘直接使用物理设备，然后使用[Synergy](https://symless.com/synergy)来实现一套键鼠控制多个系统。

## 存在的问题

软硬件对PCI-E Passthrough的支持还不是特别完善，Guest重启几次后很容易导致PCI-E Passthrough失败。另外开启VT-d后，NVIDIA的多卡P2P就废了，影响多显卡之间的通讯性能。

用了一段时间，小毛病太多，还没有达到能长期稳定使用的地步。纯折腾。