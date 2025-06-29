+++
date = '2025-06-29T15:03:53+08:00'
draft = false
title = 'Disk Format'
summary = '关于硬盘的文件系统'
+++

在备份 windows 电脑的时候遇到了一个问题，windows 的硬盘镜像没法写到 exFAT 的移动硬盘里边。试过了 minitool，但是 minitool 也没有办法给 exFAT 的硬盘划分出一个 NTFS 的分区。于是乎了解了一下 exFAT 和 NTFS 的区别。

NTFS
- 是 windows 的默认文件系统
- 支持以下功能：文件权限，日志记录，文件加密，磁盘配额
- 适用于高可靠性，安全性和大量小文件的场景
- Mac 没法直接读取

exFAT
- 为 U 盘设计的轻量级文件系统
- 兼容性强，Mac 和 Windows 都能读取
- 不能划分新分区

如果是 Mac 的话，它支持 APFS 文件系统。但是同理 APFS 不能被 Windows 直接读取