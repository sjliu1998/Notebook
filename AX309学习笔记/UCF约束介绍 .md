# UCF约束介绍 

[TOC]

## 1. 约束的分类

利用FPGA进行系统设计常用的约束主要分为3类：
1. 时序约束：主要用于规范设计的时序行为，表达设计者期望满足的时序条件，知道综合和布局布线阶段的优化算法等。
2. 布局布线约束：主要用于指定芯片I/O引脚位置以及指导软件在芯片特定的物理区域进行布局布线。
3. 其它约束：指目标芯片型号、接口位置、电气特性等约束属性。

## 2. 约束的主要作用

1. 提高设计的工作效率
	对很多数字电路设计来说，提高工作频率是非常重要的，因为高的工作频率意味着高效的电路处理能力，通过附加约束可以控制逻辑的综合、映射、布局和布线，以减少逻辑和布线的延迟，从而提高工作效率。
2. 获得正确的时序分析报告
	几乎所有的FPGA设计平台都包含静态时序分析工具，利用这类工具可以获得映射或者是布局布线后的时序分析报告，从而对设计的性能做出评估。静态时序分析工具以约束作为判断时序是否满足设计要求的标准，因此要求设计者正确输入约束，以便静态时序分析工具输出正确的时序分析报告。
3. 指定FPGA引脚位置与电气标准
	FPGA的可编程性使电路板设计加工和FPGA设计可以同时进行，而不必等FPGA引脚位置的完全确定，从而节约了系统开发时间。电路板加工完成后，设计者要根据电路板的走线对FPGA加上引脚位置约束，以保证FPGA与电路板正确连接。另外通过约束还可以指定I/O引脚所支持的接口标准和其他电气特性。为了满足日新月异的通信发展，Xilinx新型FPGA可以通过I/O引脚约束设置支持诸如AGP、BLVDS、CTT、GTL、GTLP、HSTL、LDT、LVCMOS、LVDCI、LVDS、LVPECL、LVDSEXT、LVTTL、PCI、PCIX、SSTL、ULVDS等丰富的I/O接口标准。
4. 利于模块化设计
	通过区域约束还能在FPGA上规划各个模块的实现区域，通过物理布局布线约束完成模块化设计等。

## 3. UCF约束文件的概念

FPGA设计中的约束文件有3类：用户设计文件（.UCF文件）、网表约束文件（.NCF文件）以及物理约束文件（.PCF文件），可以完成时序约束、管脚约束以及区域约束。3类约束文件的关系为：用户在设计输入阶段编写UCF文件，然后UCF文件和设计综合后生成NCF文件，最后再经过实现后生成PCF 文件。
UCF文件是ASC 2码文件，描述了逻辑设计的约束，可以用文本编辑器和Xilinx约束文件编辑器进行编辑。NCF约束文件的语法和UCF文件相同，二者的区别在于： UCF文件由用户输入，NCF文件由综合工具自动生成，当二者发生冲突时，以UCF文件为准，这是因为UCF的优先级最高。PCF文件可以分为两个部分：一部分是映射产生的物理约束，另一部分是用户输入的约束，同样用户约束输入的优先级最高。一般情况下，用户约束都应在UCF文件中完成，不建议直接修改 NCF文件和PCF文件。 

## 4. 约束设计

### 4.1. 时序约束

时序约束分为周期约束、I/O时序约束、分组约束和专门约束。

#### 4.1.1. 周期约束

周期约束是一个基本时序和综合约束，它附加在时钟网络上，时序分析工作根据周期约束检查时钟域内所有同步器件的时序是否满足要求，它将检查与同步时序约束端口相连接的所有路径的延迟，但不会检查PAD到寄存器路径。
周期约束的语法如下：
> TIMESPEC “TS_identifier”=PERIOD “TNM_reference” period {High|low} [high_or_low_time]
> 说明：
> (1) TIMESPEC是一个基本时序相关约束标识。TS_identifier包括字母TS和一个标识符identifier共同组成一个时序规范。
> (2) 参数period为要求的时钟周期，可以使用ps、ns、us或者ms等单位，大小写都可以，缺省单位为ns。
> (3) “{}”为必选项，HIGH|LOW关键词指出时钟周期里的第一个脉冲是高电平还是低电平。
> (4) “[]”内为可选项，high_or_low_time为脉冲的延续时间，缺省单位是ns，默认占空比为50%。
> (5) 定义时钟周期约束时，首先需要对待约束的时钟网络上附加一个TNM_NET约束，把由该时钟驱动的所有同步器件定义为一个分组，然后使用TIMESPEC约束定义时钟周期。

