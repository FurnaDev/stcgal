文档说明 Explanation
------------------------
此文档翻译自FAQ.md

This document was translated from FAQ.md

最后修改时间：2020年6月8日

Last modified time: June 8, 2020

常见问题
======================================

### 问题1：是否可以从芯片中读取代码（或EEPROM）存储器？

从设计上讲，这是STC的引导加载程序协议无法实现的。 STC将此视为安全功能。目前没有已知的解决方法。有关更多详细信息和讨论，请参见问题7。

### 问题2：哪些串行接口已通过stcgal测试过？

stcgal应该可以与波特率为16550的UART兼容。
但是，如今，基于USB模拟的UART是典型的情况。
以下是已通过stcgal成功测试的USB模拟UART接口芯片：

* FT232系列（操作系统：Linux，Windows）
* CH340 / CH341（操作系统：Windows，Linux需要内核4.10）
* PL2303（操作系统：Windows，Linux）
* CP2102（操作系统：Windows，Linux，macOS）

已知不起作用的接口：

* Raspberry Pi Mini UART（缺少奇偶校验支持，请启用PL011 UART）

### 问题3：stcgal 启动失败同时显示 `module 'serial' has no attribute 'PARITY_NONE'` 等类似信息

PyPI软件包“ serial”（数据序列库）和PyPI软件包“ pyserial”（stcgal所需的串行端口访问库）之间存在模块名称冲突。
您必须卸载'serial'软件包（`pip3 uninstall serial`）并重新安装'pyserial'（`pip3 install --force-reinstall pyserial`）才能解决此问题。
目前没有其他已知的解决方案。

### 问题4：stcgal无法识别MCU，并停留在“Waiting for MCU”中

有许多问题可能导致此症状：
* 电气问题和错误连接。确保正确连接了RX / TX，GND和VCC。
如果您不使用自动复位功能，还应确保仅在stcgal启动后才接通电源，因为引导加载程序仅在上电复位时被调用。
* 通过I / O引脚供电。
即使未连接VCC，也可以通过I / O引脚（例如RX / TX）为MCU供电。
在这种情况下，上电复位逻辑不起作用。请参阅下一个问题。
* erial接口不兼容。由于各种原因，一些基于USB的UART与STC MCU的兼容性很差。
您可以尝试使用选项`-l 1200`将握手波特率从标准2400波特降低到1200波特，在某些情况下可以解决这些问题。

### 问题5：如何避免MCU从I/O引脚供电？
可以采取各种补救措施来避免MCU从I/O引脚供电。
* 您可以尝试在MCU VCC和GND之间连接一个电阻（<1k），以使注入的电源短路，并希望将电压降至欠压值以下。
* 另一种选择是在可能注入功率的I / O线上插入串联电阻。例如，在RX / TX线上尝试一个类似1k的值。
* 还有另一种可能性是切换GND而不是VCC。
在大多数情况下，这应该是一个相当可靠的解决方案。

### 问题6：RC频率调整失败
首先，请确保指定的频率使用正确的单位。频率以kHz为单位指定，安全范围约为5000 kHz-30000 kHz。
此外，频率调整使用UART时钟作为时钟参考，因此UART不兼容或时钟不准确也会导致频率调整问题。如果可能的话，
尝试另一个UART芯片。

### 问题7：波特率切换失败或闪存编程失败
特别是在高编程波特率，例如， 115200波特。尝试降低波特率，或使用默认的19200波特率。
某些USB UART也会由于时序不正确而引起问题，这可能会导致各种问题。

### 问题8：如何使用自动重置功能？
标准自动重置功能的工作原理与Arduino类似。 DTR是低电平有效信号，在stcgal启动时置位500 ms，然后在其余的编程序列中置为无效。
在标准USB UART上，这将导致500 ms的低脉冲，然后是高相位。 stcgal作者推荐以下电路：
```
VCC --o      o-- MCU GND
      |      |
     .-.     |
     | | 1k  |
     | |     |
     '_'     |
      |      |
      |   ||-+
DTR --o --||<- BS170/BSS138
          ||-| (N-CH MOSFET)
             |
             |
GND ---------o
```
该电路使用一个N沟道MOSFET作为开关来切换MCU的GND。 VCC直接连接。这避免了寄生供电问题。上拉电阻可确保在DTR输入悬空时接通MCU。