title: Deskmini 310 黑苹果折腾记
author: 大丈夫没问题
tags:
  - hackintosh
  - mac
categories: []
date: 2019-06-13 20:47:00
---
家里的台机装了`manjaro`，用了一阵子感觉还是不顺手，加上苹果电脑是越来越贵了，所以有了弄一台黑苹果的想法。

期间看了不少有关黑苹果的资料，发现最简单的方法就是按照别人成功的硬件配置和EFI来安装，最终方案是按照 [asrock_deskmini310_hackintosh](https://github.com/liminghuang/asrock_deskmini310_hackintosh) 买的硬件：

|           |                                                 型号                                                | 价格（RMB） |
|:---------:|:---------------------------------------------------------------------------------------------------:|:----:|
|   主板/机箱 |                                                H3101                                                |  969 |
|    CPU    |                                               I7 8700                                               | 2299 |
|    内存   |                                  金士顿 DDR4 2666 16G 笔记本内存 x2                                 | 1228 |
|    SSD    |                                           Intel 760p 512GB                                          |  579 |
| 无线/蓝牙 |   [DW1560/BCM94352Z WIFI/Bluetooth module mini PCIE/NGFF M2](https://m.tb.cn/h.e4MwVdx?sm=19957d)   |  268 |
|    天线   | [NGFF M2无线网卡转接线天线 IPX4代转SMA线DeskMini华硕H110 X370](https://m.tb.cn/h.efged5F?sm=98c2ff) |  18  |
|  DP转VGA  |                     [DP转VGA转换器](https://item.m.jd.com/product/2388562.html)                     |  75  |

安装过程中走了不少弯路，这里记载一下：

1. 由于我买来的主板BIOS版本是3.4版本的，直接安装会报一个nvme错误，想着升级一下BIOS能不能解决这问题，升级到4.1版本后发现还是有这错误，解决方案是要用`clover configurator`编辑下EFI文件，添加[DSDT Patch](https://github.com/liminghuang/asrock_deskmini310_hackintosh#for-bios-4x)，
2. 这个方案里视频是通过HDMI和DP接口输出的，家里的显示器是VGA接口，装到一半黑屏还以为是EFI文件有问题，最后还是接了电视机的HDMI完成的安装。显示器后来用的这款 [绿联（UGREEN）DP转VGA转换器](https://item.m.jd.com/product/2388562.html?wxa_abtest=o&utm_source=iosapp&utm_medium=appshare&utm_campaign=t_335139774&utm_term=CopyURL&ad_od=share&ShareTm=gBDVk/8FzqoyyiWF%2Bv1OIUIRUi7tkAujQ2W5kHCYvZ8/1U6vzEh6m/p%2BK89EebnQhqmJ1oxCkQnfud9J9BMq/DxjfW59qvVVq1tdhSxzZMmCRBYn0NhOZIMhri0oFY/lSTnjQhGZle9gajwgMjfj0W/51ImsB2idtcaXpvBrLF0=)
3. 按照tonymacx86上的教程来安装，第二次重启安装会报错，发现还是要先将EFI文件复制到安装镜像里才能安装成功，安装过程中可能会有panic错误，重启下再次安装就好了。
4. 网卡和蓝牙买的是[DW1560 BCM94352Z](https://item.taobao.com/item.htm?ut_sk=1.WqN7QogLod0DAI3%2BU31NQxC1_21380790_1560070740119.Copy.1&id=524391843184&sourceType=item&price=53-351&suid=80BF8BDA-6252-471B-ADFD-C75185B4E4FD&un=bf3363486317a89a4b68d5f77a161062&share_crt_v=1&sp_tk=77+lWVBFQ1lWajhFTXDvv6U=&cpp=1&shareurl=true&spm=a313p.22.27q.1040917476894&short_name=h.e4MwVdx&sm=19957d&app=chrome) ，这里有一点要注意的是要买天线，不装天线的话会搜索不到蓝牙设备和WIFI
5. 天线安装有点麻烦，我是把网卡拆下来才把天线接上去的

安装视频可以参考这个：
<iframe width="560" height="315" src="https://www.youtube.com/embed/KnoS3zWpElo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

整个安装过程花了差不多七八个小时，不走弯路的话估计一两个小时就能搞定。

用了几天发现两个问题：
1. 休眠和启动时音箱会有爆音
2. 从休眠恢复会黑一下屏幕


-------

2021-04-08 原来那位大佬改用外置显卡了，他的仓库还在用clover，并且好久没更新了，现在更换了免驱动的无线网卡[BCM94360CS2](https://m.tb.cn/h.4p0rwjY)，且改用了opencore引导，用的是这位大佬的配置文件[appleserial/DeskMini](https://github.com/appleserial/DeskMini)


## 更新日志
* 2019-06-16: 更新了发现的问题
* 2019-08-07: 升级到了10.14.6一切正常 
* 2021-04-08: 更换成免驱动的无线网卡