周期约束设计实例：
> NET “usr_clk” TNM_NET= “usr_clk_i”;
> TIMESPEC “TS_usr_clk_i”=PERIOD “usr_clk_i” 5.0ns HIGH 50%
> 含义：
> 第一条约束定义时钟usr_clk驱动的所有同步器件为一个分组；
> 第二条约束定义其周期为5ns，即200MHZ,占空比为50%。

#### 4.1.2. I/O时序约束

I/O时序约束定义了时钟和I/O接口之间的时序关系，只用于与I/O接口相连的信号，不能用于内部信号。
I/O时序约束可以约束输入数据、输出数据相对于时钟的时序关系，从而在综合实现中调整布局布线，是正在开发的FPGA的输入建立时间、输出保持时间保持系统要求。
I/O时序约束的语法如下：
> OFFSET=IN “offset_time” [units] BEFORE “clk_name” [TIMEGRP “group_name”];
> OFFSET=OUT “offset_time” [units] AFTER “clk_name” [TIMEGRP “grout_name”];
> 参数：
> OFFSET_IN_AFTER：输入数据在有效时钟到达多长时间后可以到达芯片的输入引脚；
> OFFSET_IN_BEFORE：数据比相应的有效时钟沿提前多少时间到来；
> OFFSET_OUT_AFTER：输出数据在有效时钟沿之后多长时间稳定下来；
> OFFSET_OUT_BEFORE：下一个时钟信号到来之前多长时间必须输出数据。
> 说明：
> (1) OFFSET、IN、BEFORE是I/O时序约束输入建立时间标识，具体含义为：输入数据与时钟的时序关系满足offset_time定义的时间。
> (2) OFFSET、OUT、AFTER是I/O时序约束输出保持时间标识，具体含义为：输出数据与时钟的时序关系满足offset_time定义的时间。
> (3) ”offset_time”是约束要求的时间。
> (4) ”clk_name”为参考时钟。
> (5) [TIMEGRP “grout_name”]为约束的寄存器组。

I/O时序约束设计实例：
> INST “io_emif_data<0>” TNM=TS_emif_data;
> INST “io_emif_data<1>” TNM=TS_emif_data;
> INST “io_emif_data<2>” TNM=TS_emif_data;
> INST “io_emif_data<3>” TNM=TS_emif_data;
> INST “io_emif_data<4>” TNM=TS_emif_data;
> INST “io_emif_data<5>” TNM=TS_emif_data;

INST “io_emif_data<6>” TNM=TS_emif_data;

INST “io_emif_data<7>” TNM=TS_emif_data;

NET “IO_emif_clk” TNM_NET= I_emif_clk;

TIMEGRP “TS_emif_data” OFFSET = OUT 7ns AFTER “I_emif_clk”;

约束定义TS_emif_data寄存器组与时钟I_emif_clk的关系为时钟有效后7ns输出TS_emif_data寄存器的可靠数据。

 

 

4.1.3分组约束

 

分组约束是将一些具有相同时序要求的器件归为一组，进行相同的时序约束。

分组约束的语法如下：

{NET|INST} “net_name”  TNM_NET= [predefined_group] identifier;

{NET|INST|PIN}“net_or_pin_or_inst_name”  TNM =[predefined_group] identifier;

INST、NET和PIN为信号，引脚等关键词。

INST is an element such as a flip flop, register or pad in a design. NET is a signal path, a route between one point (such as a flip flop, register or pad) to another

 

TNM为分组约束关键词

TNM_NET为分组约束关键词，其作用于TNM加在网上是基本相同，即把该网线所在路径上的所有有效同步元件作为命名组的一部分。

