## TCP报文段的首部格式

TCP虽然是面向字节流的，但TCP传送的数据单元却是**报文段**。一个TCP报文段分为**首部和数据**两部分，而TCP的全部功能都体现在它在首部中各字段的作用。下面讨论TCP报文段的首部格式。

![](https://img2020.cnblogs.com/blog/2361214/202109/2361214-20210902222547591-257968641.png)

TCP报文首部前20字节是固定的（如图5-14），后面有4n字节是根据需要而增加的选项。（TCP协议首部的最小长度为20字节）

首部固定部分各字段的意义如下：

- 源端口和目的端口 （各占2字节）：分别写入源端口号和目的端口号。**TCP的分用功能也是通过端口来实现的**。

- 序号（4字节）：序号的范围是[$0$，$2^{32}-1$],共2^32个序号，在一个TCP连接中传送的字节流的**每一个字节按照顺序编号**。整个要传送的字节流的起始序号必须在连接建立时设置。首部中的序号字段值则指的是本报文段所发送的数据的第一个字节的序号。例如，一个报文段的序号值为301，而携带的数据共有100字节。显然，下一个报文段（如果还有的话）的数据序号应当从401开始，即下一个报文段的序号字段值为401，这个字段的名称也叫做**报文段序号**。

- 确认号（4字节）：是**期望收到对方的下一个报文段的第一个数据字节的序号**，例如：B正确的接收到了A发送过来的一个报文段，其序列号字段为501，而数据长度是200字节（序号为501-700），这表明B正确的接收到了A发送的到序号700为止的数据。因此，B希望收到A下一个数据序号为701的数据，于是B在发送给A的确认报文中把确认号设置为701。总之就是要记住：**确认号=N：到序号N-1为止的所有数据都已正常的收到了**。

- 数据偏移（4位）：它指出的TCP报文段的数据部分的其实部分距离TCP报文段的起始位置，其实就是指出了TCP报文段首部的长度，（TCP报文首部最大的长度为60字节）。

- 保留位（6位）：保留为今后使用，目前是0。

- 下面是6个控制位（6位）
  
  - 紧急URG（URGent）：当URG=1，表示紧急指针字段有效，它告诉系统此报文段中有紧急数据，应该尽快送达（相当于高优先级的数据），不要按照原来的排队顺序来传送，发送方TCP就要把这个紧急报文段插入到本报文段数据的最前面而紧急数据后面的数据仍然是普通数据。这里一般要配合首部中紧急指针字段（urgent pointer）使用。
  
  - 确认ACK（ACKnowledment）仅当ACK=1时候确认号字段才有效，当ACK=0时候，确认号无效。**TCP规定，在连接建立后，所有传送的报文段都必须把ACK设置为1**
  
  - 推送PSH（PuSH）当两个应用进程进行交互式通信的时候，有时在一端的应用进程希望在键入一个命令后能够立即接受到对方的响应。在这种情况下,TCP就可以使用推送操作。当发送方把PSH设置为1，并立即创建一个报文段发送出去，接收方TCP收到PSH=1的报文的时候，会把这个报文尽快的交付给应用进程。而并不是放到缓存中。（推送操作很少使用）
  
  - 复位RST（ReSet）：当RST=1时候，表明TCP连接中出现了严重的差错（例如由于主机崩溃或者其他原因），必须要释放连接，然后重新建立运输连接，RST=1还用来拒绝一个非法的报文段或者拒绝打开一个链接。RST也称之为重建位或者重置位。
  
  - 同步SYN(SYNchronization)：在建立一个用来同步序号，当SYN=1而ACK=0,表明这是一个连接请求报文段，对方若同意建立连接，则应该在相应的报文段中使得SYN=1和ACK=1。
  
  - 终止FIN（FINis）：用来释放一个连接，当FIN=1时，表示此报文段的发送方发送的数据已经发送完毕，并要求释放运输层连接。

- 窗口（2字节）：窗口值为[$0$，$2^{16}-1$]，窗口指的是发送本报文的一方的**接收窗口**。窗口值告诉对方：从本报文段的确认号开始，接收方目前只能接受对方的数据报文发送量。设置这个窗口值的原因是接收方的数据接收缓存是有限的。例如：发送了一个报文段，其确认号为701，窗口字段是1000。这就是告诉对方：“从701号算起，我（即发送此报文段的一方）的接收缓存空间还可以接受1000字节的数据（字节序号为701~1700），你在给我发送数据的时候，必须考虑到这一点。”总之：**窗口字段明确指出了现在允许对方发动的数据量。窗口值经常在动态发生变化。**

- 检验和（2字节）：检验和字段检验的范围包括首部和数据两部分。**和UDP一样，在计算检验和时，要在TCP报文段的前面加上12字节的伪首部，伪首部的格式和UDP伪首部的格式一样。但应该要把伪首部的第四个字段值改为6（TCP的协议号为6），第5字段中UDP长度改为TCP长度。**

- 紧急指针（2字节）：紧急指针仅仅在URG=1的时候才有意义，它指出本报文段的紧急数据的字节数（紧急数据结束后就是普通数据），窗口期为0也是可以发送紧急数据。

- 选项（可变，最长为40字节）
