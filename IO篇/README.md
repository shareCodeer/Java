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
    | read  | 否     | byte[] b | int | 从输入流读取一定数量的字节，然后存储这些字节到缓冲数组b中，实际读取的字节数作为整数返回，方法也会阻塞直到输入数据可用，文件到达末尾，或一个异常被抛出， 第一个字节读取被存进b[0],下一个字节被存入b[1]，如此继续，最大读到数组b的长度 | 
    | read  | 否     | byte[] b, int off, int len |  int  | 尝试从输入流读取len个字节，并存入缓冲数组b中，返回值代表实际读取到的字节数（当len为0时返回0），若到达文件末尾，则返回-1，否则至少有一个字节被读取并存入数组b中，其中off代表字节从数组的什么位置开始存入，内部实现原理其实就是简单的for循环一个一个字节的读取 |
    | close | 否     | - | void | 用于关闭输入流，释放关联到这个输入流的任何的系统资源 | 
    | available | 否 | - | int | 预估从输入流中可以读取到多少数量的字节（不包含阻塞状态的），当到达文件末尾时，返回0 |
    | markSupported | 否 | -  | boolean | 校验输入流是否支持mark和reset方法，默认InputStream 是返回false的，表示不支持标记 | 
    | mark | 否 | int readlimit | void | 标记该输入流中当前读取的字节位置，并指定可重复读取的字节数量 | 
    | reset | 否 | - | void | 重新定位到上一次mark标记的位置开始读取，结合mark方法可以实现重复从输入流中读取字节 | 

---
    
+ 实现类

##### FileInputStream

> 从主机环境的文件系统中的文件读取字节，其用来读取原始字节流的，例如图像数据，若想读取字符流，请考虑使用FileReader

+ 属性

| 属性名 | 类型 | 描述 | 
| ------ | ---- | ---- |
| fd     | final FileDescriptor | 文件描述符，用于打开文件的句柄 |
| path   | final String      | 引用文件的路径，如果使用文件描述符创建的输入流，则该字段为null |
| channel | FileChannel | - | 
| closeLock | final Object | 直接new了一个Object对象，表示关闭锁 |
| closed | volatile boolean | 表示是否关闭状态，用volatile修饰，保证多线程可见性 |

+ 构造函数
        
        // 通过打开一个实际文件的连接，创建一个FileInputStream对象，实际的文件名就是文件系统中的path值
        // 新new出来的FileDescriptor文件句柄对象就代表这个文件连接
        public FileInputStream(String name) throws FileNotFoundException {
            this(name != null ? new File(name) : null);
        }
        
        
        // 重载的构造函数
        /**
        * 1：checkRead():
        * 如果有一个安全管理器，会去调用安全管理器的checkRead方法，参数就是实际文件的路径,主要是校验其read权限，即是否有读取的权限
        * 2：isInvalid():
        * 如果该文件名不存在，或者是一个目录而不是常规的文件，又或者一些其他的原因不能被打开读取，就会抛FileNotFoundException
        * 
        */
        public FileInputStream(File file) throws FileNotFoundException {
            String name = (file != null ? file.getPath() : null);
            SecurityManager security = System.getSecurityManager();
            if (security != null) {
                security.checkRead(name);
            }
            if (name == null) {
                throw new NullPointerException();
            }
            if (file.isInvalid()) {
                throw new FileNotFoundException("Invalid file path");
            }
            fd = new FileDescriptor();
            fd.attach(this);
            path = name;
            open(name);
        }

 
+ 文件描述符[摘自木杉的博客](http://imushan.com/2018/05/29/java/language/JDK%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB-FileDescriptor/)


> 操作系统使用文件描述符来指代一个打开的文件，对文件的读写等操作，都需要使用文件描述符作为参数，Java语言虽然使用了抽象程度更高的
流来操作文件，但是底层仍然需要使用文件描述符与操作系统交互，在Java世界中与文件描述符对应的类就是FileDescriptor，这也就是为什么FileInputStream,FileOutStream类都有一个
FileDescriptor的成员变量

操作系统中的文件描述符本质上是一个非负整数，其中0， 1， 2已经固定为标准输入，标准输出，标准错误输出，所以程序中接下来打开的
文件会使用当前进程中最小可用的文件描述符号码，比如3

文件描述符本身就是一个整数，所以FileDescriptor的核心就是保存这个整数

* FileDescriptor类

        public final class FileDescriptor {
            
            // 成员变量
            private int fd;
            
            // 无参构造，可见fd并不能通过new的时候指定
            public /**/ FileDescriptor() {
                fd = -1;
                handle = -1;
            }
        }

既然无法通过Java代码指定fd的值，那么fd是怎么赋值的呢？只能通过FileInputStream 寻找答案了

在FileInputStream的构造函数中，我们看到new了一个FileDescriptor对象，
并调用了fd的attach方法关联FileInputStream实例与FileDescriptor实例，这是为了日后关闭文件描述符做准备，可是在这里还是没有对fd作赋值操作啊

实际上Java层面无法对FileDescriptor实例的fd属性赋值，真正的赋值逻辑是在FileInputStream#open0这个native方法中

## 总结一下

+ 何为文件描述符？

答：操作系统使用文件描述符来指代一个打开的文件，对文件的读写等操作，都需要使用文件描述符作为参数， 本质上就是一个正整数

+ Java流操作文件读取为何也需要文件描述符？

答：Java语言虽然使用了抽象程度更高的流来操作文件，但是底层仍然需要使用文件描述符与操作系统交互，在Java世界中与文件描述符对应的类就是FileDescriptor

+ FileDescriptor文件描述符的成员变量何时初始化？

答：通过源码知道new 一个FileDescriptor 时是无法赋值的，其成员变量真正赋值发生在FileInputStream的open0本地方法中，即在JNI代码中使用open系统调用打开文件，得到文件描述符在JNI代码中设置到FileDescriptor的fd成员变量上



    