不同之处在于当TNM约束加在PAD NET 上时，TNM的值将被赋予PAD，而不是该网线所在的路径上的同步元件，即TNM的约束不能穿过IBUF。而用TNM_NET约束就不会出现这种情况。

identifier为标识符

predefined_group为预先定义组标识符

 

4.1.4专门约束

约束文件设计的一般策略是首先设定整体约束，例如PERIOD、OFFSET等，然后对局部的电路附加专门约束，这些专门约束通常比整体约束宽松，通过在可能的地方尽量放松约束可以提高布局布线通过率，减小布局布线的时间。

 

FROM_TO约束

FROM_TO约束在两个定义的组之间进行时序约束，对两者之间的逻辑和布线延迟进行控制。

语法如下：TIMESPEC “TS_name”= FROM “group1” TO “group2” value;

其中value为延迟时间，可以使具体数值或表达式。

 

MAXDELAY约束

MAXDELAY约束定义了特定路径上的最大延迟。

语法如下：NET “net_name” MAXDELAY = value units;

 

 

4.2.2布局布线约束

布局布线约束包括引脚约束与位置约束。

 

4.2.1引脚约束

约束FPGA输入输出引脚的具体位置。

引脚约束的语法如下：NET “net_name” LOC= “PIN”;

说明：

（1）NET,LOC引脚约束关键词

（2）“net_name”为FPGA内部定义的输入输出信号名称；

（3）“PIN”为FPGA实际引脚名称。

 

【例3】引脚约束实例

NET “sys_rst_n” LOC= “J12”;

 

4.2.2位置约束

位置约束是通过约束语法将设计中的某些硬件结构约束到指定的位置。

（1）位置约束的语法如下：INST “instance_name” LOC=location;

对设计中的硬件约束到具体位置，可以约束的硬件结构包括：寄存器、IOB、LUT、BRAM、乘法器、PLL等。

（2）INST “instance_name” RLOC= location;

对设计中的硬件约束到相对位置， 可约束的硬件结构包括：寄存器、IOB、LUT、BRAM、乘法器、PLL等。必须与RLOC_ORIGIN配套使用。

（***）INST “instance_name” RLOC_ORIGIN =location；与RLOC对应，指定RLOC的起始位置约束，与RLOC配套使用。

（***）INST “instance_name“ HU_SET=value;高级属性定义约束，定义独立的组，与RLOC配套使用，以保持结构的完整性。

4.3其他约束

除了时序约束以及引脚和位置约束外，Xilinx公司还提供了其他一些约束，例如：

（1）PULLDOWN约束

NET “pad_net_name” PULLDOWN

说明：下拉约束，输出低电平，以避免在无驱动时三态门的输出悬空。

（2）PULLUP约束

NET “pad_net_name” PULLUP

说明：上拉约束，输出高电平，以避免在无驱动时三态门的输出悬空。

（3）IOSTANDARD

NET “pad_net_name” IOSTANDARD = iostandard_name

说明：输入输出引脚电平约束

（4）DRIVE

INST “instance_name” DRIVE= {2|4|6|8|12|16|24};

说明：输出电流能力约束，可选为2mA, 4mA, 6mA, 8mA, 12mA, 16mA, 24mA电流输出，默认值为12mA输出。

（5）SLEW

NET “FAST_OUT” SLEW=”FAST”

说明：输出斜率控制，可选为FAST以及SLOW，可以提高设计的信号完整性。

 

（五）电气标准

LVDS: low vvoltage differential signal 低压差分信号 HSTL： High Speed  Transciever Logic    高速收发逻辑 SSTL： Stub Series Terminated  Logic   短截线串联终端逻辑。 这三种是不同的接口逻辑标准：

LVDS 常用在液晶屏的数据传输，我们的笔记本的屏幕，大度都用LVDS接口；HSTL 和SSTL 用在与外部存储器 如DDR2 DDR SDRAM 接口

 

