**Java中的语法糖**

Java编程语言提供了很多语法糖，整理了下，主要有下面几种常用的语法糖。

- switch-case对String和枚举类的支持
- 泛型
- 包装类自动装箱与拆箱
- 方法变长参数
- 枚举
- 内部类
- 条件编译
- 断言
- 数值字面量
- 增强for循环
- try-with-resource语法
- Lambda表达式
- 字符串+号语法

**switch对String和枚举类的支持**

switch对枚举和String的支持原理其实是差不多的。下面以String为列子介绍下具体的原理。

switch关键字原生只能支持整数类型。如果switch后面是String类型的话，编译器会将其转换成String的hashCode的值，所以其实switch-case语法比较的是String的hashCode值。

如果switch后面是Enum类型的话，编译器会将其转换为这个枚举定义的下标（ordinal）。其实最后都是比较的整数类型。下面以Stirng举个列子。

源代码

```java
public class SwitchDemoString {
    public static void main(String[] args) {
        String str = "world";
        switch (str) {
            case "hello":
                System.out.println("hello");
                break;
            case "world":
                System.out.println("world");
                break;
            default:
                break;
        }
    }
}
```

反编译后的代码

```java
public class SwitchDemoString
{
    public switchDemoString()
    {
    }
    public static void main(String args[])
    {
        String str = "world";
        String s;
        switch((s = str).hashCode())
        {
        default:
            break;
        case 99162322:
            //这边需要再次通过equals方法进行判断，因为不同字符串的hashCode值是可能相同的，比如“Aa”和“BB”的hashCode就是一样的
            if(s.equals("hello"))
                System.out.println("hello");
            break;
        case 113318802:
            if(s.equals("world"))
                System.out.println("world");
            break;
        }
    }
}
```

通过反编译可以发现，进行switch的实际是哈希值，然后通过使用equals方法比较进行安全检查，这个检查是必要的，因为哈希可能会发生碰撞。因此它的性能是不如使用枚举进行switch或者使用纯整数常量。

PS：这边顺带简单介绍一个Java的反编译软件`Jad`。下载地址的话大家可以在网上找下。使用起来非常简单：

```bash
jad -p SwitchDemoString.class > SwitchDemoString.java
```

