# class字节码解析

## Class文件分析

class文件分析最简单的文件开始了解最基本的结构。

分析前需要准备以下数据：

1. 创建最java文件：

   ```java
   public Class A{}
   ```

2. 编译java文件，

   ```java
   javac A.java
   ```

   得到A.class文件

   通过十六进制文件查看器打开文件`A.class`,下面是以Sublime3打开，

   ```
   cafe babe 0000 003c 000d 0a00 0200 0307
   0004 0c00 0500 0601 0010 6a61 7661 2f6c
   616e 672f 4f62 6a65 6374 0100 063c 696e
   6974 3e01 0003 2829 5607 0008 0100 0141
   0100 0443 6f64 6501 000f 4c69 6e65 4e75
   6d62 6572 5461 626c 6501 000a 536f 7572
   6365 4669 6c65 0100 0641 2e6a 6176 6100
   2100 0700 0200 0000 0000 0100 0100 0500
   0600 0100 0900 0000 1d00 0100 0100 0000
   052a b700 01b1 0000 0001 000a 0000 0006
   0001 0000 0001 0001 000b 0000 0002 000c
   ```

   不同工具打开样式可能有所区分，空格在文件中是不存在的，只是不同工具表现形式不一样而已。

3. 反编译class文件

   ```java
   javap -verbose A.class
   ```

   结果文件

   ```java
   //
   // Source code recreated from a .class file by IntelliJ IDEA
   // (powered by FernFlower decompiler)
   //
   
   public class A {
       public A() {
       }
   }
   
     Last modified 2022年2月9日; size 176 bytes
     SHA-256 checksum ca56d664de3116a0d8e3c9b108caf13100eefef6022c4a73b18658a526df271c
     Compiled from "A.java"
   public class A
     minor version: 0
     major version: 60
     flags: (0x0021) ACC_PUBLIC, ACC_SUPER
     this_class: #7                          // A
     super_class: #2                         // java/lang/Object
     interfaces: 0, fields: 0, methods: 1, attributes: 1
   Constant pool:
      #1 = Methodref          #2.#3          // java/lang/Object."<init>":()V
      #2 = Class              #4             // java/lang/Object
      #3 = NameAndType        #5:#6          // "<init>":()V
      #4 = Utf8               java/lang/Object
      #5 = Utf8               <init>
      #6 = Utf8               ()V
      #7 = Class              #8             // A
      #8 = Utf8               A
      #9 = Utf8               Code
     #10 = Utf8               LineNumberTable
     #11 = Utf8               SourceFile
     #12 = Utf8               A.java
   {
     public A();
       descriptor: ()V
       flags: (0x0001) ACC_PUBLIC
       Code:
         stack=1, locals=1, args_size=1
            0: aload_0
            1: invokespecial #1                  // Method java/lang/Object."<init>":()V
            4: return
         LineNumberTable:
           line 1: 0
   }
   SourceFile: "A.java"
   ```

   

### 3. 十六进制class数据

