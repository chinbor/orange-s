# Eighth Chapter

---

### 一. 8253/8254定时器

#### 1. 8253/8254 的内部结构

![](https://img-blog.csdn.net/20151226161454906)  
1.1 数据总线缓冲器

> 8 位双向三态缓冲器，用于连接系统数据总线和8253 内部总线。以便编程时:
>
> * 处理器对8253写入控制字
> * 处理器写入计数初值
> * 处理器从计数器读取计数值。

1.2 读/写控制逻辑

> 8253 内部的控制电路，读写逻辑接收来自CPU 的控制信号，经过组合，产生对8253各部分的控制。具体为：
>
> * A1 、A0——用来对3个计数器和控制字存器进行寻址。
> * RD——读信号。当RD为低电平时有效，此时，表示CPU正在对8253的一个计数器进行读操作。
> * WR——写信号。当WR为低电平时有效，此时，表示CPU正在对8253的几个计数器进行写操作。
> * CS——片选信号。只有在CS为持续低电平的情况下，RD和WR才会受到确认，否则会被忽略。

1.3 控制字寄存器

> 控制字寄存器接收CPU 送来的控制字，决定每个计数器的工作方式、读写格式和计数的数制。当A1A0=11 时，用来接收CPU 输出的控制字。该寄存器是个只能写入寄存器。

1.4 计数器（计数器0、计数器计1、数器2）

> 8253 内部有3 个相互独立且结构功能完全相同的定时/计数器：0、1 和2。每个计数器包含一个控制字寄存器、16 位的初值寄存器CR（Count Register）、减“1”计数器CE（Counting Element）和16 位输出锁存寄存器OL（Out put Latch）。
>
> 减“1”计数器的初始值由程序设定，是初值寄存器的内容。计数器的当前值存放在输出锁存寄存器，CPU 可以在需要的时候读出这个值。  
> ![](https://img-blog.csdn.net/20151226162015115)  
> 每个计数器有3 根信号线：时钟输入信号CLK、门控信号输入GATE 和输出信号OUT。
>
> 计数器工作时，每出现一个CLK脉冲**\(具体说是第一个下降沿是将cr寄存器的内容取到ce寄存器，第二个clk下降沿才减一\)**，减“1”计数器中的计数值减1，当计数值减为0 时，通过OUT 输出结束信号，表明计数执行单元已为0，输出信号的波形由工作方式确定。当CLK是一个周期性的时钟信号时，计数器为定时器功能；当CLK是一个非周期性事件计数信号时，此时为计数器功能。8253 的定时时间取决于时钟脉冲的频率和计数器的初值：
>
> ```
>                      定时时间T = 时钟脉冲周期tc × 计数初值n
> ```
>
> 此外，可以通过GATE 引脚上的门控信号允许或停止计数器的过程。

#### 2. 8253 的外部引脚

* D7 ～D0：双向数据线，与系统数据总线相连，供CPU 对8253 进行读/写操作，进行信息交换。
* CS：片选线，输入低电平有效。当CS= 0 时，表示CPU 选中8253 芯片，可对8253 进行读/写操作；当CS= 1 时，表示CPU 未选中8253 芯片。CS信号是由CPU 输出的高位地址（通常A9A9～A2A2）和AEN 信号经过电路译码后产生。
* A1 ～A0：地址输入信号，用于选择8253 内部寄存器。当CS为低电平有效时，选中8253芯片，然后通过A1 A0 组合编码决定选中计数器

  > ![](https://img-blog.csdn.net/20151226163142674)

* RD 和WR ：读/写控制输入信号端，低电平有效。当CS= 0，RD= 0 时，表示CPU 可对8253 的一个计数器进行读操作；当CS= 0，WR = 0 时，表示CPU 可对8253 的一个计数器或控制字寄存器进行写操作。

* GATE ：门控输入信号。是一根外部控制计数器工作的输入信号线，用作控制启动或中止定时/计数。对于8253 这6 种不同的工作方式，GATE 信号的有效方式也不同，可以是上升沿信号有效或是电平信号有效。

* CLK：外部脉冲信号输入端CLK0、CLK1、CLK2，分别为计数器0、计数器1、计数器2的脉冲信号输入端。计数器对该引脚输入的脉冲信号进行计数，CLK脉冲信号可以由系统时钟、系统时钟分频或者其他脉冲源提供。CLK 输入脉冲可以是均匀的、连续的或周期精确的信号，也可以是不均匀的、不连续的或非周期的信号。如果输入脉冲是精确的时钟信号，则8253 起定时作用；如果输入脉冲是非周期的信号，则8253 起计数作用。

* OUT ：定时时间到或计数减为零的向外输出信号端OUT0、OUT1、OUT2，分别为计数器0、计数器1、计数器2 的脉冲信号输出端。无论8253 的计数器工作在什么方式，当计数器减到0 时，信号输出端OUT 必定有一个电平或脉冲信号输出，用以指示定时时间到或者计数结束。该信号可供CPU 检测，也可以作为中断请求信号。

#### 3. 8253/8254 的工作方式（具体请自行查阅，orange's采用了方式2，这样可以产生周期性的时钟中断，并且为10ms）

* 方式0：计数结束产生中断
* 方式1：可编程单稳态触发器
* ** 方式2：分频器（Rate Generator）**
* 方式3：方波发生器
* 方式4：软件触发选通脉冲
* 方式5：硬件触发选通脉冲

#### 4. 8253编程命令

** 编程有两条原则必须严格遵守: **

> * \(1\)对计数器设置初始值前必须先写控制字。
> * \(2\)初始值设置时,要符合控制字中的格式规定,即只写低位字节还是写高位字节,或者高低位字节都写,控制字中一旦规定,具体初始值设定时就要一致。

4.1 8253 初始化编程  
初始化工作有2点：

> * \(1\)写入控制字；
> * \(2\)按控制字的要求写入计数初值。

8253 的每个计数器内有一个8 位控制寄存器，用来存放CPU 写入的工作方式控制字，工作方式控制字格式如下图所示，该寄存器只能执行写入操作，不能执行读出操作。8253 内部的3 个计数器在结构上相互独立，在使用时须对指定的计数器写入方式控制字，写入控制字的I/O 地址相同，要求A1A0=11。  
![](https://img-blog.csdn.net/20151226171525319)

> * D0 ：计数码选择。8253 提供丰富的计数制，该位用来决定计数器在减1 计数过程中采用的是二进制还是十进制，0 表示采用二进制计数制，1 表示采用BCD \(10进制\)计数制。
>
> > 在BCD 计数制下，写入计数值的范围为0000～9999，最大值为0000，表示的是10进制的10000；在二进制计数制下，写入计数值的范围为0000～FFFFH，最大值为取0000H ,即十进制数的65536
>
> * D3 ～D1：M2、M1、M0 为8253 的工作方式的选择。8253 的每个计数器的6 种工作方式如下表所示。
>
> | D1 | D2 | D3 | 工作方式 |
> | :---: | :---: | :---: | :---: |
> | 0 | 0 | 0 | 方式0 |
> | 0 | 0 | 1 | 方式1 |
> | x（表示任意） | 1 | 0 | 方式2 |
> | x | 1 | 1 | 方式3 |
> | 1 | 0 | 0 | 方式4 |
> | 1 | 0 | 1 | 方式5 |
>
> * D5 、D4：RW1 和RW0 读写格式控制。计数值的读出或写入可按字节或字两种方式进行操作。D5D4 =01，只读写低8 位，高8 位自动置0；D5D4 = 10，只读写高8 位，低8 位自动置0；D5 D4 = 11，必须先读写低8 位，后读写高8 位；D5D4 = 00，锁存命令，用于把当前计数值存入输出锁存器，供以后读取。
> * D7、D6：计数器选择。D7D6 = 00，选择计数器0；D7D6 = 01，选择计数器1；D7D6 = 10， 
>   选择计数器2；D7D6 = 11，在8253 中为非法编码，在8254 中用于读回命令。

---

### 二. 8259a可编程中断控制器

#### 1. 可编程中断控制器

可编程中断控制器（PIC - Programmable Interrupt Controller\)是微机系统中管理设备中断请求的管理者。当PIC向处理器的INT引脚发出一个中断信号时，处理器会立刻停下当时所做的事情并询问PIC需要执行哪个中断服务请求。PIC则通过向数据总线发出与中断请求对应的中断号来告知处理器要执行哪个中断服务过程。处理器则根据读取的中断号通过查询中断向量表（在32位保护模式下是中断描述符表）取得相关设备的中断向量（即中断服务程序的地址）并开始执行中断服务程序。当中断服务程序执行结束，处理器就继续执行被中断信号打断的程序。

#### 2. 8259A的级联

在80X86微机机系统中采用了8259A可编程中断控制器芯片。每个8259A芯片可以管理8个中断源。通过多片级联方式，8259A能构成最多管理64个中断向量的系统（orange's中只采用了2级级联）。

在PC/AT系列兼容机中，使用了两片8259A芯片，共可管理15级中断向量。其级连示意图如下 ：  
![](https://img-blog.csdn.net/20180304202408628?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzQ5MDg5Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
其中从芯片的INT引脚连接到主芯片的IR2引脚上，即8259A从芯片发出的中断信号将作为8259A主芯片的IRQ2输入信号。

IRQ9引脚的作用与IRQ2相同， 即PC/AT机利用硬件电路把IRQ2引脚重新定向到了PIC的IRQ9引脚上，并利用BIOS中的软件把IRQ9的中断int 71重新定向到了IRQ2的中断int 0x0A的中断处理过程。这样一来可使得任何使用IRQ2的PC/XT的8位设配卡在PC/AT机下面仍然能正常使用，做到了PC机的向下兼容。

为什么要把IRQ2重定向到IRQ9上？

> 早期的IBM PC/XT只有一个8259A，这样就只能处理8种IRQ。但很快就发现这根本不能满足需求，所以到了IBM PC/AT，又以级连的方式增加了一个8259A，这样就可以多处理7种IRQ。原来的8259A被称作Master PIC，新增的被称作Slave PIC。但由于CPU只有1根中断线，Slave PIC不得不级连在Master PIC上，占用了IRQ2，那么在IBM PC/XT上使用IRQ2的设备将无法再使用它，但新的系统又必须和原有系统保持兼容，怎么办？
>
> 由于新增加的Slave PIC在原有系统中不存在，所以，设计者从Slave PIC的IRQ中挑出IRQ9，要求软件设计者将原来的IRQ2重定向到IRQ9上，也就是说IRQ9的中断服务程序需要去掉用IRQ2的中断服务程序。这样，将原来接在IRQ2上的设备现在接在IRQ9上，在软件上只需要增加IRQ9的中断服务程序，由它调用IRQ2的中断服务程序，就可以和原有系统保持兼容。而在当时，增加的IRQ9中断服务程序是由PC开发商开发的BIOS提供的，不需要用户进行另外设置，所以就从根本上保证了兼容。

#### 3. 8259A的工作原理

在总线控制器控制下，8259A芯片可以处于编程状态和操作状态。编程状态是CPU使用IN或OUT指令对8259A芯片进行初始化编程的状态。一旦完成了初始化编程，芯片即进入操作状态，此时芯片即可随时响应外部设备提出的中断请求（IRQ0 -IRQ15），同时系统还可以使用操作命令字随时修改其中断处理方式。通过中断判优选择，芯片将选中当前最高优先级的中断请求作为中断服务对象，并通过CPU引脚INT通知CPU中断请求的到来，CPU响应后，芯片从数据总线D7-D0将编程设定的当前服务对象的中断号送出，CPU由此获取对应的中断向量值，并执行中断服务程序。

一个8259A芯片的逻辑框图如下：  
![](https://img-blog.csdn.net/20180304202628824?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzQ5MDg5Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
图中，中断请求寄存器 IRR \(Interrupt Request Register\)用来保存中断请求输入引脚上所有请求，寄存器的8个比特位（D7—D0）分别对应引脚IR7—IR0。中断屏蔽寄存器 IMR \(Interrup Mask Register\)用于保存被屏蔽的中断请求线对应的比特位，哪个比特位被置1就屏蔽哪一级中断请求。即IMR对IRR进行处理，其每个比特位对应IRR的每个请求比特位。对高优先级输入线的屏蔽并不会影响低优先级中断请求线的输入。优先级解析器PR\(Priority Resolver\) 用于确定 IRR 中所设置比特位的优先级，选通最高优先级的中断请求到正在服务寄存器ISR \(In-Service Register\)中。ISR中保存着正在接受服务的中断请求。

来自各个设备的中断请求线分别连接到8259A的IR0—IR7引脚上。当这些引脚上有一个或多个中断请求信号到来时，中断请求寄存器 IRR 中相应的比特位被置位锁存。此时若中断屏蔽寄存器 IMR 中对应位被置位，则相应的中断请求就不会送到优先级解析器中。未屏蔽的中断请求被送到优先级解析器之后，优先级最高的中断请求会被选出。此时8259A就会向CPU发送一个INT信号，而CPU则会在执行完当前的一条指令之后向8259A发送一个INTA（INTERRUPT ACKNOWLEDGE）来响应中断信号。8259A在收到这个响应信号之后就会把所选出的最高优先级中断请求保存到正在服务寄存器ISR中，即ISR中对应比特被置位。与此同时，中断请求寄存器 IRR 中的对应比特位被复位，表示该中断请求开始被处理。此后，CPU会向8259A发出第2个INTA脉冲信号，该信号用于通知 8259A送出中断号。在该脉冲信号期间，8259A就会把一个代表中断号的8位数据发送到数据总线上供CPU读取。

到此为止，CPU中断周期结束。如果8259A使用的是自动结束中断\(AEOI，Automatic End of Interrupt\) 方式，那么在第2个 INTA 脉冲信号的结尾处正在服务寄存器 ISR 中的当前服务中断比特位就会被复位。若8259A 使用非自动结束方式，那么在中断服务程序结束时，程序就需要向8259A发送一个结束中断（EOI）命令以复位 ISR 中的比特位。如果中断请求来自级联的第2个8259A芯片，那么就需要向两个芯片都发送EOI命令。此后8259A就会去判断下一个最高优先级的中断，并重复上述处理过程。

#### 4. 8259A 的中断优先级

4.1 固定优先级方式

* 在固定优先级方式中，IR7～IR0 的中断优先级是由系统确定的。优先级由高到低的顺序是：IR0, IR1, IR2, …, IR7。其中，IR0的优先级最高，IR7的优先级最低。

4.2 自动循环优先级方式

* 在自动循环优先权方式中，IR7～IR0的优先级别是可以改变的，而且是自动改变。其变化规律是：当某一个中断请求IRi服务结束后，该中断的优先级自动降为最低，而紧跟其后的中断请求IR\(i＋1\)的优先级自动升为最高。

4.3 中断嵌套方式

* 普通嵌套方式

> 也叫做完全嵌套或者普通完全嵌套。此方式是8259A在初始化时默认选择的方式。其特点是：IR0优先级最高，IR7优先级最低。在CPU中断服务期间，若有新的中断请求到来，只允许比当前服务的优先级更高的中断请求进入，对于“同级”或“低级”的中断请求则禁止响应。

* 特殊完全嵌套方式

> 其特点是：IR7～IR0 的优先级顺序与普通嵌套方式相同；不同之处是在CPU中断服务期间，除了允许高级别中断请求进入外，还允许同级中断请求进入，从而实现了对同级中断请求的特殊嵌套。
>
> 在多片8259A级联的情况下，主片通常设置为特殊完全嵌套方式，从片设置为普通嵌套方式。当主片响应某一个从片的中断请求时，从片中的 IR7～IR0 的请求都是通过主片中的某个IRi请求引入的。因此从片的 IR7～IR0 对于主片IRi来说，它们属于同级，只有主片工作于特殊完全嵌套方式时，从片才能实现完全嵌套。

4.4 中断屏蔽方式

* 普通屏蔽方式

  > 写入操作命令字 OCW1，将中断屏蔽寄存器\(IMR\)中的Di位置1，以达到对IRi（i＝0～7）中断请求的屏蔽。  
  > ![](https://img-blog.csdn.net/20180304204017293)  
  > 若 Mi=1，则屏蔽对应中断请求级IRi；若 Mi=0，则允许IRi. 另外，屏蔽高优先级并不会影响其他低优先级的中断请求。

* 特殊屏蔽方式

  > 8259A工作在特殊屏蔽方式时，所有未被屏蔽的优先级中断请求（较高的和较低的）均可在某个中断过程中被响应，即低优先级别的中断可以打断正在服务的高优先级中断。

> 在特殊屏蔽方式中，可在中断服务子程序中用中断屏蔽命令屏蔽当前正在处理的中断级，同时可使其在ISR中的对应位清零，这样一来不仅屏蔽了当前正在处理的中断级，而且也真正开放了较低级别的中断请求。在这种情况下，虽然CPU仍然继续执行较高级别的中断服务子程序，但由于ISR中对应位已经清零，就如同没有响应该中断一样。所以，此时对于较低级别的中断请求，CPU可以响应。

4.5 中断结束方式  
** 中断结束方式是指CPU为某个中断请求服务结束后，应及时清除中断服务标志位，否则就意味着中断服务还在继续，致使比它优先级低的中断请求无法得到响应。中断服务标志位存放在中断服务寄存器（ISR）中，当某个中断源IRi被响应后，ISR中的Di位被置1，服务完毕应及时清除。8259A提供了三种中断结束方式。**

* 自动结束方式

> 当 ICW4 中的自动中断结束 \(AEOI\) 比特位置位时，通过CPU发出的第二个中断响应信号INTA脉冲的后沿，将ISR中的中断服务标志位清除。这种中断服务结束方式是由硬件自动完成的。
>
> 需要注意的是：ISR中为“1”位的清除是在中断响应过程中完成的，并非中断服务子程序的真正结束，因8259A并没有保存任何标志来表示当前服务尚未结束，此时，若有中断请求出现，且IF（x86的EFLAGS寄存器b9，中断允许位）＝1，则无论其优先级如何\(比本级高、低或相同\)，都将得到响应，尤其是当某一中断请求信号被CPU响应后，如不及时撤销，就会再次被响应（二次中断）。这样可能会打乱正在服务的程序。因此这种方式只适用在中断请求信号的持续时间有一定限制，且没有中断嵌套的场合。

* 普通结束方式

> 普通结束方式是通过在中断服务子程序中编程写入操作命令字OCW2，向8259A传送一个普通中断结束（EOI，end of interrupt）命令（命令中不指定要复位的中断级）来清除ISR中优先级别最高的置位。
>
> 由于这种结束方式是清除ISR中优先权级别最高的那个置位，适合使用在完全嵌套方式下。因为在完全嵌套方式下，中断优先级是固定的，8259A总是响应优先级最高的中断，保存在ISR中的最高优先级的对应位，一定对应于正在执行的服务程序。

* 特殊结束方式

> 特殊结束方式是通过在中断服务子程序中编程写入操作命令字OCW2，向8259A传送一个特殊EOI命令（命令中指定出要复位的中断级）来清除ISR中的指定位。
>
> 在某些情况下，中断请求的响应顺序并不遵从固定的优先级。比如8259A工作在特殊屏蔽方式时，低优先级中断可以打断正在服务的高优先级中断，高优先级中断也可以打断正在服务的低优先级中断，此时，根据ISR的内容无法确定出刚刚所处理的中断。这就需要在EOI命令中指定出要复位的中断级。

#### 5. 8259A的编程

5.1 初始化命令字  
在8259A可以正常工作之前，必须首先设置初始化命令字 ICW \(Initialization Command Words\)寄存器组的内容。而在其工作过程中，则可以使用写入操作命令字 OCW \(Operation Command Words\)寄存器组来随时设置和管理8259A的工作方式。

A0线用于选择操作的寄存器。在PC/AT微机系统中，当A0=0时芯片的端口地址是0x20\(主芯片\)和0xA0\(从芯片）；当 A0=1时端口就是0x21\(主芯片\)和0xA1\(从芯片\)。

初始化命令字的编程操作流程如下图所示。由图可以看出，对 ICW1和 ICW2 的设置是必需的。而只有当系统中包含多片 8259A 芯片并且是级联的情况下才需要对 ICW3 进行设置。这需要在 ICW1 的设置中明确指出。 另外，是否需要对 ICW4 进行设置也需要在 ICW1 中指明。  
![](https://img-blog.csdn.net/20180304213512671)

** ICW1 **

当发送的字节第 5 比特位\(D4\)=1，并且地址线 A0=0 时，表示是对 ICW1 编程。此时对于 PC/AT 微机系统的多片级联情况下，8259A 主芯片的端口地址是 0x20，从芯片的端口地址是 0xA0。

ICWl 的格如下：  
![](https://img-blog.csdn.net/20180304213619802?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzQ5MDg5Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
![](https://img-blog.csdn.net/20180304213632710?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzQ5MDg5Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

** ICW2 **

ICW2 用于设置芯片送出的中断号的高5位。在设置了 ICW1 之后，当 A0=1 时表示对 ICW2 进行设置。此时对于PC/AT微机系统的多片级联情况下，8259A主芯片的端口地址是0x21，从芯片的端口地址是0xA1。

ICW2 格式如下。  
![](https://img-blog.csdn.net/20180304213725656?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzQ5MDg5Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
在使用 8086/88 处理器的系统或兼容系统中 T7~T3 是中断号的高5位，与 8259A 芯片自动设置的低3位（8259A 按 IR0～IR7 三位编码值自动填入）组成一个8位的中断号。8259A在收到第2个中断响应脉冲INTA时会把此中断号送到数据总线上，以供 CPU 读取。  
![](https://img-blog.csdn.net/20180304213808645)  
Linux-0.11 系统把主片的 ICW2 设置为 0x20，表示主片中断请求0~7级对应的中断号是 0x20~0x27；把从片的 ICW2 设置成 0x28，表示从片中断请求8~15级对应的中断号是 0x28~0x2f。

** ICW3 **

![](https://img-blog.csdn.net/20180304213847731?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzQ5MDg5Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

主芯片的端口地址是0x21，从芯片的端口地址是0xA1。

对于主片，Si=1,表示IRi接从片的INT引脚。说得啰嗦点，就是主片 S7~S0 各比特位对应级联的从片。哪位为1则表示主片的该中断请求引脚IR上信号来自从片， 否则对应的IR引脚没有连从片。

对于从片，ID2~ID0 三个比特位对应各从片的标识号，即连接到主片的中断级。当某个从片接收到级联线（CAS2—CAS0）输入的值与自己的 ID2~ID0 相等时，表示此从片被选中。此时该从片应该向数据总线发送自己当前被选中的中断请求的中断号。

Linux-0.11 内核把8259A主片的 ICW3 设置为 0x04，即 S2=1，其余各位为0。表示主芯片的 IR2 引脚连接一个从芯片。从芯片的 ICW3 被设置为 0x02，即其标识号为2。表示此从片连接到主片的IR2引脚。 因此，中断优先级的排列次序为：0级最高，1级次之，接下来是从片上的 8~15 级，最后是主片的 3~7 级。

** ICW4 **   
当 ICW1 的位0 \(IC4\)置位时，表示需要 ICW4。地址线 A0=1，主芯片的端口地址是0x21，从芯片的端口地址是0xA1。  
![](https://img-blog.csdn.net/20180304214103377)

![](https://img-blog.csdn.net/20180304214138112)  
Linux-0.11内核送往8259A主芯片和从芯片的 ICW4 命令字的值均为 0x01。表示 8259A 芯片被设置成普通全嵌套、非缓冲、非自动结束中断方式，并且用于 8086 及其兼容系统。

5.2 操作命令字

在对 8259A 设置了初始化命令字后，芯片就已准备好接收设备的中断请求信号了。但在 8259A 工作期间，我们也可以利用操作命令字 OCW1~OCW3 来监测 8259A 的工作状况，或者随时改变初始化时设定的 8259A 的工作方式。

需要说明的是，与初始化命令字ICW1～ICW4需要按规定的顺序进行设置不同，操作命令字OCW1～OCW3的设置没有规定其先后顺序，使用时可根据需要灵活选择不同的操作命令字写入到8259A中

** OCW1 **

* OCW1 用于对 8259A 中中断屏蔽寄存器 IMR 进行读/写操作。地址线A0需为1。

![](https://img-blog.csdn.net/20180304214235462)

若 Mi=1，则屏蔽对应中断请求级IRi；若 Mi=0，则允许IRi. 另外，屏蔽高优先级并不会影响其他低优先级的中断请求。

在 Linux-0.11 内核初始化过程中，代码在设置好相关的设备驱动程序后就会利用该操作命令字来修改相关中断请求屏蔽位。例如在软盘驱动程序初始化结束时，为了允许软驱设备发出中断请求，就会读端口0x21以取得 8259A 芯片的当前屏蔽字，然后与上~0x40（取反） 来复位M6\(软盘控制器连接到了中断请求IR6上），最后再写回中断屏蔽寄存器中。

** OCW2 \(需要A0 = 0\)**

![](https://img-blog.csdn.net/20180304214422347)

Linux-0.11 内核仅使用该操作命令字在中断处理过程结束之前向 8259A 发送结束中断（EOI）命令。所使用的OCW2 值为 0x20，表示固定优先级、一般EOI。

** OCW3 **

OCW3用于设置或清除特殊屏蔽方式和读取寄存器状态（IRR 和 ISR）。当 D4D3=01，且地址线 A0=0 时，表示对OCW3进行编程。在Linux-0.11 内核中并没有用到该操作命令字。

![](https://img-blog.csdn.net/20180304214640195?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzQ5MDg5Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



**命令字端口地址速查表：**

| 命令字 | A0 | 主片端口地址 | 从片端口地址 | 备注 |
| :---: | :---: | :---: | :---: | :---: |
| ICW1 | 0 | 0X20 | 0XA0 | D4 = 1 |
| ICW2 | 1 | 0X21 | 0XA1 |  |
| ICW3 | 1 | 0X21 | 0XA1 |  |
| ICW4 | 1 | 0X21 | 0XA1 |  |
| OCW1 | 1 | 0X21 | 0XA1 |  |
| OCW2 | 0 | 0X20 | 0XA0 | D4:D3 = 00 |
| OCW3 | 0 | 0X20 | 0XA0 | D4:D3 = 01 |



原文：[https://blog.csdn.net/u013007900/article/details/50408903](https://blog.csdn.net/u013007900/article/details/50408903)  
原文：[https://blog.csdn.net/longintchar/article/details/79439466](https://blog.csdn.net/longintchar/article/details/79439466)

[< prev](chapter7.md) [next >](chapter9.md)
