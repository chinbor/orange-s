## orange's 第九章学习笔记

#### **chapter9/a**

* 主要分析的是**task\_**_**hd**函数和**task\_fs**函数，尤其注意的是**task**_**\_**_**hd**函数，这个函数里面有硬盘的初始化工作，还有消息的判断，还有就是**hd\_identify**函数的调用，这个函数里面的**interrupt\_wait**函数和**hd\_cmd\_out**函数尤其需要注意，特别是**interrupt\_wait函数，他这里还分了两种情况的，具体见书中333页，对于书中的程序用你觉得最理想的状态去考虑执行过程，因为书中的程序大多都是作者调试好的，也就是对于程序执行的时间的把控，作者是通过大量调试得出来的。所以不要太较真的去考虑，虽然多思考是对的，但是，我们是学习原理，学习机制。还有一点，对于书中的device register用的是0XA0，所以不是LBA模式，是CHS模式来定位扇区。**_



