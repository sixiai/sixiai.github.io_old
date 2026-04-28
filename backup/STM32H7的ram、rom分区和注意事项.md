## 1、STM32H7系列ram的分布
ram（随机存取存储器，random access memory），以STM32H743VIT6为例，为1M RAM+2M Flash的存储空间。
其中1M RAM = 128Kbyte_DTCM +512Kbyte_AXI_SRAM + 128Kbyte_SRAM1 + 128Kbyte_SRAM2 + 32Kbyte_SRAM13+ 64Kbyte_SRAM4 
```c
/* 
   说明:
   - x: 可执行 (eXecutable)
   - r: 可读 (Readable)
   - w: 可写 (Writable)
   - K: Kilobyte (1024 bytes)
   - M: Megabyte (1024 * 1024 bytes)
*/

MEMORY
{
  /* --- 内核紧密耦合内存 (Tightly-Coupled Memory, TCM) --- */
  /* CPU内核专用，零等待周期访问，DMA不可访问 */
  ITCMRAM (xrw) : ORIGIN = 0x00000000, LENGTH = 64K     /* 指令紧密耦合内存 (Instruction TCM) */
  DTCMRAM (xrw) : ORIGIN = 0x20000000, LENGTH = 128K    /* 数据紧密耦合内存 (Data TCM) */

  /* --- 主SRAM域 (SRAM Domain) --- */
  /* 高性能，通过64位AXI总线连接，CPU和DMA均可访问 */
  AXISRAM (xrw) : ORIGIN = 0x24000000, LENGTH = 512K    /* AXI SRAM，主SRAM */

  /* --- 其他SRAM (位于SRAM Domain，但通常分开命名) --- */
  /* 通过总线矩阵连接，CPU和DMA均可访问，速度略低于AXI SRAM */
  SRAM1   (xrw) : ORIGIN = 0x30000000, LENGTH = 128K    /* SRAM1 (D2域) */
  SRAM2   (xrw) : ORIGIN = 0x30020000, LENGTH = 128K    /* SRAM2 (D2域) */
  SRAM3   (xrw) : ORIGIN = 0x30040000, LENGTH = 32K     /* SRAM3 (D2域) */
  
  /* --- 备份SRAM (Backup SRAM) --- */
  /* 位于备份域(D3域)，可在低功耗模式下保持数据 */
  BKPSRAM (rw)  : ORIGIN = 0x38800000, LENGTH = 4K      /* Backup SRAM */

  /* --- FLASH 内存 (Flash Memory) --- */
  /* 非易失性存储，用于存放代码和只读数据 */
  FLASH   (rx)  : ORIGIN = 0x08000000, LENGTH = 2048K   /* 主Flash，2MB */

  /* --- 片上系统内存 (System Memory) --- */
  /* 存储了ST官方的Bootloader，用于通过USART, USB等接口烧录程序 */
  SYSROM  (rx)  : ORIGIN = 0x1FF00000, LENGTH = 120K    /* System ROM / Bootloader */
}
```
## 2、各内存区域详细说明
1、ITCMRAM (Instruction TCM)
用途: 存放最关键、需要最快执行速度的程序代码，如中断服务函数、核心算法。
特性: CPU指令总线零等待访问，速度最快。DMA无法访问。

2、DTCMRAM (Data TCM)
用途: 存放最关键、需要最快访问速度的数据，如中断使用的全局变量、高速数据缓冲区、任务栈（Stack）。
特性: CPU数据总线零等待访问，速度最快。DMA无法访问。

3、AXISRAM (AXI SRAM)
用途: 性能最高的通用SRAM，可作为程序的主RAM。用于存放全局变量、静态变量、堆（Heap）和栈（Stack）。
特性: 通过高速AXI总线连接，CPU和DMA均可高速访问。是STM32H7性能的核心之一。

4、SRAM1, SRAM2, SRAM3
用途: 通用SRAM，可用于数据缓冲、外设DMA传输等。
特性: 位于D2域，通过总线矩阵连接，CPU和所有DMA主设备（Master）均可访问，非常灵活。

5、BKPSRAM (Backup SRAM)
用途: 用于在系统复位或进入待机/VBAT低功耗模式时，保存少量关键数据。
特性: 由备用电源（VBAT）供电，只要VBAT有电，数据就不会丢失。

6、FLASH
用途: 存放整个程序的代码和只读数据 (const)。程序启动时，需要初始化的变量值也保存在这里，由启动代码复制到SRAM。
特性: 非易失性（断电不丢失），但访问速度远慢于SRAM。CPU读取Flash时会有多个等待周期，因此需要ART加速器或Cache来提升性能。

7、SYSROM (System ROM)
用途: 固化在芯片内部的只读存储器，包含了ST的出厂Bootloader。通过设置BOOT0和BOOT1引脚，可以让芯片从这里启动，从而通过串口、USB等方式更新Flash中的程序。
特性: 用户不可更改。

这个内存布局是理解和优化STM32H7性能的关键。通过在链接器脚本中合理地分配代码和数据到不同的内存区域，可以最大化地发挥其高性能优势。

<img width="941" height="1076" alt="Image" src="https://github.com/user-attachments/assets/af239e4c-d40e-46ee-adb6-edc4c6c16ea8" />

<img width="1243" height="979" alt="Image" src="https://github.com/user-attachments/assets/9918af33-bb60-47d6-9b7d-8bced8d4eee5" />

## 3、keil中强制设定存储位置
```c
//添加__attribute__((section(".ARM.__at_0x24000000")))，就把变量定义到了0x24000000开始的AXI SRAM中
uint8_t test_buf_1[1000] __attribute__((section(".ARM.__at_0x24000000")));
uint8_t test_buf_2[1000] __attribute__((section(".ARM.__at_0x24000000")));

```

打开..\MDK-ARM\STM32H7_RAM\STM32H7_RAM.map文件可以查询变量存储位置
test_buf_1                               0x24000000   Data        1000  main.o(.ARM.__at_0x24000000)
test_buf_2                               0x240003e8   Data        1000  main.o(.ARM.__at_0x24000000)