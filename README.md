制作方法：
1. 从 GitHub fork 仓库：DragonBluep/uboot-mt7621
2. 进入自己仓库的 Actions 页面，从 Workflows 中选择 Build customized u-boot。
3. 点击 Run workflow 并配置相关参数诸如：Flash 类型，复位键引脚等。

4. 等待完成后从 Artifacts 下载最终得到的 u-boot 文件，刷写 u-boot-mt7621.bin 到第一个分区。


一些 Actions 输入参数的说明：
1. 闪存类型 Flash Type:
    NOR 或者 NAND 或者支持 NMBM 的 NAND。目前已知仅有 H3C 和 ASUS 的 Wi-Fi6 设备支持 NMBM。
    容量小于 64 MiB 一般就是 NOR Flash。
2. 分区表 MTD Partition Table：
    MTD 分区表必须和你要启动的固件的分区表相匹配，可以从固件源码的设备树 dts 文件中找到。
    u-boot 和 firmware 可以改大小，不可以重命名，其它分区的名字大小可以随便改，可以添加/删除分区。
3. 内核加载地址 Kernel Load Address：
    内核位于闪存中的偏移地址。内核加载地址是 firmware 分区前的所有分区大小的十六进制表示。
    比如说：分区表为：192k(u-boot),64k(u-boot-env),64k(factory),-(firmware)
    192kB + 64kB + 64kB = 320kB = 320 * 1024 B = 327680 B = 0x50000 B
    因此内核加载地址需要配置为 0x50000
4. 复位键 Reset Button GPIO：
    复位键的引脚 GPIO 编号，可以从固件源码的设备树 dts 文件中找到。
    取值范围 0 - 48，任意其它值为禁用，不适配此项将无法按复位键进入恢复模式。
5. 指示灯 Reset Button GPIO：
    指示灯的引脚 GPIO 编号，可以从固件源码的设备树 dts 文件中找到。
    取值范围 0 - 48，任意其它值为禁用，不适配此项按复位键时指示灯将不会闪烁。
6. CPU 主频 CPU Frequency：
    最好不要超频，使用 880 MHz 就好，这已经是 2013 年左右的芯片了，上限就在那里。
    取值范围 400 - 1200，频率太高可能会无法启动。
7. 内存频率 DRAM Frequency：
   不要超出内存芯片 datasheet 上的最大值，如果不清楚，板载 DDR2 就选800，板载 DDR3 就选 1200。
8. DDR 兼容模式 Use Old DDR Timing Parameters:
    勾选之后将会使用旧的 u-boot 1.1.3 的内存时序参数，有时候 DDR3（目前仅发现 512 MiB）
    存在兼容性问题无法启动，此时可以尝试勾选此项并适当调低内存频率。
9. 波特率 Baud Rate:
   串口波特率，老一点的设备一般是 57600。

注意事项：
1. 升级 u-boot 时，如果启动菜单选项有 Upgrade bootloader (advanced mode)，务必选这个。
2. 内存除以下特殊情况外只需要选择对应的大小和类型就行，一些需要额外关照的次品内存芯片（白片/黑片）：
    制造商 Winbond 型号 W9751G6KB_A02@1066MHz 请选择 DDR2-W9751G6KB-64MiB-1066MHz
    制造商 Winbond 型号 W971GG6KB25@800MHz 请选择 DDR2-W971GG6KB25-128MiB-800MHz
    制造商 Winbond 型号 W971GG6KB18@1066MHz 请选择 DDR2-W971GG6KB18-128MiB-1066MHz  
    以上几种情况需要将内存频率配置为蓝色的对应值。
    集成 128 MiB 的 MT7621DA，内存芯片表面没有标识或者百度 / 必应上找不到 datasheet 的 128 MiB DDR3 请选择 DDR3-128MiB-KGD
3. DDR3 要是启动困难可以尝试勾选 Use Old DDR Timing Parameters 以使用旧的内存时序参数 "0x0A000000"。
    DDR2 64MiB、DDR3 128/256 MiB 测试通过，DDR3 512 MiB 有一个坛友反馈正常，另一个反馈概率性启动困难，都是硬改的。
    研究了一下，BREED 和旧版 u-boot 里的默认参数都是 "0x0A00"。新的 u-boot 使用的是 "0x0C00"。
    所以具体使用 "0x0A00" 还是 "0x0C00" 自己凭感觉决定，建议使用新参数，启动困难再改，我胡诌的。
4. 硬改 512 MiB 内存最好挑选已知兼容的芯片：
    NT5CC256M16EP-EK  MT41K256M16TW-107  NT5CB256M16DP-EK  或者拆机帖见过的芯片
5. 常见的闪存都支持，但最好确定一下你的闪存是否在列表中
   SPI: uboot-mt7621/spi_flash_ids.c at main · DragonBluep/uboot-mt7621 (github.com)
   如果你的闪存 ID 无法在上述文件中找到，可以从以下 u-boot 最新源码中查找对应值，然后照葫芦画瓢修改并粘贴到源码中
   SPI: u-boot/spi-nor-ids.c at master · u-boot/u-boot (github.com)
6. 如果无法进入网页刷机界面，尝试以下解决办法：
   更换好一点的网线，最好长度一米以上，8 根线芯的，有时候协商不上千兆速率，会导致无法打开网页；
   在控制面板\网络和 Internet\网络连接中把以太网适配器右键禁用再重新启用；
   在 win10 / win11 设置中搜索网络重置并应用；
   尝试更新有线网卡驱动；

u-boot 使用方法：
1. 配置电脑的静态 IP 地址为：
   IP 地址:  192.168.1.2
   掩码:      255.255.255.0
   网关:      192.168.1.1（或者留空）
2. 网页刷写固件方法：
   a) 按住复位键不放，插入电源等待指示灯闪烁1秒 (1~6秒区间均可)后即可松开复位键。
   b) 从浏览器打开 http://192.168.1.1 进入恢复页面，上传固件写入到 Flash 中。

3. TFTP 加载 initramfs kernel 方法：
   a) 按住复位键不放，插入电源等待指示灯闪烁6秒自动停止后即可松开复位键。
   b) 使用 TFTP 服务器（如 tftpd64）上传 initramfs 镜像 "recovery.bin"。系统将被下载到 RAM 中启动，不会写入 Flash。

如何适配你的设备，以 RAISECOM MSG1500 为例：
1. 首先找到一份源码，确定复位键、指示灯的 GPIO 和分区表，或者从 TTL 输出的日志也行
    git.openwrt.org Git - target/linux/ramips/dts/mt7621_raisecom_msg1500-x-00.dts
2. 由上述信息得闪存类型为 NAND，复位键配置为 15，LED 配置为13， 分区表写为 512k(u-boot),512k(config),256k(factory),-(firmware)
    内核加载地址为 0x140000，将参数填入 Actions，就可以进行制作。
3. 再给个 JCG Q20 的例子：NAND, 256MiB DDR3, 1200MHz DDR, Baud 115200, Reset 18, LED 15, CPU 880MHz
    kernel addr:  0x180000, partition table: 512k(u-boot),512k(config),512k(factory),-(firmware)