上面的命令就可以将class文件反编译成Java文件。这个只是`Jad`的最简单的使用，详细用法可以参考这篇[博客](https://www.cnblogs.com/dkblog/archive/2008/04/07/1980817.html)

**对泛型的支持**

在JDK5中，Java语言引入了泛型机制。但是这种泛型机制其实是通过类型擦除来实现的，即Java中的泛型只在程序源代码中有效（源代码阶段提供类型检查），在编译后的字节码中自动用强制类型转换进行替代。也就是说，Java语言中的泛型机制其实就是一颗语法糖，相较与C++、C#相比，其泛型实现实在是不那么优雅。

```java
/**
* 在源代码中存在泛型
*/
public static void main(String[] args) {
    Map<String,String> map = new HashMap<String,String>(16);
    map.put("name","csx-mg");
    String name = map.get("name");
    System.out.println(name);
}
```

通过`jad`反编译出来的代码

```java
public static void main(String[] args) {
    //类型擦除
    Map map = new HashMap();
    map.put("name", "csx-mg");
    //强制转换
    String name = (String)map.get("name");
    System.out.println(name);
}
```

通过上面反编译后的代码我们发现虚拟机中其实是没有泛型的，只有普通类和普通方法，所有泛型类的类型参数在编译时都会被擦除，泛型类并没有自己独有的Class类对象。

**包装类型的自动装箱和拆箱**

我们知道在 Java  中的8个基本类型和对应的包装类型之间是可以互相赋值的（这个过程叫自动装箱、拆箱过程）。其实这背后的原理是编译器做了优化。如下面代码，将基本类型赋值给包装类其实是调用了包装类的valueOf()方法创建了一个包装类再赋值给了基本类型。而包装类赋值给基本类型就是调用了包装类的xxxValue()方法拿到基本数据类型后再赋值的。

```java
public static void main(String[] args) {
    Integer a = 1;
    int b = 2;
    int c = a + b;
    System.out.println(c);
}
```

通过`jad`反编译出来的代码

```java
public static void main(String[] args) {
    // 自动装箱
    Integer a = Integer.valueOf(1); 
    byte b = 2;
    //自动拆箱
    int c = a.intValue() + b;
    System.out.println(c);
}
```

**变长方法参数**

变长参数特性是在JDK 5中引入的，使用变长参数有两个条件，一是变长的那一部分参数具有相同的类型，二是变长参数必须位于方法参数列表的最后面。变长参数同样是Java中的语法糖，**其内部实现是编译器在编译源代码的时候将变长参数部分转换成了Java数组。**

```java
public class Test {

    public static void main(String[] args) {
        m1("csx-mg","reading","writing","fishing");
    }

    public static void m1(String name,String... hobbits){
        System.out.println("l am "+name);
        System.out.print("l have hobbits:[");
        for (int i = 0; i < hobbits.length; i++) {
            if(i!=hobbits.length-1) {
                System.out.print(hobbits[i]+",");
            }else {
                System.out.print(hobbits[i]+"]");
            }
        }
    }
}
```

上面代码输出：

```
l am csx-mg
l have hobbits:[reading,writing,fishing]
```

通过`jad`反编译出来的代码

```java
public class Test
{

    public Test()
    {
    }

    public static void main(String args[])
    {
        //入参也被转成成了数组
        m1("csx-mg", new String[] {
            "reading", "writing", "fishing"
        });
    }
    
    //这边已经将变长参数转换成了数组
    public static transient void m1(String name, String hobbits[])
    {
        System.out.println((new StringBuilder()).append("l am ").append(name).toString());
        System.out.print("l have hobbits:[");
        for(int i = 0; i < hobbits.length; i++)
            if(i != hobbits.length - 1)
                System.out.print((new StringBuilder()).append(hobbits[i]).append(",").toString());
            else
                System.out.print((new StringBuilder()).append(hobbits[i]).append("]").toString());

    }
}
```

**枚举**

java中类的定义使用class，枚举类的定义使用enum。但在Java的字节码结构中，其实并没有枚举类型，**枚举类型只是一个语法糖，在编译完成后就会被编译成一个普通的类，也是用Class修饰。这个类继承java.lang.Enum，并被final关键字修饰。**

```java
public enum Fruit {
    APPLE,ORINGE
}	
```

将Fruit的class文件进行反编译

```java
//继承java.lang.Enum并声明为final
public final class Fruit extends Enum
{

    public static Fruit[] values()
    {
        return (Fruit[])$VALUES.clone();
    }

    public static Fruit valueOf(String s)
    {
        return (Fruit)Enum.valueOf(Fruit, s);
    }

    private Fruit(String s, int i)
    {
        super(s, i);
    }
    //枚举类型常量
    public static final Fruit APPLE;
    public static final Fruit ORANGE;
    private static final Fruit $VALUES[];//使用数组进行维护

    static
    {
        APPLE = new Fruit("APPLE", 0);
        ORANGE = new Fruit("ORANGE", 1);
        $VALUES = (new Fruit[] {
            APPLE, ORANGE
        });
    }
}
```

通过上面反编译的代码，我们可以知道当我们使用enmu来定义一个枚举类型的时候，编译器会自动帮我们创建一个final类型的类继承Enum类，所以枚举类型不能被继承。

**内部类**

Java语言中之所以引入内部类，是因为有些时候一个类只想在一个类中有用，我们不想让其在另外一个地方被使用。内部类之所以是语法糖，是因为其只是一个编译时的概念，一旦编译完成，编译器就会为内部类生成一个单独的class文件，名为outer$innter.class。

```java
public class Outer {
    class Inner{
    }
}
```

使用javac编译后，生成两个class文件Outer.class和Outer*I**n**n**e**r*.*c**l**a**s**s*，其中*O**u**t**e**r*

Inner.class的内容如下：

```java
class Outer$Inner {
    Outer$Inner(Outer var1) {
        this.this$0 = var1;
    }
}
```

**条件编译**

一般情况下，源程序中所有的行都参加编译。但有时希望对其中一部分内容只在满足一定条件下才进行编译，即对一部分内容指定编译条件，这就是“条件编译”（conditional compile）。

Java中的条件编译是通过编译器的优化原则实现的：

- 如果if的条件是false，则在编译时忽略这个if语句。
- 忽略未使用的变量。

```java
public class ConditionalCompilation02
{
    public static void main(String[] args) {
        if(CompilationConfig.DEBUG_MODE)
        {
            System.out.println("[DEBUG MODE]You can print sth");
        }
        else
        {
            System.out.println("[RELEASE MODE]You can print sth");
        }
    }
}
```

所以，Java语法的条件编译，是通过判断条件为常量的if语句实现的。根据if判断条件的真假，编译器直接把分支为false的代码块消除。通过该方式实现的条件编译，必须在方法体内实现，而无法在正整个Java类的结构或者类的属性上进行条件编译。

**断言**

在Java中，assert关键字是从Java 4开始引入的，为了避免和老版本的Java代码中使用了assert关键字导致错误，Java在执行的时候默认是不启动断言检查的（这个时候，所有的断言语句都将忽略！）。

如果要开启断言检查，则需要用开关-enableassertions或-ea来开启。

其实断言的底层实现就是if语言，如果断言结果为true，则什么都不做，程序继续执行，如果断言结果为false，则程序抛出AssertError来打断程序的执行。

```java
public class Test {
    public static void main(String[] args) {
        System.out.println("begin to test assert...");
        boolean flag;
        //当布尔值为true的时候不会抛出错误
        flag = true;
        assert flag;
        //这边会抛出AssertionError错误，但是需要加入vm参数-ea
        flag = false;
        assert flag;
    }
}
```

代码输出

```
begin to test assert...
Exception in thread "main" java.lang.AssertionError
	at com.csx.demo.spring.boot.utildemo.Test.main(Test.java:8)
```

通过`jad`反编译出来的代码

```java
public class Test
{

    public Test()
    {
    }

    public static void main(String args[])
    {
        System.out.println("begin to test assert...");
        boolean flag = true;
        if(!$assertionsDisabled && !flag)
            throw new AssertionError();
        flag = false;
        if(!$assertionsDisabled && !flag)
            throw new AssertionError();
        else
            return;
    }

    static final boolean $assertionsDisabled = !com/csx/demo/spring/boot/utildemo/Test.desiredAssertionStatus();

}
```

通过反编译后的代码可以看出assert断言就是通过对布尔标志位进行了一个if判断。（需要注意的是如果需要使用断言，需要设置JVM参数-ea开始断言）

**数值字面量**

Java中支持如下形式的数值字面量

- 十进制：默认的
- 八进制：整数之前加数字0来表示
- 十六进制：整数之前加"0x"或"0X"
- 二进制（新加的）：整数之前加"0b"或"0B"

另外在在jdk7中，数值字面量，不管是整数还是浮点数，都允许在数字之间插入任意多个下划线。这些下划线不会对字面量的数值产生影响，目的就是方便阅读。比如：

- 1_500_000
- 5_6.3_4
- 89_3___1

下划线只能出现在数字中间，前后必须是数字。所以“_100”、“0b_101“是不合法的，无法通过编译。这样限制的动机就是可以降低实现的复杂度。有了这个限制，Java编译器只需在扫描源代码的时候将所发现的数字中间的下划线直接删除就可以了。如果不添加这个限制，编译器需要进行语法分析才能做出判断。比如：_100,可能是一个整数字面量100，也可能是一个变量名称。这就要求编译器的实现做出更复杂的改动。

```java
public class Test {
    public static void main(String[] args) {
        //十进制
        int a = 10;
        //二进制
        int b = 0B1010;
        //八进制
        int c = 012;
        //十六进制
        int d = 0XA;

        double e = 12_234_234.23;
        System.out.println("a："+a);
        System.out.println("b："+b);
        System.out.println("c："+c);
        System.out.println("d："+d);
        System.out.println("e："+e);
    }
}
```

通过`jad`反编译出来的代码

```java
public class Test
{

    public Test()
    {
    }

    public static void main(String args[])
    {
        int a = 10;
        //编译器已经将二进制，八进制，十六进制数转换成了10进制数
        int b = 10;
        int c = 10;
        int d = 10;
        //编译器已经将下滑线删除
        double e = 12234234.23D;
        System.out.println((new StringBuilder()).append("a\uFF1A").append(a).toString());
        System.out.println((new StringBuilder()).append("b\uFF1A").append(b).toString());
        System.out.println((new StringBuilder()).append("c\uFF1A").append(c).toString());
        System.out.println((new StringBuilder()).append("d\uFF1A").append(d).toString());
        System.out.println((new StringBuilder()).append("e\uFF1A").append(e).toString());
    }
}
```

**增强for循环**

增强for循环的对象要么是一个数组，要么实现了Iterable接口。这个语法糖主要用来对数组或者集合进行遍历，其在循环过程中不能改变集合的大小。增强for循环主要使代码更加简洁，其背后的原理是编译器将增强for循环转换成了普通的for循环或者while循环。

```java
public static void main(String[] args) {
    String[] params = new String[]{"hello","world"};
    //增强for循环对象为数组
    for(String str : params){
        System.out.println(str);
    }

    List<String> lists = Arrays.asList("hello","world");
    //增强for循环对象实现Iterable接口
    for(String str : lists){
        System.out.println(str);
    }
}
```

编译器编译后的代码

```java
public static void main(String[] args) {
   String[] params = new String[]{"hello", "world"};
   String[] lists = params;
   int var3 = params.length;
   //数组形式的增强for退化为普通for
   for(int str = 0; str < var3; ++str) {
       String str1 = lists[str];
       System.out.println(str1);
   }

   List var6 = Arrays.asList(new String[]{"hello", "world"});
   Iterator var7 = var6.iterator();
   //实现Iterable接口的增强for使用iterator接口进行遍历
   while(var7.hasNext()) {
       String var8 = (String)var7.next();
       System.out.println(var8);
   }

}
```

**try-with-resource语法**

当一个外部资源的句柄对象实现了AutoCloseable接口，JDK7中便可以利用try-with-resource语法更优雅的关闭资源，消除板式代码。

```java
public static void main(String[] args) {
    try (FileInputStream inputStream = new FileInputStream(new File("test"))) {
        System.out.println(inputStream.read());
    } catch (IOException e) {
        throw new RuntimeException(e.getMessage(), e);
    }
}
```

将外部资源的句柄对象的创建放在try关键字后面的括号中，当这个try-catch代码块执行完毕后，Java会确保外部资源的close方法被调用。代码是不是瞬间简洁许多！try-with-resource并不是JVM虚拟机的新增功能，只是JDK实现了一个语法糖，当你将上面代码反编译后会发现，其实对JVM虚拟机而言，它看到的依然是之前的写法：

```java
public static void main(String[] args) {
    try {
        FileInputStream inputStream = new FileInputStream(new File("test"));
        Throwable var2 = null;
        try {
            System.out.println(inputStream.read());
        } catch (Throwable var12) {
            var2 = var12;
            throw var12;
        } finally {
            if (inputStream != null) {
                if (var2 != null) {
                    try {
                        inputStream.close();
                    } catch (Throwable var11) {
                        var2.addSuppressed(var11);
                    }
                } else {
                    inputStream.close();
                }
            }
        }

    } catch (IOException var14) {
        throw new RuntimeException(var14.getMessage(), var14);
    }
}
```

其实背后的原理也很简单，那些我们没有做的关闭资源的操作，编译器都帮我们做了。

**Lambda表达式**

Lambda表达式虽然看着很先进，但其实Lambda表达式的本质只是一个"语法糖",**由编译器推断并帮你转换包装为常规的代码**,因此你可以使用更少的代码来实现同样的功能。**本人建议不要乱用,因为这就和某些很高级的黑客写的代码一样,简洁,难懂,难以调试,维护人员想骂娘**。

lambda表达式允许你通过表达式来代替功能接口。Lambda表达式还增强了集合库。 Java SE  8添加了2个对集合数据进行批量操作的包: java.util.function 包以及java.util.stream 包。  流(stream)就如同迭代器(iterator),但附加了许多额外的功能。 总的来说,lambda表达式和 stream  是自Java语言添加泛型(Generics)和注解(annotation)以来最大的变化。

**Lambda表达式的语法**

```java
基本语法:
(parameters) -> expression
或
(parameters) ->{ statements; }
```

Lambda表达式的一些简单列子

```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```

**基本的Lambda例子(实现功能接口)**

```java
String[] atp = {"Rafael Nadal", "Novak Djokovic",
                "Stanislas Wawrinka",
                "David Ferrer", "Roger Federer",
                "Andy Murray", "Tomas Berdych",
                "Juan Martin Del Potro"};

        List<String> players =  Arrays.asList(atp);

        //实现功能接口
        players.forEach((String player) ->{
            System.out.println(player);
        });
        Runnable runnable = () -> {
            System.out.println("l am a new thread...");
        };
        new Thread(runnable).start();   
```

**使用Lambdas排序集合**

```java
players.sort(new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o1.compareTo(o2);
            }
        });
Comparator<String> comparator = (String o1,String o2) ->{return o1.compareTo(o2);};
players.sort(comparator);
```

**使用Lambdas和Streams**

Stream是对集合的包装,通常和lambda一起使用。 使用lambdas可以支持许多操作,如 map, filter, limit,  sorted, count, min, max, sum, collect 等等。  同样,Stream使用懒运算,他们并不会真正地读取所有数据,遇到像getFirst() 这样的方法就会结束链式语法。

**字符串对+号的支持**

```java
String s=null;
s=s+"abc";
System.out.println(s);
```

上面的代码输出什么？

字符串+号拼接原理：运行时，两个字符串str1, str2的拼接首先会new一个`StringBuilder`对象，然后分别对字符串进行append操作，最后调用`toString()`方法。

看下反编译出来的代码：

```java
public static void main(String args[])
{
    String s = null;
    s = (new StringBuilder()).append(s).append("abc").toString();
    System.out.println(s);
}
```

所以答案是：nullabc

但是如果在编译期能确定字符相加的结果，则会进行编译期优化。

```java
String s = "a" + "b"
```

对于上面的表达式，编译器直接优化成

```
String s = "ab";
```