【LVDS】 1.LVDS有两种 LVDS_25,LVDS_33(这两种在做输出时，相应BANK的VCCO 必须分别是2.5V和3.3V) 2.LVDS的输入端，需要端接100欧姆的电阻（并在差分对之间），Spartan6支持内部端接，只需要在LVDS原语例化时将DIFF_TERM设置成TRUE.以前这是只能在Viretx系列上才能有的福利

 

【HSTL和SSTL】

1.HSTL：HSTL最主要的应用是可以用于高速存储器读可。传统的慢速存储器访问时间阻碍了高速处理器的运算操作。在中频区域（100MHz和180MHz之间），可供选择基于单端信号的I/O结构有：HSTL、GTL/GTL+、SSTL和低压TTL（LVTTL）。在180MHz以上的范围，HSTL标准是唯一可用的单端I/O接口。利用HSTL的速度，快速I/O接口明显地提高了整个系统的性能。HSTL是高速存储器应用的I/O接口选择，同时也很完美地提供了驱动多个内存模块地址总线的能力。 HSTL到底用在什么地方，目前还不是很清楚 2.SSTL这个接触过。DDR2的部分逻辑接口电平就是用的这种标准。 SSTL细分的话有：SSTL3（3.3V） SSTL2 （2.5V） SSTL18（1.8V）  SSTL15（1.5V） 

其中SSTL3 用于SDRAM, SSTL2 DDR 驱动，SSTL18 用在DDR2 ，SSTL15用在DDR3，  SSTL 还会分等级 常用的是等级I 和等级II如 SSTL18_II

 

（六）其他

6.1通配符 在UCF文件中，通配符指的是“*”和“?”。“*”可以代表任何字符串以及空，“?”则代表一个字符。在编辑约束文件时，使用通配符可以快速选择一组信号，当然这些信号都要包含部分共有的字符串。例如： NET "*CLK?" FAST; 将包含“CLK”字符并以一个字符结尾的所有信号，并提高了其速率。 在位置约束中，可以在行号和列号中使用通配符。例如： INST "/CLK_logic/*" LOC = CLB_r*c7; 把CLK_logic层次中所有的实例放在第7列的CLB中。 6.2定义设计层次  在UCF文件中，通过通配符*可以指定信号的设计层次。其语法规则为： * 遍历所有层次 Level1/* 遍历level1及以下层次中的模块 Level1/*/ 遍历level1种的模块，但不遍历更低层的模块 

6.3管脚和区域约束语法  LOC约束是FPGA设计中最基本的布局约束和综合约束，能够定义基本设计单元在FPGA芯片中的位置，可实现绝对定位、范围定位以及区域定位。此外， LOC还能将一组基本单元约束在特定区域之中。LOC语句既可以书写在约束文件中，也可以直接添加到设计文件中。换句话说，ISE中的FPGA底层工具编辑器（FPGA Editor）、布局规划器（Floorplanner）和引脚和区域约束编辑器的主要功能都可以通过LOC语句完成。 

 

INST "instance_name " LOC = location; 其中“location”可以是FPGA芯片中任一或多个合法位置。如果为多个定位，需要用逗号“,”隔开，如下所示： LOC = location1,location2,...,locationx;

目前，还不支持将多个逻辑置于同一位置以及将多个逻辑至于多个位置上。需要说明的是，多位置约束并不是将设计定位到所有的位置上，而是在布局布线过程中，布局器任意挑选其中的一个作为最终的布局位置。 范围定位的语法为： INST “instance_name” LOC=location:location [SOFT]; 

 

 

使用LOC完成端口定义时，其语法如下： NET "Top_Module_PORT" LOC = "Chip_Port"; 其中，“Top_Module_PORT”为用户设计中顶层模块的信号端口，“Chip_Port”为FPGA芯片的管脚名。 

 

 

6.4 LOC属性说明 LOC语句通过加载不同的属性可以约束管脚位置、CLB、Slice、TBUF、块RAM、硬核乘法器、全局时钟、数字锁相环（DLL）以及DCM模块等资源，基本涵盖了FPGA芯片中所有类型的资源。由此可见，LOC语句功能十分强大，表4-5列出了LOC的常用属性。 

 
