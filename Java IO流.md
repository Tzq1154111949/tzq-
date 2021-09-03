# java IO流学习总结

近期学习了Java的IO流，尝试着总结一下。

java.io 包下的IO流很多：

![img](https://img2018.cnblogs.com/blog/1490873/201810/1490873-20181009205826281-2118584242.png)

其中，以Stream结尾的为字节流，以Writer或者Reader结尾的为字符流。所有的输入流都是抽象类IuputStream（字节输入流）或者抽象类Reader（字符输入流）的子类，所有的输出流都是抽象类OutputStream(字节输出流)或者抽象类Writer(字符输出流)的子类。字符流能实现的功能字节流都能实现，反之不一定。如：图片，视频等二进制文件，只能使用字节流读写。

##  1、字符流FileReader和FileWriter

FileReader类

| 构造方法摘要                                                 |
| ------------------------------------------------------------ |
| `**FileReader**(File file)`      在给定从中读取数据的 `File` 的情况下创建一个新 `FileReader`。 |
| `**FileReader**(FileDescriptor fd)`      在给定从中读取数据的 `FileDescriptor` 的情况下创建一个新 `FileReader`。 |
| `**FileReader**(String fileName)`      在给定从中读取数据的文件名的情况下创建一个新 `FileReader`。 |

 FileWriter类

| 构造方法摘要                                                 |
| ------------------------------------------------------------ |
| `**FileWriter**(File file)`      根据给定的 File 对象构造一个 FileWriter 对象。 |
| `**FileWriter**(File file, boolean append)`      根据给定的 File 对象构造一个 FileWriter 对象。 |
| `**FileWriter**(FileDescriptor fd)`      构造与某个文件描述符相关联的 FileWriter 对象。 |
| `**FileWriter**(String fileName)`      根据给定的文件名构造一个 FileWriter 对象。 |
| `**FileWriter**(String fileName, boolean append)`      根据给定的文件名以及指示是否附加写入数据的 boolean 值来构造 FileWriter 对象。 |

使用FileReader和FileWriter类完成文本文件复制：

```java
 1 import java.io.FileReader;
 2 import java.io.FileWriter;
 3 import java.io.IOException;
 4 
 5 public class CopyFile {
 6     public static void main(String[] args) throws IOException {
 7         //创建输入流对象
 8         FileReader fr=new FileReader("C:\\Test\\copyfrom.txt");//文件不存在会抛出java.io.FileNotFoundException
 9         //创建输出流对象
10         FileWriter fw=new FileWriter("C:\\Test\\copyto.txt");
11         /*创建输出流做的工作：
12          *         1、调用系统资源创建了一个文件
13          *         2、创建输出流对象
14          *         3、把输出流对象指向文件        
15          * */
16         //文本文件复制，一次读一个字符
17         method1(fr, fw);
18         //文本文件复制，一次读一个字符数组
19         method2(fr, fw);
20         
21         fr.close();
22         fw.close();
23     }
24 
25     public static void method1(FileReader fr, FileWriter fw) throws IOException {
26         int ch;
27         while((ch=fr.read())!=-1) {//读数据
28             fw.write(ch);//写数据
29         }
30         fw.flush();
31     }
32 
33     public static void method2(FileReader fr, FileWriter fw) throws IOException {
34         char chs[]=new char[1024];
35         int len=0;
36         while((len=fr.read(chs))!=-1) {//读数据
37             fw.write(chs,0,len);//写数据
38         }
39         fw.flush();
40     }
41 }
```



## 2、字符缓冲流BufferedReader和BufferedWriter

字符缓冲流具备文本特有的表现形式，行操作

public class **BufferedReader** extends Reader

(1)从字符输入流中读取文本，缓冲各个字符，从而实现字符、数组和行的高效读取。

(2)可以指定缓冲区的大小，或者可使用默认的大小。大多数情况下，默认值就足够大了。

(3)通常，Reader 所作的每个读取请求都会导致对底层字符或字节流进行相应的读取请求。因此，建议用 BufferedReader 包装所有其 read() 操作可能开销很高的 Reader（如 FileReader 和 InputStreamReader）。例如，

```
 BufferedReader in
   = new BufferedReader(new FileReader("foo.in"));
```

(4)将缓冲指定文件的输入。如果没有缓冲，则每次调用 read() 或 **readLine()** 都会导致从文件中读取字节，并将其转换为字符后返回，而这是极其低效的。 

public class **BufferedWriter** extends Writer

(1)将文本写入字符输出流，缓冲各个字符，从而提供单个字符、数组和字符串的高效写入。

(2)可以指定缓冲区的大小，或者接受默认的大小。在大多数情况下，默认值就足够大了。

(3)该类提供了 **newLine() 方法**，它使用平台自己的行分隔符概念，此概念由**系统属性 `line.separator`** 定义。并非所有平台都使用新行符 ('\n') 来终止各行。因此调用此方法来终止每个输出行要优于直接写入新行符。

(4)通常 Writer 将其输出立即发送到底层字符或字节流。除非要求提示输出，否则建议用 BufferedWriter 包装所有其 write() 操作可能开销很高的 Writer（如 FileWriters 和 OutputStreamWriters）。例如，

```
 PrintWriter out
   = new PrintWriter(new BufferedWriter(new FileWriter("foo.out")));
```

(5)缓冲 PrintWriter 对文件的输出。如果没有缓冲，则每次调用 print() 方法会导致将字符转换为字节，然后立即写入到文件，而这是极其低效的。

 

 使用BufferedReader和BufferedWriter完成文件复制

```java
1 import java.io.BufferedReader;
 2 import java.io.BufferedWriter;
 3 import java.io.FileReader;
 4 import java.io.FileWriter;
 5 import java.io.IOException;
 6 
 7 public class CopyFile2 {
 8     public static void main(String[] args) throws IOException {
 9         //创建输入流对象
10         BufferedReader br=new BufferedReader(new FileReader("C:\\Test\\copyfrom.txt"));//文件不存在会抛出java.io.FileNotFoundException
11         //创建输出流对象
12         BufferedWriter bw=new BufferedWriter(new FileWriter("C:\\Test\\copyto.txt"));
13         //文本文件复制
14         char [] chs=new char[1024];
15         int len=0;
16         while((len=br.read(chs))!=-1) {
17             bw.write(chs, 0, len);
18         }
19         //释放资源
20         br.close();
21         bw.close();
22     }
23 }
```

缓冲区的工作原理：1、使用了底层流对象从具体设备上获取数据，并将数据存储到缓冲区的数组内。2、通过缓冲区的read()方法从缓冲区获取具体的字符数据，这样就提高了效率。3、如果用read方法读取字符数据，并存储到另一个容器中，直到读取到了换行符时，将另一个容器临时存储的数据转成字符串返回，就形成了readLine()功能。

## 3、字节流FileInputStream和FileOutputStream 

public class **FileInputStream** extends InputStream

`(1)FileInputStream` 从文件系统中的某个文件中获得输入字节。哪些文件可用取决于主机环境。

`(2)FileInputStream` 用于读取诸如图像数据之类的原始字节流。

| 构造方法摘要                                                 |
| ------------------------------------------------------------ |
| `**FileInputStream**(File file)`      通过打开一个到实际文件的连接来创建一个 `FileInputStream`，该文件通过文件系统中的 `File` 对象 `file` 指定。 |
| `**FileInputStream**(FileDescriptor fdObj)`      通过使用文件描述符 `fdObj` 创建一个 `FileInputStream`，该文件描述符表示到文件系统中某个实际文件的现有连接。 |
| `**FileInputStream**(String name)`      通过打开一个到实际文件的连接来创建一个 `FileInputStream`，该文件通过文件系统中的路径名 `name` 指定。 |

public class **FileOutputStream** extends OutputStream

(1)文件输出流是用于将数据写入 `File` 或 `FileDescriptor` 的输出流。文件是否可用或能否可以被创建取决于基础平台。特别是某些平台一次只允许一个 `FileOutputStream`（或其他文件写入对象）打开文件进行写入。在这种情况下，如果所涉及的文件已经打开，则此类中的构造方法将失败。(2) `FileOutputStream` 用于写入诸如图像数据之类的原始字节的流。

| 构造方法摘要                                                 |
| ------------------------------------------------------------ |
| `**FileOutputStream**(File file)`      创建一个向指定 `File` 对象表示的文件中写入数据的文件输出流。 |
| `**FileOutputStream**(File file, boolean append)`      创建一个向指定 `File` 对象表示的文件中写入数据的文件输出流。 |
| `**FileOutputStream**(FileDescriptor fdObj)`      创建一个向指定文件描述符处写入数据的输出文件流，该文件描述符表示一个到文件系统中的某个实际文件的现有连接。 |
| `**FileOutputStream**(String name)`      创建一个向具有指定名称的文件中写入数据的输出文件流。 |
| `**FileOutputStream**(String name, boolean append)`      创建一个向具有指定 `name` 的文件中写入数据的输出文件流。 |

例：使用字节流复制图片

```java
 1 import java.io.FileInputStream;
 2 import java.io.FileOutputStream;
 3 import java.io.IOException;
 4 
 5 public class CopImg {
 6     public static void main(String[] args) throws IOException {
 7         FileInputStream fin=new FileInputStream("C:\\Users\\Administrator\\Desktop\\Img.jpg");
 8         FileOutputStream fout=new FileOutputStream("C:\\Users\\Administrator\\Desktop\\ImgCopy.jpg");
 9         int len=0;
10         byte[] buff=new byte[1024];
11         while((len=fin.read(buff))!=-1) {
12             fout.write(buff, 0, len);
13         }
14         fin.close();
15         fout.close();
16     }
17 }
```

## 4、字节缓冲流BufferedInputStream和BufferedOutputStream

public class **BufferedInputStream** extends FilterInputStream

`BufferedInputStream` 为另一个输入流添加一些功能，即缓冲输入以及支持 `mark` 和 `reset` 方法的能力。在创建 `BufferedInputStream` 时，会创建一个内部缓冲区数组。在读取或跳过流中的字节时，可根据需要从包含的输入流再次填充该内部缓冲区，一次填充多个字节。`mark` 操作记录输入流中的某个点，`reset` 操作使得在从包含的输入流中获取新字节之前，再次读取自最后一次 `mark` 操作后读取的所有字节。

public class **BufferedOutputStream** extends FilterOutputStream

该类实现缓冲的输出流。通过设置这种输出流，应用程序就可以将各个字节写入底层输出流中，而不必针对每次字节写入调用底层系统。

例：使用字节缓冲流实现图片的复制

```java
 1 import java.io.BufferedInputStream;
 2 import java.io.BufferedOutputStream;
 3 import java.io.FileInputStream;
 4 import java.io.FileOutputStream;
 5 import java.io.IOException;
 6 
 7 public class CopyImg {
 8     public static void main(String[] args) throws IOException {
 9         BufferedInputStream bfin=new BufferedInputStream(new FileInputStream("C:\\Users\\Administrator\\Desktop\\Img.jpg"));
10         BufferedOutputStream bfout=new BufferedOutputStream(new FileOutputStream("C:\\Users\\Administrator\\Desktop\\ImgCopybuff.jpg"));
11         int len=0;
12         byte[] buff=new byte[1024];
13         while((len=bfin.read(buff))!=-1) {
14             bfout.write(buff, 0, len);
15         }
16         bfin.close();
17         bfout.close();
18     }
19 }
```

## 5、转换流：InputStreamReader和OutputStreamWriter

InputStreamReader和OutputStreamWriter是字符和字节的桥梁，也可称之为字符转换流。原理：字节流+编码。

FileReader和FileWriter作为子类，仅作为操作字符文件的便捷类存在。当操作的字符文件，使用的是默认编码表时可以不用父类，而直接使用子类完成操作，简化代码。

一旦要指定其他编码时，不能使用子类，必须使用字符转换流。

 

public class **InputStreamReader** extends Reader

(1)InputStreamReader 是字节流通向字符流的桥梁：它使用指定的 `charset` 读取字节并将其解码为字符。它使用的字符集可以由名称指定或显式给定，或者可以接受平台默认的字符集。

(2)每次调用 InputStreamReader 中的一个 read() 方法都会导致从底层输入流读取一个或多个字节。要启用从字节到字符的有效转换，可以提前从底层流读取更多的字节，使其超过满足当前读取操作所需的字节。

(3)为了达到最高效率，可以考虑在 BufferedReader 内包装 InputStreamReader。例如：

 *BufferedReader in = new BufferedReader(new InputStreamReader(System.in))；//重要*



public class **OutputStreamW**

```java
 1 import java.io.BufferedReader;
 2 import java.io.FileWriter;
 3 import java.io.IOException;
 4 import java.io.InputStream;
 5 import java.io.InputStreamReader;
 6 import java.io.Reader;
 7 
 8 /**
 9  * 使用标准输入流，读取键盘录入的数据，存储到项目根目录下的a.txt中
10  * 将字节输入流转换成字符输入流，InputStreamReader
11  */
12 public class InputStreamReaderDemo {
13     public static void main(String[] args) throws IOException {
14         //创建输入流对象
15         BufferedReader r=new BufferedReader(new InputStreamReader(System.in));
16         //创建输出流对象
17         FileWriter fw=new FileWriter("a.txt");
18         //读写数据
19         char[] chs=new char[1024];
20         int len;
21         while((len=r.read(chs))!=-1) {
22             fw.write(chs,0,len);
23             fw.flush();
24         }
25         //释放资源
26         r.close();
27         fw.close();
28 
29     }
30 
31     public static void method2() throws IOException {
32         //创建输入流对象
33         InputStream is=System.in;
34         Reader r=new InputStreamReader(is);
35         //创建输出流对象
36         FileWriter fw=new FileWriter("a.txt");
37 
38         //读写数据
39         char[] chs=new char[1024];
40         int len;
41         while((len=r.read(chs))!=-1) {
42             fw.write(chs,0,len);
43             fw.flush();
44         }
45         //释放资源
46         is.close();
47         fw.close();
48     }
49 
50     public static void method1() throws IOException {
51         //创建输入流对象
52         InputStream is=System.in;
53         //创建输出流对象
54         FileWriter fw=new FileWriter("a.txt");
55         
56         //读写数据
57         byte[] bys=new byte[1024];
58         int len;
59         while((len=is.read(bys))!=-1) {
60             fw.write(new String(bys,0,len));
61             fw.flush();
62         }
63         //释放资源
64         is.close();
65         fw.close();
66     }    
67 }

```

**riter** extends Writer

（1）OutputStreamWriter 是字符流通向字节流的桥梁：可使用指定的 `charset` 将要写入流中的字符编码成字节。它使用的字符集可以由名称指定或显式给定，否则将接受平台默认的字符集。

（2）每次调用 write() 方法都会导致在给定字符（或字符集）上调用编码转换器。在写入底层输出流之前，得到的这些字节将在缓冲区中累积。可以指定此缓冲区的大小，不过，默认的缓冲区对多数用途来说已足够大。注意，传递给 write() 方法的字符没有缓冲。

（3）为了获得最高效率，可考虑将 OutputStreamWriter 包装到 BufferedWriter 中，以避免频繁调用转换器。例如：

Writer out = new BufferedWriter(new OutputStreamWriter(System.out));*//重要*

例如：利用标准输出流将文本输出到命令行

```java
 1 import java.io.BufferedReader;
 2 import java.io.BufferedWriter;
 3 import java.io.FileNotFoundException;
 4 import java.io.FileReader;
 5 import java.io.IOException;
 6 import java.io.OutputStream;
 7 import java.io.OutputStreamWriter;
 8 import java.io.Writer;
 9 
10 /**
11  * 读取项目目录下的文件copy.java,并输出到命令行
12  * 由于标准输出流是字节输出流，所以只能输出字节或者字节数组，但是我们读取到的数据是字符串，如果想进行输出，
13  * 还需要转换成字节数组(method1)。
14  * 要想通过标准输出流输出字符串，把标准输出流转换成一种字符输出流即可(method2)。
15  */
16 public class OutputStreamWriterDemo {
17     public static void main(String[] args) throws IOException {
18         //创建输入流
19         BufferedReader br=new BufferedReader(new FileReader("copy.java"));
20         //创建输出流
21         BufferedWriter bw=new BufferedWriter(new OutputStreamWriter(System.out));
22         String line;//用于接收读到的数据
23         while((line=br.readLine())!=null) {
24             bw.write(line);
25             bw.write("\r\n");
26         }
27         br.close();
28         bw.close();
29     }
30 
31     public static void method2() throws FileNotFoundException, IOException {
32         //创建输入流
33         BufferedReader br=new BufferedReader(new FileReader("copy.java"));
34         //创建输出流
35         //OutputStream os=System.out;
36         Writer w=new OutputStreamWriter(System.out);//多态，父类引用指向子类对象
37         String line;//用于接收读到的数据
38         while((line=br.readLine())!=null) {
39             w.write(line);
40             w.write("\r\n");
41         }
42         br.close();
43         w.close();
44     }
45 
46     public static void method1() throws FileNotFoundException, IOException {
47         //创建输入流
48         BufferedReader br=new BufferedReader(new FileReader("copy.java"));
49         //创建输出流
50         OutputStream os=System.out;
51         String line;//用于接收读到的数据
52         while((line=br.readLine())!=null) {
53             os.write(line.getBytes());
54             os.write("\r\n".getBytes());
55         }
56         br.close();
57         os.close();
58     }
59 }

```

```java
 1 import java.io.FileInputStream;
 2 import java.io.FileNotFoundException;
 3 import java.io.FileOutputStream;
 4 import java.io.IOException;
 5 import java.io.InputStreamReader;
 6 import java.io.OutputStreamWriter;
 7 import java.io.UnsupportedEncodingException;
 8 
 9 public class TransStreamDemo {
10     public static void main(String[] args) throws IOException {
11         writeCN();
12         readCN();
13     }
14 
15     public static void readCN() throws UnsupportedEncodingException, FileNotFoundException, IOException {
16         //InputStreamReader将字节数组使用指定的编码表解码成文字
17         InputStreamReader isr=new InputStreamReader(new FileInputStream("temp.txt"),"utf-8");
18         char[] buff=new char[1024];
19         int len=isr.read(buff);
20         System.out.println(new String(buff,0,len));
21         isr.close();
22     }
23 
24     public static void writeCN() throws UnsupportedEncodingException, FileNotFoundException, IOException {
25         //OutputStreamWriter将字符串按照指定的编码表转成字节，再使用字符流将这些字节写出去
26         OutputStreamWriter osw=new OutputStreamWriter(new FileOutputStream("temp.txt"),"utf-8");//本身是字符流，传入字节流
27         osw.write("你好");
28         osw.close();
29     }
30 }
```



## 6、打印流PrintWriter和PrintStream

public class **PrintWriter** extends Writer

(1)向文本输出流打印对象的格式化表示形式。此类实现在 `PrintStream` 中的所有 `print` 方法。不能输出字节，但是可以输出其他任意类型。

(2)与 `PrintStream` 类不同，如果启用了自动刷新，则只有在调用 `println`、`printf` 或 `format` 的其中一个方法时才可能完成此操作，而不是每当正好输出换行符时才完成。这些方法使用平台自有的行分隔符概念，而不是换行符。

(3)此类中的方法不会抛出 I/O 异常，尽管其某些构造方法可能抛出异常。客户端可能会查询调用 `checkError()` 是否出现错误。 

```java
 1 import java.io.FileWriter;
 2 import java.io.IOException;
 3 import java.io.PrintWriter;
 4 /**
 5  * 注意：创建FileWriter对象时boolean参数表示是否追加；
 6  *              而创建打印流对象时boolean参数表示是否自动刷新
 7  */
 8 public class PrintWriterDemo {
 9     public static void main(String[] args) throws IOException {
10         //PrintWriter pw=new PrintWriter("print.txt");
11         PrintWriter pw=new PrintWriter(new FileWriter("print.txt"),true);
12         pw.write("测试打印流");
13         pw.println("此句之后换行");
14         pw.println("特有功能：自动换行和自动刷新");
15         pw.println("利用构造器设置自动刷新");
16         pw.close();
17     }
18 }
```

使用字符打印流复制文本文件：

```
 1 import java.io.BufferedReader;
 2 import java.io.FileReader;
 3 import java.io.FileWriter;
 4 import java.io.IOException;
 5 import java.io.PrintWriter;
 6 /**
 7  * 使用打印流复制文本文件
 8  */
 9 public class PrintWriterDemo {
10     public static void main(String[] args) throws IOException {
11         BufferedReader br=new BufferedReader(new FileReader("copy.java"));
12         PrintWriter pw=new PrintWriter("printcopy.java");
13         String line;
14         while((line=br.readLine())!=null) {
15             pw.println(line);
16         }
17         br.close();
18         pw.close();
19     }
20 }

```

public class PrintStream extends FilterOutputStreamimplements Appendable, Closeable
(1)PrintStream 为其他输出流添加了功能（增加了很多打印方法），使它们能够方便地打印各种数据值表示形式(例如：希望写一个整数，到目的地整数的表现形式不变。字节流的write方法只将一个整数的最低字节写入到目的地，比如写fos.write(97)，到目的地（记事本打开文件）会变成字符'a',需要手动转换：fos.write(Integer.toString(97).getBytes());而采用PrintStream：ps.print(97)，则可以保证数据的表现形式)。

```
1 //PrintStream的print方法 
2  public void print(int i) {
3         write(String.valueOf(i));
4  }
```

(2)与其他输出流不同，PrintStream 永远不会抛出 IOException；而是，异常情况仅设置可通过 checkError 方法测试的内部标志。
另外，为了自动刷新，可以创建一个 PrintStream；这意味着可在写入 byte 数组之后自动调用 flush 方法，可调用其中一个 println 方法，或写入一个换行符或字节 ('\n')。
(3)PrintStream 打印的所有字符都使用平台的默认字符编码转换为字节。在需要写入字符而不是写入字节的情况下，应该使用 PrintWriter 类。  
使用字节打印流复制文本文件：

```
1 import java.io.BufferedReader;
 2 import java.io.FileReader;
 3 import java.io.IOException;
 4 import java.io.PrintStream;
 5 
 6 public class PrintStreamDemo {
 7     public static void main(String[] args) throws IOException {
 8         BufferedReader br=new BufferedReader(new FileReader("copy.java"));
 9         PrintStream ps=new PrintStream("printcopy2.java");
10         String line;
11         while((line=br.readLine())!=null) {
12             ps.println(line);
13         }
14         br.close();
15         ps.close();
16     }
17 }
```



## 7、对象操作流ObjectInputStream和ObjectOutputStream

public class **ObjectOutputStream** extends OutputStream implements ObjectOutput, ObjectStreamConstants

（1）ObjectOutputStream 将 Java 对象的基本数据类型和图形写入 OutputStream。只能使用 ObjectInputStream 读取（重构）对象。

（2）只能将支持 java.io.Serializable 接口的对象写入流中。

（3）writeObject 方法用于将对象写入流中。所有对象（包括 String 和数组）都可以通过 writeObject 写入。可将多个对象或基元写入流中。必须使用与写入对象时相同的类型和顺序从相应 ObjectInputstream 中读回对象。

构造方法：`**ObjectOutputStream**(OutputStream out)` 　　创建写入指定 OutputStream 的 ObjectOutputStream。

public class **ObjectInputStream** extends InputStream implements ObjectInput, ObjectStreamConstants

（1）ObjectInputStream 对以前使用 ObjectOutputStream 写入的基本数据和对象进行反序列化。

（2）只有支持 java.io.Serializable 或 java.io.Externalizable 接口的对象才能从流读取。

`（3）readObject` 方法用于从流读取对象。应该使用 Java 的安全强制转换来获取所需的类型。在 Java 中，字符串和数组都是对象，所以在序列化期间将其视为对象。读取时，需要将其强制转换为期望的类型。 

例：对象读写：

```
1 import java.io.Serializable;
 2 //学生类
 3 public class Student implements Serializable{
 4     private static final long serialVersionUID = -8942780382144699003L;
 5     String name;
 6     int age;
 7     public Student(String name,int age){
 8         this.name=name;
 9         this.age=age;
10     }
11     @Override
12     public String toString() {
13         return "Student [name=" + name + ", age=" + age + "]";
14     }    
15 }

```

```
1 import java.io.EOFException;
 2 import java.io.FileInputStream;
 3 import java.io.FileNotFoundException;
 4 import java.io.FileOutputStream;
 5 import java.io.IOException;
 6 import java.io.ObjectInputStream;
 7 import java.io.ObjectOutputStream;
 8 
 9 /**
10  * 使用对象输出流写对象和对象输入流读对象
11  *注意：如果Student没有序列化，会抛出java.io.NotSerializableException
12  *Serializable：序列号，是一个标识接口，只起标识作用，没有方法
13  *当一个类的对象需要IO流进行读写的时候，这个类必须实现接口
14  */
15 public class ObjectOperate {
16     public static void main(String[] args) throws IOException, ClassNotFoundException {
17         writeObject();
18         //创建对象输入流的对象
19         ObjectInputStream ois=new ObjectInputStream(new FileInputStream("a.txt"));
20         //读取对象
21         try {
22             while(true){
23                 Object obj=ois.readObject();
24                 System.out.println(obj);
25             }
26         }catch(EOFException e){
27             System.out.println("读到了文件末尾");
28         }
29         
30         //释放资源
31         ois.close();
32         
33     }
34 
35     public static void writeObject() throws FileNotFoundException, IOException {
36         //创建对象输出流的对象
37         FileOutputStream fos=new FileOutputStream("a.txt");
38         ObjectOutputStream oos=new ObjectOutputStream(fos);
39         //创建学生对象
40         Student s1=new Student("张三",20);
41         Student s2=new Student("李四",30);
42         Student s3=new Student("王五",10);    
43         //写出学生对象
44         oos.writeObject(s1);
45         oos.writeObject(s2);
46         oos.writeObject(s3);
47         //释放资源
48     }
49 }

```

```
1 import java.io.FileInputStream;
 2 import java.io.FileNotFoundException;
 3 import java.io.FileOutputStream;
 4 import java.io.IOException;
 5 import java.io.ObjectInputStream;
 6 import java.io.ObjectOutputStream;
 7 import java.util.ArrayList;
 8 /**
 9  * 使用对象输出流写对象和对象输入流读对象
10  *解决读取对象出现异常的问题,使用集合类
11  */
12 public class ObjectOperate2 {
13     public static void main(String[] args) throws IOException, ClassNotFoundException {
14         listMethod();
15         //创建对象输入流对象
16         ObjectInputStream ois=new ObjectInputStream(new FileInputStream("b.txt"));
17         //读取数据
18         Object obj=ois.readObject();
19         //System.out.println(obj);
20         //向下转型
21         ArrayList<Student> list=(ArrayList<Student>) obj;
22         for(Student s:list) {
23             System.out.println(s);
24         }
25         //释放资源
26         ois.close();
27     }
28 
29     public static void listMethod() throws IOException, FileNotFoundException {
30         //创建对象输出流的对象
31         ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream("b.txt"));
32         //创建集合类
33         ArrayList<Student> list=new ArrayList<Student>();
34         //添加学生对象
35         list.add(new Student("zhangsan",20));
36         list.add(new Student("lisi",30));
37         //写出集合对象
38         oos.writeObject(list);
39         //释放资源
40         oos.close();
41     }
42 
43 }

```

序列化接口Serializable的作用：没有方法，不需要覆写，是一个标记接口为了启动一个序列化功能。唯一的作用就是给每一个需要序列化的类都分配一个序列版本号，这个版本号和类相关联。在序列化时，会将这个序列号也一同保存在文件中，在反序列化时会读取这个序列号和本类的序列号进行匹配，如果不匹配会抛出java.io.InvalidClassException.

注意：静态数据不会被序列化，因为静态数据在方法区，不在对象里。

或者使用transient关键字修饰，也不会序列化。

## 8、SequenceInputStream 

表示其他输入流的逻辑串联。它从输入流的有序集合开始，并从第一个输入流开始读取，直到到达文件末尾，接着从第二个输入流读取，依次类推，直到到达包含的最后一个输入流的文件末尾为止。

案例：媒体文件切割与合并

```
1 import java.io.File;
 2 import java.io.FileInputStream;
 3 import java.io.FileOutputStream;
 4 import java.io.IOException;
 5 import java.util.Properties;
 6 
 7 public class CutFile {
 8     /**
 9      * 将一个媒体文件切割成碎片
10      * 思路：1、读取源文件，将源文件的数据分别复制到多个文件
11      * 2、切割方式有两种：按照碎片个数切，或者按照指定大小切
12      * 3、一个输入流对应多个输出流
13      * 4、每个碎片都需要编号，顺序不能错
14      * @throws IOException 
15      */
16     public static void main(String[] args) throws IOException {
17         File srcFile=new File("C:\\Users\\Administrator\\Desktop\\Test\\img.jpg");
18         File partsDir=new File("C:\\Users\\Administrator\\Desktop\\cutFiles");
19         splitFile(srcFile,partsDir);
20     
21     }
22     //切割文件
23     private static void splitFile(File srcFile, File partsDir) throws IOException {
24         if(!(srcFile.exists()&&srcFile.isFile())) {
25             throw new RuntimeException("源文件不是正确文件或者不存在");
26         }
27         if(!partsDir.exists()) {
28             partsDir.mkdirs();
29         }
30         FileInputStream fis=new FileInputStream(srcFile);
31         FileOutputStream fos=null;
32         
33         byte[] buf=new byte[1024*60];
34         
35         int len=0;
36         int count=1;
37         while((len=fis.read(buf))!=-1) {
38             //存储碎片文件
39             fos=new FileOutputStream(new File(partsDir,(count++)+".part"));
40             fos.write(buf, 0, len);
41             fos.close();
42         }
43         /*将源文件和切割的信息也保存起来，随着碎片文件一起发送
44          * 信息：源文件的名称
45          * 碎片的个数
46          *将这些信息单独封装到一个文件中
47          *还要一个输出流完成此操作 */
48         
49         String fileName=srcFile.getName();
50         int partCount=count;
51         fos=new FileOutputStream(new File(partsDir,count+".properties"));
52 //        fos.write(("fileName="+fileName+System.lineSeparator()).getBytes());
53 //        fos.write(("fileCount="+Integer.toString(partCount)).getBytes());
54         Properties prop=new Properties();
55         prop.setProperty("fileName", srcFile.getName());
56         prop.setProperty("partCount", Integer.toString(partCount));
57         //将属性集中的信息持久化
58         prop.store(fos, "part file info");
59         fis.close();
60         fos.close();
61         
62     }
63 }
```

```
1 import java.io.File;
 2 import java.io.FileFilter;
 3 import java.io.FileInputStream;
 4 import java.io.FileNotFoundException;
 5 import java.io.FileOutputStream;
 6 import java.io.IOException;
 7 import java.io.SequenceInputStream;
 8 import java.util.ArrayList;
 9 import java.util.Collections;
10 import java.util.Enumeration;
11 import java.util.List;
12 import java.util.Properties;
13 
14 public class mergeFile {
15     public static void main(String[] args) throws IOException {
16         File pathDir=new File("C:\\Users\\Administrator\\Desktop\\cutFiles");
17         //获取配置文件
18         File configFile=getconfigFile(pathDir);
19         //获取配置文件信息的属性集
20         Properties prop=getProperties(configFile);
21         System.out.println(prop);
22         //获取属性集信息，将属性集信息传递到合并方法中
23         merge(pathDir,prop);
24     }
25 
26     private static Properties getProperties(File configFile) throws IOException {
27         FileInputStream fis=null;
28         Properties prop=null;
29         try {
30         //读取流和配置文件相关联
31         fis=new FileInputStream(configFile);
32         prop=new Properties();
33         //流中的数据加载到集合中
34         prop.load(fis);
35         }finally {
36             if(fis!=null) {
37                 fis.close();
38             }
39         }
40         return prop;
41     }
42 
43     public static File getconfigFile(File pathDir) {    
44         //判断是否存在properties文件
45         if(!(pathDir.exists()&&pathDir.isDirectory())) {
46             throw new RuntimeException(pathDir.toString()+"不是有效目录");
47         }
48         File[] files=pathDir.listFiles(new FileFilter() {
49             @Override
50             public boolean accept(File pathname) {
51                 return pathname.getName().endsWith(".properties");
52             }
53             
54         });
55         if(files.length!=1) {
56             throw new RuntimeException(pathDir.toString()+"properties扩展名的文件不存在或者不唯一");
57         }
58         File configFile=files[0];
59         return configFile;
60     }
61 
62     public static void merge(File pathDir, Properties prop) throws FileNotFoundException, IOException {
63         String fileName=prop.getProperty("fileName");
64         int partCount=Integer.valueOf(prop.getProperty("partCount"));
65         List<FileInputStream> list=new ArrayList<FileInputStream>();
66         for(int i=1;i<partCount;i++) {
67             list.add(new FileInputStream(pathDir.toString()+"\\"+i+".part"));
68         }
69         //List自身无法获取Enumeration工具类，到Collection中找
70         Enumeration<FileInputStream> en=Collections.enumeration(list);
71         SequenceInputStream sis=new SequenceInputStream(en);
72         FileOutputStream fos=new FileOutputStream(pathDir.toString()+"\\"+fileName);
73         byte[] buf=new byte[1024];
74         int len=0;
75         while((len=sis.read(buf))!=-1) {
76             fos.write(buf,0,len);
77         }
78         fos.close();
79         sis.close();
80     }
81 
82 }
```



## 9、用于操作数组和字符串的流对象

ByteArrayInputStream ByteArrayOutputStream

CharArrayReader 　CharArrayWriter

StringReader　　　　StringWriter

关闭这些流都是无效的，因为这些都未调用系统资源，不需要抛IO异常。

```
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;

/**
 * 源和目的都是内存的读写过程
 *用流的思想操作数组中的数据
 */
public class ByteArrayStreamDemo {
    public static void main(String[] args) {
        //源：内存
        ByteArrayInputStream bis=new ByteArrayInputStream("andhhshad".getBytes());
        //目的：内存
        ByteArrayOutputStream bos=new ByteArrayOutputStream();//内部有个可自动增长的数组
        //因为都是源和目的都是内存，没有调用底层资源，所以不要关闭，即使调用了close也没有任何效果，关闭后仍然可使用，不会抛出异常。
        int ch=0;
        while((ch=bis.read())!=-1) {
            bos.write(ch);
        }

        System.out.println(bos.toString());
    }
}
```



## 10、RandomAccessFile

![img](https://img2018.cnblogs.com/blog/1490873/201810/1490873-20181010114606004-1720813451.png)

```
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.RandomAccessFile;

/**
 * 注意：随机读写。数据需要规律。用RandomAccessFile类需要明确要操作的数据的位置。
 *
 */
public class RandomAccessFileDemo {
    public static void main(String[] args) throws IOException {
        //writeFile();
        RandomAccessFile raf=new RandomAccessFile("tempFile\\test.txt","r");
        raf.seek(8*1);//读第二个人
        byte[] buf=new byte[4];
        raf.read(buf);
        String name=new String(buf);
        System.out.println("name="+name);

        int age=raf.readInt();
        System.out.println("age="+age);
        raf.close();
    }

    public static void writeFile() throws FileNotFoundException, IOException {
        //明确要操作的位置，可以多个线程操作同一份文件而不冲突。多线程下载的基本原理。
        RandomAccessFile raf=new RandomAccessFile("tempFile\\test.txt","rw");

        raf.write("张三".getBytes());
        raf.writeInt(97);//保证字节原样性

        raf.write("李四".getBytes());
        raf.writeInt(99);//保证字节原样性

        System.out.println(raf.getFilePointer());//获取随机指针
        raf.seek(8*2);//设置指针
        raf.write("王五".getBytes());
        raf.writeInt(100);//保证字节原样性
        raf.close();
    }
}
```



## 11、File类：

File: 文件和目录路径名的抽象表示形式，File类的实例是不可改变的

（1）File类常用功能

```
File: 文件和目录路径名的抽象表示形式，File类的实例是不可改变的

 * File类的构造方法：
 *             File(String pathname) 将指定的路径名转换成一个File对象
 *             File(String parent,String child) 根据指定的父路径和文件路径创建对象
 *             File(File parent,String child)
 * File类常用功能：
 *             创建：boolean createNewFile()：当指定文件(或文件夹)不存在时创建文件并返回true，否则返回false，路径不存在则抛出异常
 *                 boolean mkdir()  ：当指定文件（或文件夹）不存在时创建文件夹并返回true，否则返回false
 *                 boolean mkdirs() :创建指定文件夹，所在文件夹目录不存在时，则顺道一块创建　　　　　　　　
 *             删除：boolean delete()：删除文件
　　　　　　　　　　　　注意：要删除一个目录，需要先删除这个目录下的所有子文件和子目录
 *             获取：File getAbsoluteFile()
 *                 File getParentFile()
 *                 String getAbsolutePath()
 *                 String getParent()
 *                 String getPath()
 *                 long lastModified() 
*             判断： boolean exists();
 *                 boolean isAbsolute() 
 *                 boolean isDirectory() 
 *                 boolean isFile() 
 *                 boolean isHidden()    
 *             修改：boolean renameTo(File dest)： 将当前File对象所指向的路径修改为指定File所指的路径 （修改文件名称）    
```

案例：打印指定文件（夹）及其所有子目录

```
 1 import java.io.File;
 2 /**
 3  *打印目录
 4  */
 5 public class FileDemo1 {
 6     public static void main(String[] args){
 7         File f=new File("E:\\BaiduNetdiskDownload");
 8         printTree( f,0);
 9     }
10     
11     public static void printTree(File f,int level) {
12         for(int j=0;j<level;j++) {
13             System.out.print("\t");
14         }
15         System.out.println(f.getAbsolutePath());
16         if(f.isDirectory()) {
17             level++;
18             File strs[]=f.listFiles();
19             for(int i=0;i<strs.length;i++) {
20                 File f0=strs[i];
21                 printTree(f0,level+1);
22             }
23         }
24     }
25 }
```

 （2）File类重要方法之过滤器

```
String[] list()
String[] list(FilenameFilter)
File[] listFiles()
File[] listFiles(FilenameFilter)
File[] listFiles(FileFilter filter)
```

File类的list方法可以获取目录下的各个文件，传入过滤器还能按照特定需求取出需要的文件。下面来看一下过滤器怎么用的。首先看

```
String[] list(FilenameFilter)
```

查看FilenameFilter源码，发现其实是一个接口：

```
public interface FilenameFilter {
    /**
     * Tests if a specified file should be included in a file list.
     *
     * @param   dir    the directory in which the file was found.
     * @param   name   the name of the file.
     * @return  <code>true</code> if and only if the name should be
     * included in the file list; <code>false</code> otherwise.
     */
    boolean accept(File dir, String name);
}
```

那么我们要想使用过滤器，应该先实现接口，假设我们要找某个文件夹下的txt文件：

```
 1 import java.io.File;
 2 import java.io.FilenameFilter;
 3 
 4 public class FileDemo {
 5     public static void main(String[] args) {
 6         File file = new File("C:\\Test");
 7         if(file.isDirectory()) {
 8             String[] list=file.list(new FilenameFilterbytxt());//传入过滤器
 9             for(String l:list) {
10                 System.out.println(l);
11             }
12         }
13     }
14 }
15 
16 class FilenameFilterbytxt implements FilenameFilter{
17     @Override
18     public boolean accept(File dir, String name) {
19         // TODO Auto-generated method stub
20         return name.endsWith(".txt");//当文件名以.txt结尾时返回true.
21     }
22     
23 }
```

但是我们看到第8行代码只是传入了过滤器，那么accept方法是如何被调用的呢？查看

String[] list(FilenameFilter) 的源码：

```
 1  /*list(FilenameFilter)源码解析*/
 2 public String[] list(FilenameFilter filter) {
 3         String names[] = list();//调用list()方法获取所有名称
 4         if ((names == null) || (filter == null)) {
 5             return names; 
 6         }
 7         List<String> v = new ArrayList<>();//用于保存过滤后的文件名
 8         for (int i = 0 ; i < names.length ; i++) {//遍历
 9             //调用filter的accept方法，传入当前目录this和遍历到的名称names[i]
10             if (filter.accept(this, names[i])) {
11                 v.add(names[i]);//满足过滤器条件的添加到集合中
12             }
13         }
14         return v.toArray(new String[v.size()]);//将集合转成数组返回，限定增删操作
15     }                        
```

也就是说，我们实现的accept方法是在构造器中被调用的。

类似地，FileFilter 也是一个接口，采用匿名内部类的方式实现接口，使用File[] listFiles(FileFilter filter)获取目录下所有文件夹：

```
 1 import java.io.File;
 2 import java.io.FileFilter;
 3 
 4 public class FileDemo {
 5     public static void main(String[] args) {
 6         File file = new File("C:\\Test");
 7         if(file.isDirectory()) {
 8             File[] list=file.listFiles(new FileFilter() {
 9                 @Override
10                 public boolean accept(File pathname) {
11                     // TODO Auto-generated method stub
12                     return pathname.isDirectory();
13                 }
14                 
15             });
16             
17             for(File f:list) {
18                 System.out.println(f);
19             }
20         }
21     }
22 }
```

 File[] listFiles(FileFilter filter) 方法的源码如下：

```
 1 /*File[] listFiles(FileFilter filter)源码解析*/
 2  public File[] listFiles(FileFilter filter) {
 3         String ss[] = list();//调用list()获取所有的名称数组
 4         if (ss == null) return null;//健壮性判断，数组为null则返回
 5         ArrayList<File> files = new ArrayList<>();//创建File类型集合
 6         for (String s : ss) {//遍历
 7             File f = new File(s, this);//private File(String child, File parent)私有构造调用
 8             if ((filter == null) || filter.accept(f))//条件判断
 9                 files.add(f);//添加到集合
10         }
11         return files.toArray(new File[files.size()]);//集合转成数组返回
12     }
```

 **12、IO流使用规律总结：**

 （1）明确要操作的数据是数据源还是数据目的(要读还是要写)

　　　　　　源：InputStream　　Reader

　　　　　　目的：OutputStream　　Writer

 （2）明确要操作的设备上的数据是字节还是文本

　　　　　　源：

　　　　　　　　　　字节：InputStream

　　　　　　　　　　文本：Reader

　　　　　　目的：

　　　　　　　　　　字节：OutputStream

　　　　　　　　　　文本：Writer

（3）明确数据所在的具体设备

　　　　　　源设备：

　　　　　　　　硬盘：文件 File开头

　　　　　　　　内存：数组，字符串

　　　　　　　　键盘：System.in

　　　　　　　　网络：Socket

　　　　　　目的设备：

　　　　　　　　硬盘：文件 File开头

　　　　　　　　内存：数组，字符串

　　　　　　　　屏幕：System.out

　　　　　　　　网络：Socket

（4）明确是否需要额外功能？

　　　　需要转换——转换流 InputStreamReader OutputStreamWriter

　　　　需要高效——缓冲流Bufferedxxx

　　　　多个源——序列流 SequenceInputStream

　　　　对象序列化——ObjectInputStream,ObjectOutputStream

　　　　保证数据的输出形式——打印流PrintStream Printwriter

　　　　操作基本数据，保证字节原样性——DataOutputStream,DataInputStream