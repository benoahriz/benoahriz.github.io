---
title: SMP affinity makes a difference
date: 2016-07-13 14:25:48
tags:
author:      "Benjamin Rizkowsky"
---

# A good driver.

Having a driver that uses the proper interrupt handling makes a difference in high bandwidth pci devices like RAID cards and Infiniband.

For example having the stock 2.6.32-220 megasas driver only makes use of one interrupt.

This was the stock driver in the centos 2.6.32-220 kernel

``` bash
[root@demo mnt]# cat /proc/interrupts |grep megasas
  79:       2506          0          0          0          0          0          0          0  IR-PCI-MSI-edge      megasas

```

This is the driver from LSI compiled against the same kernel sources. Notice the additional interrupt usage.

``` bash
[root@demo mnt]# cat /proc/interrupts |grep megasas
  80:       2506          0          0          0          0          0          0          0  IR-PCI-MSI-edge      megasas
  81:        124          0          0          0          0          0          0          0  IR-PCI-MSI-edge      megasas
  82:         24          0          0          0          0          0          0          0  IR-PCI-MSI-edge      megasas
  83:          7          0          0          0          0          0          0          0  IR-PCI-MSI-edge      megasas
  84:        993          0          0          0          0          0          0          0  IR-PCI-MSI-edge      megasas
  85:         80          0          0          0          0          0          0          0  IR-PCI-MSI-edge      megasas
  86:         17          0          0          0          0          0          0          0  IR-PCI-MSI-edge      megasas
  87:          8          0          0          0          0          0          0          0  IR-PCI-MSI-edge      megasas

```
