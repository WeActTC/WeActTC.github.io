---
title: 'STM32 下载烧录问题汇总'
date: 2019-11-30
tags:
 - STM32
 - 2019
categories:
 - STM32
top_img:
cover:  /images/STM32/STM32F4x1仿真.png
---
![STM32F4X1](/images/STM32/STM32F4x1仿真.png "STM32F4X1CXU6")
# ISP下载
按住BOOT0键和NRST键，然后松开NRST键，0.5秒后松开BOOT0键，即可进入串口下载或DFU下载，USB连接MCU的TYPE-C接口或者串口连接PA9、PA10，下载软件推荐STM32CubeProg。
## 串口下载
 USB转串口 (ex.：CH340) `TX` - `PA10` ,`RX` - `PA9`，同时不要将MCU的`Type-C`连接到电脑，必须使用外部供电，不然会影响MCU下载。

 ![STM32CubeProgrammer 串口下载1](/images/STM32/stm32cubeprog串口1.png "STM32CubeProgrammer 串口下载1")
  ![STM32CubeProgrammer 串口下载2](/images/STM32/stm32cubeprog串口2.png "STM32CubeProgrammer 串口下载2")
## USB下载
1. STM32CubeProgrammer勾选USB模式
  ![STM32CubeProgrammer USB下载](/images/STM32/stm32cubeprogUSB1.png "STM32CubeProgrammer USB下载")
2. MCU进入ISP模式，使用USB连接电脑
3. 选择固件，其余操作跟串口下载一致
4. STM32F401的USB下载受天气影响可能存在一定的不稳定性，如反复出现如下ERROR，请采用串口下载，并断开USB连接。
```
Error: failed to download Segment[0]
Error: failed to download the File
```

> 上述`ERROR`造成原因：室温偏低，HSI产生偏差，`USB下载`使用的是`外部高速晶振`，而`ISP程序`（`ST的自举程序`）通过`HSI`测量外部晶振HSE频率然后再配置时钟，当`HSI`偏差过大，`HSE`测量频率不正确，从而使得USB时序不对，造成下载错误。具体详情可见网盘 `/通用文档/AN2606 STM32微控制器系统存储器自举模式.pdf`

> 解决方法：适当加热MCU至`25°C`以上（`用手捂热`）

# ST-Link/J-link下载
连接STM32的SW接口：

| SW接口 |
| :--: |
|GND|
|SCK|
|DIO|
|3.3V|

在MDK软件点击下载按钮或者在STM32CubeProg中选择ST-Link根据提示操作即可。
1. CubeMX工程或标准库工程，要使能SW调试接口，不然调试器是不能识别出MCU
2. 代码工程晶振设置不对或其他异常导致调试器不能识别MCU，此时手动设置MCU进入ISP模式，调试器就能识别出MCU，再点击下载即可
3.  ISP模式只是ST公司固化在MCU里面的一段启动代码，检查BOOTx设置，运行模式则转跳地址`0x08000000`运行，下载模式则等待下载命令，此时SW调试下载接口是开放的，调试器可以读取下载MCU代码。
4. 调试接口分SW接口和JTAG接口，ARM的调试器基本都支持SW接口
5. JLink 能连接上芯片，但是不能下载，请升级Jlink驱动到最新版本，V6.50a测试可用

| JTAG接口 |转接| SW接口 |
| :--: | :--:|:--: |
|TMS|  | SWDIO|
|TCK|  | SWCLK |
|VTEST 1脚|某些JLink需要接到3.3V才识别MCU|3.3V|
|3.3V| | 3.3V |
|GND|  | GND|

![JTAG接口 和 SW 接口](/images/STM32/JTAG.png "JTAG接口 和 SW 接口")

# HID FLASH 下载
> 部分板子测试时已经预烧录了HID Bootloader

STM32F401CC、STM32F411CE 核心板均可使用，实现类似 51 单片机下载，但无需串口，只需一根数据线，
和修改Keil工程两个地方（详情见视频）即可实现。速度比串口下载更快且更方便

![](/images/STM32/HIDFlash2.png "WeAct HID FW Bootloader 软件界面")

```
百度网盘资料
链接：https://pan.baidu.com/s/1vW9H-q9C5n2yVAEp38pw5A 
提取码：gxnx 
```
## APP 工程修改方法
1. 修改工程ROM起始地址为 `0x8004000`
![](/images/STM32/stm32cubeideset.jpg "STM32CubeIDE 设置")
2. main()函数开头增加以下代码
```
SCB->VTOR = FLASH_BASE | 0x4000; 
```
## 软件使用步骤
1. 将核心板用数据线连接电脑，出现WeAct HID设备
2. 按住KEY键，重新上电或复位进入Bootloader
3. 软件选择固件，点击<下载固件>即可完成下载
4. 所选固件会随KEIL重新编译而更新，无需重新选择
## 进入Bootloader方法
1. 按住<KEY键>，重新上电或复位，`C13 LED` 闪烁即可松开
2. APP进入Bootloader 参考stm32f401_test_APP 0x8004000.zip 工程
## 在Bootloader 中
1. 单击/双击<`KEY键`>为 `C13 LED` 亮灭
2. SW 调试口开放，可以用调试器烧写，无需进入DFU模式
## 退出Bootloader 方法
1. 复位MCU， 复位键/上位机点击<MCU 复位>
2. 长按<`KEY键`>，`C13 LED` 闪烁即可松开
## 注意事项：
1. 首次烧录 `Bootloader`，MCU不会往下运行，同时 `C13 LED` 200MS闪烁，只需再次复位MCU即可
2.  `Bootloader` 以及APP烧录软件源码均不开放
## 使用教程视频
<video width="400" height="200" controls>
<source src="/images/STM32/HIDFlash软件下载操作视频.mp4">
</video>

PS：以上方法是可以解决99%的下载烧录问题哦
>这里强烈推荐使用调试器烧录程序，方便快捷，亦可调试
>网上老旧资料较多，推荐参考[ST官网](https://www.stmcu.com.cn "www.stmcu.com.cn")比较科学
关于stm32的下载烧录问题，不定时更新

![](/images/weact-logo1.png "一个致力于设计独一无二电子模块的工作室")
<p align="right">一个致力于设计独一无二电子模块的工作室</p>