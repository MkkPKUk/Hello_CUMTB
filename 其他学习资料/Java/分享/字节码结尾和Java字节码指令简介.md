# 勇闯JVM:Class文件格式结尾和Java字节码指令简介(二)

上次聊到属性表，当时确实有点晚三点多了吧，然后感觉后面东西又有点多就没有写。现在来补坑了，在把Java字节码指令顺便回顾一下。

接上次内容

### 属性表集合

属性表(attribute_info)在前面的讲解之中已经出现过数次，Class文件、字段表、方法表都可以 携带自己的属性表集合，以描述某些场景专有的信息。与Class文件中其他的数据项目要求严格的顺序、长度和内容不同，属性表集合的限制稍微宽松一 些，不再要求各个属性表具有严格顺序，并且《Java虚拟机规范》允许只要不与已有属性名重复，任 何人实现的编译器都可以向属性表中写入自己定义的属性信息，Java虚拟机运行时会忽略掉它不认识 的属性。到如今。定义的属性已经有29种了。所以还是自行百度~~ 咋门把所有都讲完不太现实。就聊聊`code`属性

#### Code属性

字节码里面的每项基本都是由一个一个的表组成的，所以Code属性也不例外，也是表组成的。

Code属性表结构:

 ![image-20201031013755023](https://zouyishan.oss-cn-beijing.aliyuncs.com/images/20201031013756.png)

**attribute_name_index：**是一项指向CONSTANT_Utf8_info型常量的索引，这个常量固定为Code。代表了属性名称，和attribute_length搭配的是该属性的长度。占了4个字节。所以这两个一共占了6个字节



**max_stack:** 代表了操作数栈深度的最大值。表示操作数栈都不会超过这个深度。虚拟机运行的时候需要根据这个值来分配栈帧（Stack Frame）中的操作栈深度。



**max_locals:** 代表了局部变量所需要的空间。单位是变量槽，变量槽是虚拟机为局部变量分配的最小的内存单位。对于int 这些不超过32位的只需要一个变量槽就可以方法参数（包括实例方法中的隐藏参数“this”）、显式异常处 理程序的参数（Exception Handler Parameter，就是try-catch语句中catch块中所定义的异常）、方法体中 定义的局部变量都需要依赖局部变量表来存放。`并不是在方法中用了多少个局部变量，就把这 些局部变量所占变量槽数量之和作为max_locals的值，操作数栈和局部变量表直接决定一个该方法的栈 帧所耗费的内存，不必要的操作数栈深度和变量槽数量会造成内存的浪费。`。**Java虚拟机的做法是将局 部变量表中的变量槽进行重用，当代码执行超出一个局部变量的作用域时，这个局部变量所占的变量 槽可以被其他局部变量所使用，Javac编译器会根据变量的作用域来分配变量槽给各个变量使用，根据 同时生存的最大局部变量数量和类型计算出max_locals的大小。**



**code_length:**  这就是用来表示字节码的长度，code用于存储字节码指令的一些列字节流。就是一些u1的字节码嘛。`关于code_length虽然是一个u4类型的长度值，应该是最大有2的32次幂，但是<Java>虚拟机规范中明确限制了一个方法不允许操作65535条字节码指令，一般不乱搞基本没事。除开JSP`

 ![image-20201106000857461](https://zouyishan.oss-cn-beijing.aliyuncs.com/images/20201106000907.png)



在给看看对应的字节码，但是可能篇幅有限，还自行百度==

 ![image-20201106001109087](https://zouyishan.oss-cn-beijing.aliyuncs.com/images/20201106001122.png)



同时需要注意的是:每个方法的Args_size起点都是1，Locals都是1。因为我们提到过，Code属性包括"this"关键字。`Javac编译器编译的时候把对this关键字的访问转变为对一个普通方法参数的访问，然后在虚拟机调用实例方法时自动传入此参数而已。`,除非方法被声明为static



#### Try Catch异常

上文说到，code属性包括this，局部变量也包含异常的try catch。

 ![image-20201106001854634](https://zouyishan.oss-cn-beijing.aliyuncs.com/images/20201106001856.png)

如下看个具体例子：

 ![image-20201106001943962](https://zouyishan.oss-cn-beijing.aliyuncs.com/images/20201106001945.png)



### 其他属性

#### Exception

这个Exception属性就是与Code属性平级的一个属性。接下来的属性都没有Code属性那么重要，就给个结构的图片

 ![image-20201106002123206](https://zouyishan.oss-cn-beijing.aliyuncs.com/images/20201106002124.png)



#### LineNumberTable属性

LineNumberTable属性用于描述Java源码行号与字节码行号（字节码的偏移量）之间的对应关系。 `它并不是运行时必需的属性`，但默认会生成到Class文件之中，可以在Javac中使用-g：none或-g：lines 选项来取消或要求生成这项信息。`如果选择不生成LineNumberTable属性，对程序运行产生的最主要影 响就是当抛出异常时，堆栈中将不会显示出错的行号，并且在调试程序的时候，也无法按照源码行来 设置断点。`



#### LocalVariableTable及LocalVariableTypeTable属性

LocalVariableTable属性用于描述栈帧中局部变量表的变量与Java源码中定义的变量之间的关系，`它 也不是运行时必需的属性`，但默认会生成到Class文件之中，可以在Javac中使用-g：none或-g：vars选项 来取消或要求生成这项信息。如果没有生成这项属性，`最大的影响就是当其他人引用这个方法时，所 有的参数名称都将会丢失，譬如IDE将会使用诸如arg0、arg1之类的占位符代替原有的参数名，这对程 序运行没有影响`，但是会对代码编写带来较大不便，而且在调试期间无法根据参数名称从上下文中获 得参数值。



在JDK 5引入泛型之后，LocalVariableTable属性增加了一个“姐妹属性”—— LocalVariableTypeTable。这个新增的属性结构与LocalVariableTable非常相似，仅仅是把记录的字段描述 符的descriptor_index替换成了字段的特征签名（Signature）。对于非泛型类型来说，描述符和特征签名 能描述的信息是能吻合一致的，`但是泛型引入之后，由于描述符中泛型的参数化类型被擦除掉，描 述符就不能准确描述泛型类型了。因此出现了LocalVariableTypeTable属性，使用字段的特征签名来完 成泛型的描述`



#### SourceFile及SourceDebugExtension属性

略



#### ConstantValue属性

通知虚拟机自动为静态变量赋值，只有被static关键字修饰的变量才能用这个属性。



对于非static类型的变量都是在<init>()方法中进行的; 而如果用final和static来修饰一个变量，并且这个是基本类型或者java.lang.String的话，就会生成ConstantValue属性来初始化;如果这个变量没有被final修饰，或者不是基本类型或者java.lang.String的话，则将会在<clinit>()方法中进行初始化。

虽然final才更符合"ConstantValue"的语义。。。



#### InnerClasses属性

记录内部类与宿主类之间的关联



#### Deprecated及Synthetic属性

两个属性都是bool值

**deprecated:**  代表这个类或者字段，或者方法是不是不推荐使用了

**Synthetic: ** 代表是不是由Java源码直接产生。而或者是编译器自动生成的。 



#### StackMapTable属性

这个属性会在虚拟机类加载的字节码验证阶段被新类型检查验证器（Type Checker）使用，目的在于代替以前比较消耗性能的基于数据流分析的 类型推导验证器。



#### Signature属性

它是一个可选的定长属性，可以出现于类、字段 表和方法表结构的属性表中。主要记录泛型信息

运行期做反射时无法获得泛型信息。Signature属性就是为了弥补这个缺陷而
增设的，现在Java的反射API能够获取的泛型类型，最终的数据来源也是这个属性。



#### BootstrapMethods属性

BootstrapMethods属性在JDK 7时增加到Class文件规范之中，它是一个复杂的变长属性，位于类 文件的属性表中。这个属性用于保存invokedynamic指令引用的引导方法限定符。



#### MethodParameters属性

MethodParameters是在JDK 8时新加入到Class文件格式中的，它是一个用在方法表中的变长属性。 MethodParameters的作用是记录方法的各个形参名称和信息。



## 字节码指令简介

Java虚拟机的指令由一个字节长度的、代表着某种特定操作含义的数字(称为操作码), 以及跟随其后的零至多个代表此操作所需的参数(称为操作数)构成。所以是面向操作数的架构，不是寄存器的架构。

`由于限制了Java虚 拟机操作码的长度为一个字节（即0～255），这意味着指令集的操作码总数不能够超过256条。又由于 Class文件格式放弃了编译后代码的操作数长度对齐，这就意味着虚拟机在处理那些超过一个字节的数 据时，不得不在运行时从字节中重建出具体数据的结构，譬如要将一个16位长度的无符号整数使用两个无符号字节存储起来`

所以16位的数字就要用两个8位的想方设法的表示，难受。

这样缺点明显，但是优点也明显，省去了对齐，就相当于解放了多出来的空间。明显要小很多。小数据量高效传输。`用编译时间换大小了`
 ![image-20201106011130508](https://zouyishan.oss-cn-beijing.aliyuncs.com/images/20201106011132.png)



### 字节码与数据类型

在Java虚拟机的指令集中，大多数指令都包含其操作所对应的数据类型信息。举个例子，iload指 令用于从局部变量表中加载int型的数据到操作数栈中，而fload指令加载的则是float类型的数据。这两 条指令的操作在虚拟机内部可能会是由同一段代码来实现的，但在Class文件中它们必须拥有各自独立 的操作码。



因为Java虚拟机的操作码长度只有一字节，所以包含了数据类型的操作码就为指令集的设计带来 了很大的压力：`如果每一种与数据类型相关的指令都支持Java虚拟机所有运行时数据类型的话，那么 指令的数量恐怕就会超出一字节所能表示的数量范围了。因此，Java虚拟机的指令集对于特定的操作 只提供了有限的类型相关指令去支持它，换句话说，指令集将会被故意设计成非完全独立的。有一些单独的指令可以在必要的时候用来将一些不支持的类型转换为可被支持的类型。`





**下面列举了Java虚拟机所支持的与数据类型相关的字节码指令，通过使用数据类型列所代表的 特殊字符替换opcode列的指令模板中的T，就可以得到一个具体的字节码指令。如果在表中指令模板 与数据类型两列共同确定的格为空，则说明虚拟机不支持对这种数据类型执行这项操作。**

 ![image-20201106011712940](https://zouyishan.oss-cn-beijing.aliyuncs.com/images/20201106011715.png)



剩下的字节码太多了就不方便展示，用起来都是一个字节，上网搜搜作用就行。[还是放个网站](https://segmentfault.com/a/1190000008722128)~~

