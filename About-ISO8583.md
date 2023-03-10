# ISO8583
## 定义
> 8583报文大部分情况下用在POS终端与后台收单系统的数据交换, 一般情况下(请注意这里的用词)  
> 一段完整的交易报文由以下几个部分组成:  
> 长度【总长度，即"报文头"+"数据"部分的长度】  +　报文头　+  数据(8583报文内容)


- 报文头[TPDU]为5个字节构成如下：    

> 类型ID[1个字节] + 源设备地址[2个字节] + 目的设备地址[2个字节]


- 数据 报文内容由三部分按照以下顺序组成：  

> 消息类型表示 +  位图 + 数据元素


- 说明  


> 不同的应用领域, 上面几个部分大长度和格式上有一些差别, 有一些应用甚至前面的"长度"部分.所以如果等一下你看到下面一些数据的长度或格式跟你的不一样  
> 先说说"长度"部分, 一般两个字节, 表示报文的总长度(即"报文头"+"数据"部分的长度), 在两个字节在报文里的表示方法因系统与终端的协议不同而不同.   
> 
> 一般有两种:  
> 1 BCD方法, 比如报文的总长度是134字节, 那么在实际的报文中, 这两个字节为"01h,34h"(注意16进制)  
> 2 实际的计算的长度（十进制转换为16进制）, 比如还是134长度的字节, 实际的报文中，两个字节为"00h, 86h"(注意也是16进制，00h*256+86h = 134d).

### 1:消息类型标识
> 每个消息类型标识时报文内容得头.
> 占用四个字节,在打包时用压缩得BCD码.向外传输


### 2:位图
> 位图是8583包的灵魂，它是打包解包确定字段域的关键，而了解每个字段域的属性则是填写数据的基础，在此中用8个字节(1个字节为8位,则为8*8=64)
> 设置其后64位的域.8583通过检查位图中的1的位置可以确定其后的数据的域,然后根据域中规定的长度来取数据,这样解包就很方便了,打包则为相反.
> 
> 首先检查域中有无数据,如有,则在相应的位图中的位置放置1,来标识此域中有数据打包与解包


<pre>
0      1     0    1    0

第一个1表示：第一个域中有数据
第二个1表示：第三个域中有数据
</pre>



### 3:数据元素
### 3.1:数据元素的压缩
-   如果数据元素为数字,则才用压缩的BCD码来压缩数据,以便于在网络中传输
-   如果为字母用ASCII 码表示的字节的数

###3.2:数据元素的定长与不定长
-   如为定长,则用压缩的BCD码来处理,
-   如为不定长,则才用字节的长度值+ 字节的数据(其中长度值采用压缩的BCD码,而字节的数据如为字母则才用ASCII码来表示,如为数字则也用压缩的BCD码)


------------------------------------------------------------------------------------
## 打包和解包
### 打包：
1. 把数据放入数据结构中
1. 对数据进行检验并对位图进行处理
1. 对数据压缩并送出去

### 解包：
1. 获得数据
1. 对数据按8583格式进行解包


ISO8583包（简称8583包）是一个国际标准的包格式，最多由128个字段域组成，每个域都有统一的规定，并有定长与变长之分。
8583包前面一段为位图，用来确定包的字段域组成情况。其中位图是8583包的灵魂，它是打包解包确定字段域的关键。
 而了解每个字段域的属性则是填写数据的基础。
 
## 位图说明：
位置：在8583包的第1 位
格式：定长 
类型：B16（二进制16位，16*8=128bit） 
描述： 
如将位图的第一位设为'1'，表示使用扩展位图（128个域），否则表示只使用基本位图（64个域）。 
如使用某数据域，应在位图中将相应的位设位'1'，如使用41域，需将位图的41位设为'1'。 
选用条件：如使用65到128域，需设位图域第一位为'1' 

## 域的定义： 
```c++
typedef struct ISO8583 
{ 
int bit_flag; /*域数据类型0 -- string, 1 -- int, 2 -- binary*/ 
char *data_name; /*域名*/ 
int length; /*数据域长度*/ 
int length_in_byte;/*实际长度（如果是变长）*/ 
int variable_flag; /*是否变长标志0：否 2：2位变长, 3：3位变长*/ 
int datatyp; /*0 -- string, 1 -- int, 2 -- binary*/ 
char *data; /*存放具体值*/ 
int attribute; /*保留*/ 
}
```

> ISO 8583金融交易信息数据包由信息类型（Message_type）、64或128bits的位图（Bit_map）和
> 按位图描述的顺序排列的数据元序列（ELEMENTS）等三段组成
 
### 解释
<pre>
信息类型是一个4位数字的数字型字段，用来描述每一个交易信息的类别和功能，其中前两位数字标明信息类别，如授权信息、金融交易信息、管理信息，等等。
在一个金融系统中，信息类型的定义应该是唯一的，无二义性的。网间交易具有不同的信息类型定义时, 应在交换报文的发送前和接收后完成类型转换处理。 

位图由64或128位二进制比特位构成，每一位用1或0来表示与该比特位相对应的数据元存在或不存在。位图的第一位为1时，表示64位的位图后紧接着
一个扩展的64位位图。本AIMS系统EDC规格未使用扩展位图。 

数据元指交易中一个数据项的实际内容，数据元在数据包中是否存在及存放位置由位图中的相应比特位确定。一些数据元有固定的长度，一些数据元为变长。
具有可变长度类型的数据元应在实际数据之前附加标明长度的前缀字节。



以64个域的报文来举例，域是什么我也说不清楚，你可以把它想象为医院放药的抽屉，一个抽屉预先定义好要放什么东西，比如伟哥，或者感冒冲剂，
一般情况下定义放伟哥的抽屉最好永远放伟哥[类型]，不要放别的东西，当然你也可以放板蓝根，但这样的话容易出错，也不太规范。

数量是这么规定的，有三种情况：
首先是定量，也就是说定义好这个抽屉放30瓶伟哥，就放30瓶一瓶也不能多，一瓶也不能少。
其次是LLVAR，也就是说用1位字节定义数量，比如0x12表示里头放12瓶，当然你也可以理解为16+2=18瓶。但要是0x12表示12，那0x13就等于13，
不要0x12=12 ，0x13=19。
最后是LLLVAR,是2位字节表示数量，比如 0x01,0x04 = 104

域也就是这样的，一共有64个域，每个域预先定义了内容和长度

有一个叫做BITMAP的，也就是位图，定义了一个数据包里包含了几个域。
</pre>