```
cafe babe 每个Class文件的头4个字节被称为魔数（Magic Number），它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。
					GIF或者JPEG等文件在文件头中都存有魔数。使用魔数而不是扩展名来进行识别主要是基于安全考虑，因为文件扩展名可以随意改动
紧接着魔数的4个字节存储的是Class文件的版本号：
0000 			第5和第6个字节是次版本号，JDK 12之前次版本号一直没用，全部为零，JDK 12及以后版本为了标识复杂的新功能特性，需要以“公测”						的形式放出，把次版本号标识为65535，以便Java虚拟机在加载类文件时能够区分出来。
003c 			第7和第8字节为主版本号：60，版本号是根据编辑jdk版本生成的，见版本号列表
紧接着主、次版本号之后的是常量池，常量也可以比喻为Class文件里的资源仓库，它是Class文件结构中与其它项目关联最多的数据，通常也是占用Class文件空间最大的数据项目之一
======================常量池：constant_pool_count=======================================
常量池第一项为u2类型数据，代表常量池容量计数值，第0项常量为空，如果后面某些指向常量池的索引值的数据在特殊情况下表达“不引用任何一个常量池项目”的含义时，可以把索引值设置为0来表示，所以常量数为常量池数减1。
000d						常量数：13-1 = 12
tag含义见：常量池中的17种数据类型的结构总表
#1		0a				tag=CONSTANT_Methodref_info
			0002			index=#2	
			0003			index=#3	
#2		07				tag=CONSTANT_Class_info
			0004		 	index=4
#3		0c				tag=CONSTANT_NameAndType
			0005			index=#5
			0006			index=#6
#4		01 				tag=CONSTANT_Utf8_info
			0010 			length=16
			6a	61	76	61	2f	6c	61	6e	67	2f	4f	62	6a	65	63	74 
		 [106,97,	118,97,	47,	108,97,	110,103,47,	79,	98,	106,101,99,	116]
		 [j, 	a, 	v,  a,	/,	l,	a,	n,	g,	/,	O,	b,	j,	e,	c,	t]=java/lang/Object
#5		01				tag=CONSTANT_Utf8_info
			0006			length=6
			3c	69	6e	69	74	3e
		 [60,	105,110,105,116,62]
		 [<,	i,	n,	i,	t,	>]=<init>
#6		01				tag=CONSTANT_Utf8_info
			0003 			length=3
			28	29	56
		 [40, 41,	86]
		 [(,	),	V]=()V
#7		07				tag=CONSTANT_Class_info
			0008			index=8
#8		01				tag=CONSTANT_Utf8_info
			0001			length=1
			41				
		 [65]
		 [A]=A
#9		01				tag=CONSTANT_Utf8_info
			0004			length=4
			43	6f	64	65
		 [67,	111,100,101]
		 [C,	o,	d,	e]=Code
#10		01				tag=CONSTANT_Utf8_info
			000f 			length=15
			4c	69	6e	65	4e	75	6d	62	65	72	54	61	62	6c	65	
		 [76,	105,110,101,78,	117,109,98,	101,114,84,	97,	98,	108,101]
		 [L,	i,	n,	e,	N,	u,	m,	b,	e,	r,	T,	a,	b,	l,	e]=LineNumberTable
#11		01				tag=CONSTANT_Utf8_info
			000a			length=10
      53	6f	75	72	63	65	46	69	6c	65
     [83,	111,117,114,99,	101,70,	105,108,101]
     [S,	o,	u,	r,	c,	e,	F,	i,	l,	e]=SourceFile
#12   01				tag=CONSTANT_Utf8_info
			0006			length=6			
			41	2e	6a	61	76	61
		 [65,	46,	106,97,	118,97]
		 [A,	.,	j,	a,	v,	a]=A.java
=========acces_flags=====================================
见访问标志表
0021	0x0020 | 0x0001 = 0x0021	(ACC_PUBLIC | ACC_SUPER)
====================this_class===========================
00 07		#7
====================super_class===========================
00 02		#2
===============interface_count=======================
0000		接口计数器：0
==============fields_count==============
0000
==============method_count========================
00 01
==============method_info===========================
0001			access_flags=ACC_PUBLIC
0005			name_index=#5  <init>
0006			descriptor_index=6 ()V
0001			attributes_count=1
===============attribute_info=======================================
对于每一个属性，它的名称都要从常量池中引用的一个constant_Utf8_info类型的常量来表示，只需要通过一个u4的长度属性去说明属性值所占用的位数即可，见属性表结构表
0009			Code
0000 001d length= 29
0001			max_stack=1
0001			max_locals=1
0000 0005 code_length=5
2a	b7	00	01	b1 
0000 			exception_table_length=0
0001 			attributes_count=1
=============LineNumberTable===================
LineNumberTable属性用于描述Java源码行号与字节码行号之间的对应关系。它并不是运行时必需的属性，默认会生成到Class文件中，可以在Javac中使用-g:none或-g:lines选项来取消或显示，对程序产生的影响主要是抛出异常时，堆栈中将不会显示出错的行号，并且在调试程序的时候，也无法按照源码行来设置断点
000a 			#10
0000 0006	length=6
0001			line_number_table_length=1
=============LineNumberInfo===================
0000			start_pc=0
0001			line_number=1
===============LocalVariableTable========================
LocalVariableTable属性用于描述栈帧中局部变量表的变量与Java源码中定义的变量之间的关系，它不是运行时必需的属性，但默认会生成到class文件中，可以在Javac中使用-g:none或-g: vars选项来取消或要求生成这项信息。如果没有生成这项属性，最大的影响就是当别人引用这个方法时，所有的参数名称都将会丢失，譬如IDE将会使用诸如：arg0,arg1之类的占位符代替原有的参数名。
0001
===============LocalVariableTypeTable========================
此属性是LocalVariableTable的“姐妹属性”，两个属性结构相似，仅仅是把记录的字段描述符的descriptor_index替换成了字段特征签名。对于非泛型来说，描述符和特征签名能描述的信息是能吻合一致的，但是泛型引入之后，由于描述符中泛型的参数化类型被擦除掉，描述符就不能准确描述泛型类型了。因此出现了此类型属性，使用字段的特征签名来完成泛型的描述。
===============sourceFile===================
000b 			attribute_name_index=#11
0000 0002 length=0 
000c			sourFile_index=#12
				


```





