# IO 篇

## 开场
说IO之前，先解释一下何为比特（位），字节，字符的概念

+ 比特：二进制中的一位数（1或0）称作1比特

+ 字节：计算机中规定1字节等于8比特，也即8位

+ 字符：结合不同的编码方式，如ASCII码，一个英文字母占一个字节空间，一个中文汉字占两个字节空间

在GBK编码中，英文字符占一个字节空间，中文占两个字节

而在UTF-8编码中，英文仍占一个字节空间，中文（含繁体）却占了三到四个字节

这里提一点，UTF-8编码是变长码，即保留了欧美国家Unicode通用性的同时，又避免了流量和空间的浪费，某些字符编码采用一个字节（英语），某些
采用两个，三个字节等

UTF-8在读取时，若是一个合法的单字节（8位比特）编码，就可以直接判定为一个字符，若不合法，再继续下一个字节，若这两个字节是合法的就判定为一个字符
否则继续读下去，直到读出合法的字符来

---

#### 代码示例：

    // UTF-8
    String name = "源代码";
    System.out.println(name.getBytes().length);// 9
    
    // change setting file global encoding to GBK
    System.out.println(name.getBytes().length);// 6

#### 小插曲

计算机存储信息的形式均已二进制比特的形式存储，Java I/O分别相应提供了字节形式操作类以及字符形式操作类

## Java IO

#### 基于字节操作的I/O接口

##### InputStream 抽象类

+ 类描述

> This abstract class is the superclass of all classes representing an input stream of bytes.
Applications that need to define a subclass of **InputStream** must always provide a method that returns the next byte of input.

代表所有字节输入流类的超类，若想继承该抽象类，必须提供一个方法返回下一个输入字节

+ 属性

        private static final int MAX_SKIP_BUFFER_SIZE = 2048;

+ 方法列表

    |方法名 | 是否抽象 | 入参 | 出参 | 描述 |
    | ----- | -------  | ---- | ---- | ---- | 
    | read  | 是     | 无   | int  |  从输入流数据中读取下一个字节，返回值用int类型（范围0-255）表示8位二进制，若因为已经到达流的末尾且没有字节可用，则返回-1，方法将会阻塞直到输入数据可用，或一个异常被抛出  | 
    | read  | 否     | byte[] b | int | 读取一定数量的字节从输入流，然后存储这些字节到缓冲数组b中，实际读取的字节数作为整数返回，方法也会阻塞直到输入数据可用，文件到达末尾，或一个异常被抛出， 第一个字节读取被存进b[0],下一个字节被存入b[1]，如此继续，最大读到数组b的长度 | 
    
