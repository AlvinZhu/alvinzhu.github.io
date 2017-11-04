---
layout: post
mathjax: true
category: Linux
title: Ubuntu系统安装、维护与迁移技巧
tags: Ubuntu Install Linux
author: Alvin Zhu
date: 2017-11-04
---

* content
{:toc}
系统出问题了怎么办？重装...

换新电脑了怎么办？重装...

换硬盘了怎么办？还是重装...

其实没必要的。这篇文章就介绍如何避免以上尴尬。





## Ubuntu安装技巧

Ubuntu具体的按照方法可以参考相关文档，我只给出我的建议：

分区表示例如下：

```
Device        Start       End   Sectors   Size Type
/dev/sda1      2048    264191    262144   128M EFI System
/dev/sda2    264192  33818623  33554432    16G Linux swap
/dev/sda3  33818624 500117503 466298880 222.4G Linux filesystem
```

1. 第一个是EFI分区，128MB大小，放boot loader，默认FAT32格式，挂载到`/boot/efi`。

   如果是BIOS+MBR的启动方式，第一个分区512MB大小，格式化成ext格式，挂到/boot下。

2. 第二个分区是swap分区，大小16GB。

3. 第三个是root分区，硬盘所有剩余空间都给它，格式化为btrfs。


解释下我为什么这么分。第1条应该没什么争议。第2条的关键是swap的位置，放在了root前面，这样做的好处是方便迁移和折腾，比如虚拟机里面要扩充硬盘大小，比如我想临时装个双系统，比如我要把系统迁移到其他硬盘或机器上，这都比把swap放在最后要方便。别管网上说的要根据内存大小划分swap分区大小的那一套说法，那是很多年前的事情了，Ubuntu的休眠功能默认屏蔽了，即使可以开启，在有些电脑上也有BUG，驱动问题，而且休眠也很少用，所以我不需要2倍内存大小的swap，内存够大，swap一般都是空着的，留个swap也只是为了某些软件的兼容性，比如VMware。第3条，为啥只有一个root，home等其他分区呢？因为我用btrfs，自带逻辑卷功能，ubuntu使用btrfs的时候默认会创建两个子卷，一个root，一个home，够用了。

## Ubuntu维护技巧

既然使用了btrfs，最有用的功能之一当然是它的snapshot功能。写个脚本，定期快照，系统出问题了恢复。因为是“Copy on Write”，快照创建和恢复都是瞬间完成，不用担心速度问题。懒人不想写脚本可以使用snapper，实现了类似MacOS的Time Machine功能，但是备份恢复速度比这个老古董快到哪去了，完全感觉不到它的存在。

有了这个机制，装软件、改系统配置之前都snapshot一下，不行就恢复，系统还会被我搞坏吗？

## Ubuntu迁移技巧

系统经过一段时间的使用，已经配置的得心应手了，然后有了心电脑，或者换硬盘，难道我再重装一遍吗？当然不干。

使用[Clonezilla](http://www.clonezilla.org/)，备份恢复，搞定。这个可以满足大多数情况下的需求。毕竟Clonezilla也有Bug。

当然也可以手动迁移。关键词：tar、btrfs send、btrfs receive、chroot、grub-install、fdisk、gparted。这几个工具玩转了，从此告别重装系统。

当然，我只能保证2年内不重装，Ubuntu的LTS大版本更新的时候，还是要重装下，直接升级不靠谱。