### 以下为不同版本生成的文件区分

```cafe babe 0000 0033 0010 0a00 0300 0d07
cafe babe 0000 0032 0010 0a00 0300 0d07
000e 0700 0f01 0006 3c69 6e69 743e 0100
0328 2956 0100 0443 6f64 6501 000f 4c69
6e65 4e75 6d62 6572 5461 626c 6501 0012
4c6f 6361 6c56 6172 6961 626c 6554 6162
6c65 0100 0474 6869 7301 0003 4c41 3b01
000a 536f 7572 6365 4669 6c65 0100 0641
2e6a 6176 610c 0004 0005 0100 0141 0100
106a 6176 612f 6c61 6e67 2f4f 626a 6563
7400 2100 0200 0300 0000 0000 0100 0100
0400 0500 0100 0600 0000 2f00 0100 0100
0000 052a b700 01b1 0000 0002 0007 0000
0006 0001 0000 0001 0008 0000 000c 0001
0000 0005 0009 000a 0000 0001 000b 0000
0002 000c 


cafe babe 0000 0033 0010 0a00 0300 0d07
000e 0700 0f01 0006 3c69 6e69 743e 0100
0328 2956 0100 0443 6f64 6501 000f 4c69
6e65 4e75 6d62 6572 5461 626c 6501 0012
4c6f 6361 6c56 6172 6961 626c 6554 6162
6c65 0100 0474 6869 7301 0003 4c41 3b01
000a 536f 7572 6365 4669 6c65 0100 0641
2e6a 6176 610c 0004 0005 0100 0141 0100
106a 6176 612f 6c61 6e67 2f4f 626a 6563
7400 2100 0200 0300 0000 0000 0100 0100
0400 0500 0100 0600 0000 2f00 0100 0100
0000 052a b700 01b1 0000 0002 0007 0000
0006 0001 0000 0001 0008 0000 000c 0001
0000 0005 0009 000a 0000 0001 000b 0000
0002 000c 


cafe babe 0000 0034 0010 0a00 0300 0d07
000e 0700 0f01 0006 3c69 6e69 743e 0100
0328 2956 0100 0443 6f64 6501 000f 4c69
6e65 4e75 6d62 6572 5461 626c 6501 0012
4c6f 6361 6c56 6172 6961 626c 6554 6162
6c65 0100 0474 6869 7301 0003 4c41 3b01
000a 536f 7572 6365 4669 6c65 0100 0641
2e6a 6176 610c 0004 0005 0100 0141 0100
106a 6176 612f 6c61 6e67 2f4f 626a 6563
7400 2100 0200 0300 0000 0000 0100 0100
0400 0500 0100 0600 0000 2f00 0100 0100
0000 052a b700 01b1 0000 0002 0007 0000
0006 0001 0000 0001 0008 0000 000c 0001
0000 0005 0009 000a 0000 0001 000b 0000
0002 000c 


cafe babe 0000 0039 0010 0a00 0200 0307
0004 0c00 0500 0601 0010 6a61 7661 2f6c
616e 672f 4f62 6a65 6374 0100 063c 696e
6974 3e01 0003 2829 5607 0008 0100 0141
0100 0443 6f64 6501 000f 4c69 6e65 4e75
6d62 6572 5461 626c 6501 0012 4c6f 6361
6c56 6172 6961 626c 6554 6162 6c65 0100
0474 6869 7301 0003 4c41 3b01 000a 536f
7572 6365 4669 6c65 0100 0641 2e6a 6176
6100 2100 0700 0200 0000 0000 0100 0100
0500 0600 0100 0900 0000 2f00 0100 0100
0000 052a b700 01b1 0000 0002 000a 0000
0006 0001 0000 0001 000b 0000 000c 0001
0000 0005 000c 000d 0000 0001 000e 0000
0002 000f 


cafe babe 0000 003c 000d 0a00 0200 0307
0004 0c00 0500 0601 0010 6a61 7661 2f6c
616e 672f 4f62 6a65 6374 0100 063c 696e
6974 3e01 0003 2829 5607 0008 0100 0141
0100 0443 6f64 6501 000f 4c69 6e65 4e75
6d62 6572 5461 626c 6501 000a 536f 7572
6365 4669 6c65 0100 0641 2e6a 6176 6100
2100 0700 0200 0000 0000 0100 0100 0500
0600 0100 0900 0000 1d00 0100 0100 0000
052a b700 01b1 0000 0001 000a 0000 0006
0001 0000 0001 0001 000b 0000 0002 000c