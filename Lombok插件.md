 Java开发神器Lombok使用详解

# 什么是Lombok

Lombok是一款Java开发插件，可以通过它定义的注解来精简冗长和繁琐的代码，主要针对简单的Java模型对象（POJO）。

好处就显而易见了，可以节省大量重复工作，特别是当POJO类的属性增减时，需要重复修改的Getter/Setter、构造器方法、equals方法和toString方法等。

而且Lombok针对这些内容的处理是在编译期，而不是通过反射机制，这样的好处是并不会降低系统的性能。

下面我们就看看具体的使用。
Lombok的安装

Lombok的安装分两部分：Idea插件的安装和maven中pom文件的导入。

第一步，在Idea的插件配置中搜索Lombok（可能需要梯子）或官网下载本地安装。

![image](C:\Users\tzq\Desktop\Java整理\Lombok插件\lombok-1.jpg)

同时，在插件的描述中也能够看到它支持的注解。

第二步，引入pom中依赖，当前最细版本1.18.10。

```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.10</version>
</dependency>
```

如果是通过Idea创建Spring Boot项目，可在创建项目时直接在“Developer Tool”中选择Lombok。

完成了以上两步，就可以在代码中使用该款神器了。
Lombok的使用
@Data

@Data最常用的注解之一。注解在类上，提供该类所有属性的getter/setter方法，还提供了equals、canEqual、hashCode、toString方法。

这里的提供什么意思？就是开发人员不用手写相应的方法，而Lombok会帮你生成。

使用@Data示例如下，最直观的就是不用写getter/setter方法。

```java
@Data
public class Demo {
	private int id;
	private String remark;
}
```

我们看该类编译之后是什么样子。

```java
public class Demo {
    private int id;
    private String remark;

    public Demo() {
    }
    
    public int getId() {
        return this.id;
    }
    
    public String getRemark() {
        return this.remark;
    }
    
    public void setId(final int id) {
        this.id = id;
    }
    
    public void setRemark(final String remark) {
        this.remark = remark;
    }
    
    public boolean equals(final Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof Demo)) {
            return false;
        } else {
            Demo other = (Demo)o;
            if (!other.canEqual(this)) {
                return false;
            } else if (this.getId() != other.getId()) {
                return false;
            } else {
                Object this$remark = this.getRemark();
                Object other$remark = other.getRemark();
                if (this$remark == null) {
                    if (other$remark != null) {
                        return false;
                    }
                } else if (!this$remark.equals(other$remark)) {
                    return false;
                }
    
                return true;
            }
        }
    }
    
    protected boolean canEqual(final Object other) {
        return other instanceof Demo;
    }
    
    public int hashCode() {
        int PRIME = true;
        int result = 1;
        int result = result * 59 + this.getId();
        Object $remark = this.getRemark();
        result = result * 59 + ($remark == null ? 43 : $remark.hashCode());
        return result;
    }
    
    public String toString() {
        return "Demo(id=" + this.getId() + ", remark=" + this.getRemark() + ")";
    }

}
```

上面的反编译代码，我们可以看到提供了默认的构造方法、属性的getter/setter方法、equals、canEqual、hashCode、toString方法。

使用起来是不是很方便，最关键的是，当新增属性或减少属性时，直接删除属性定义即可，效率是否提升了很多？

为了节省篇幅，后面相关注解我们就不再看反编译的效果了，大家使用idea直接打开编译之后对应的.class文件即可看到。
@Setter

作用于属性上，为该属性提供setter方法; 作用与类上，为该类所有的属性提供setter方法， 都提供默认构造方法。

```java
public class Demo {
	private int id;
	@Setter
	private String remark;
}

@Setter
public class Demo {
	private int id;
	private String remark;
}
```

@Getter

基本使用同@Setter方法，不过提供的是getter方法，不再赘述。
@Log4j

作用于类上，为该类提供一个属性名为log的log4j日志对象。

```java
@Log4j
public class Demo {
}
```

该属性一般使用于Controller、Service等业务处理类上。与此注解相同的还有@Log4j2，顾名思义，针对Log4j2。
@AllArgsConstructor

作用于类上，为该类提供一个包含全部参的构造方法，注意此时默认构造方法不会提供。

```java
@AllArgsConstructor
public class Demo {
	private int id;
	private String remark;
}
```

效果如下：

```java
public class Demo {
    private int id;
    private String remark;

    public Demo(final int id, final String remark) {
        this.id = id;
        this.remark = remark;
    }

}
```

@NoArgsConstructor

作用于类上，提供一个无参的构造方法。可以和@AllArgsConstructor同时使用，此时会生成两个构造方法：无参构造方法和全参构造方法。
@EqualsAndHashCode

作用于类上，生成equals、canEqual、hashCode方法。具体效果参看最开始的@Data效果。
@NonNull

作用于属性上，提供关于此参数的非空检查，如果参数为空，则抛出空指针异常。

使用方法：

```java
public class Demo {
	@NonNull
	private int id;
	private String remark;
}
```

效果如下：

```java
public class Demo {
    @NonNull
    private int id;
    private String remark;
}
```

@RequiredArgsConstructor

作用于类上，由类中所有带有@NonNull注解或者带有final修饰的成员变量作为参数生成构造方法。
@Cleanup

作用于变量，保证该变量代表的资源会被自动关闭，默认调用资源的close()方法，如果该资源有其它关闭方法，可使用@Cleanup(“methodName”)来指定。

```java
public void jedisExample(String[] args) {
    try {
        @Cleanup Jedis jedis =   redisService.getJedis();
    } catch (Exception ex) {
        logger.error(“Jedis异常:”,ex)
    }
}
```

效果相当于：

```java
public void jedisExample(String[] args) {

    Jedis jedis= null;
    try {
        jedis = redisService.getJedis();
    } catch (Exception e) {
        logger.error(“Jedis异常:”,ex)
    } finally {
        if (jedis != null) {
            try {
                jedis.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}
```

@ToString

作用于类上，生成包含所有参数的toString方法。见@Data中toString方法。
@Value

作用于类上，会生成全参数的构造方法、getter方法、equals、hashCode、toString方法。与@Data相比多了全参构造方法，少了默认构造方法、setter方法和canEqual方法。

该注解需要注意的是：会将字段添加上final修饰，个人感觉此处有些失控，不太建议使用。
@SneakyThrows

作用于方法上，相当于把方法内的代码添加了一个try-catch处理，捕获异常catch中用Lombok.sneakyThrow(e)抛出异常。使用@SneakyThrows(BizException.class)指定抛出具体异常。

```
@SneakyThrows
public int getValue(){
	int a = 1;
	int b = 0;
	return a/b;
}
```

效果如下：

```java
public int getValue() {
    try {
        int a = 1;
        int b = 0;
        return a / b;
    } catch (Throwable var3) {
        throw var3;
    }
}
```

@Synchronized

作用于类方法或实例方法上，效果与synchronized相同。区别在于锁对象不同，对于类方法和实例方法，synchronized关键字的锁对象分别是类的class对象和this对象，而@Synchronized的锁对象分别是私有静态final对象lock和私有final对象lock。也可以指定锁对象。

```java
public class FooExample { 

 private final Object readLock = new Object(); 

 @Synchronized 
 public static void hello() { 
     System.out.println("world");   
 } 

 @Synchronized("readLock") 
 public void foo() { 
   System.out.println("bar"); 
 } 

}
```

效果相当于如下：

```java
public class FooExample { 

  private static final Object $LOCK = new Object[0]; 
  private final Object readLock = new Object(); 

  public static void hello() { 
    synchronized($LOCK) { 
      System.out.println("world"); 
    } 
  }   

  public void foo() { 
    synchronized(readLock) { 
        System.out.println("bar");   
    } 
  } 
}
```

val

使用val作为局部变量声明的类型，而不是实际写入类型。 执行此操作时，将从初始化表达式推断出类型。

```java
public Map<String, String> getMap() {
	val map = new HashMap<String, String>();
	map.put("1", "a");
	return map;
}
```

效果如下：

```java
public Map<String, String> getMap() {
    HashMap<String, String> map = new HashMap();
    map.put("1", "a");
    return map;
}
```

也就是说在局部变量中，Lombok帮你推断出具体的类型，但只能用于局部变量中。
@Builder

作用于类上，如果你喜欢使用Builder的流式操作，那么@Builder可能是你喜欢的注解了。

使用方法：

```java
@Builder
public class Demo {
	private int id;
	private String remark;
}
```

效果如下：

```java
public class Demo {
    private int id;
    private String remark;

    Demo(final int id, final String remark) {
        this.id = id;
        this.remark = remark;
    }
    
    public static Demo.DemoBuilder builder() {
        return new Demo.DemoBuilder();
    }
    
    public static class DemoBuilder {
        private int id;
        private String remark;
    
        DemoBuilder() {
        }
    
        public Demo.DemoBuilder id(final int id) {
            this.id = id;
            return this;
        }
    
        public Demo.DemoBuilder remark(final String remark) {
            this.remark = remark;
            return this;
        }
    
        public Demo build() {
            return new Demo(this.id, this.remark);
        }
    
        public String toString() {
            return "Demo.DemoBuilder(id=" + this.id + ", remark=" + this.remark + ")";
        }
    }

}
```

我们可以看到，在该类内部提供了DemoBuilder类用来处理具体的流式操作。同时提供了全参的构造方法。
小结

最后，说一下个人的看法，此神器虽然好用，但也不建议大家无条件的使用，为了程序的效率等问题，该自己亲手写的代码还是要自己亲手写。毕竟，只有定制化的才能达到最优化和最符合当前场景。

# Lombok有什么坏处？

### 1.要求开发者一定要在IDE中安装对应的插件

因为Lombok的使用要求开发者一定要在IDE中安装对应的插件。

如果未安装插件的话，使用IDE打开一个基于Lombok的项目的话会提示找不到方法等错误。导致项目编译失败。

也就是说，如果项目组中有一个人使用了Lombok，那么其他人就必须也要安装IDE插件。否则就没办法协同开发。

更重要的是，如果我们定义的一个jar包中使用了Lombok，那么就要求所有依赖这个jar包的所有应用都必须安装插件，这种侵入性是很高的。
代码可读性，可调试性低

在代码中使用了Lombok，确实可以帮忙减少很多代码，因为Lombok会帮忙自动生成很多代码。

但是这些代码是要在编译阶段才会生成的，所以在开发的过程中，其实很多代码其实是缺失的。

在代码中大量使用Lombok，就导致代码的可读性会低很多，而且也会给代码调试带来一定的问题。

比如，我们想要知道某个类中的某个属性的getter方法都被哪些类引用的话，就没那么简单了。

### 2.过度依赖

因为Lombok使代码开发非常简便，这就使得部分开发者对其产生过度依赖。

在使用Lombok过程中，如果对于各种注解的底层原理不理解的话，很容易产生意想不到的结果。

举一个简单的例子，我们知道，当我们使用@Data定义一个类的时候，会自动帮我们生成equals()方法 。

但是如果只使用了@Data，而不使用@EqualsAndHashCode(callSuper=true)的话，会默认是@EqualsAndHashCode(callSuper=false),这时候生成的equals()方法只会比较子类的属性，不会考虑从父类继承的属性，无论父类属性访问权限是否开放。

这就可能得到意想不到的结果。

### 3.影响升级

因为Lombok对于代码有很强的侵入性，就可能带来一个比较大的问题，那就是会影响我们对JDK的升级。

按照如今JDK的升级频率，每半年都会推出一个新的版本，但是Lombok作为一个第三方工具，并且是由开源团队维护的，那么他的迭代速度是无法保证的。

所以，如果我们需要升级到某个新版本的JDK的时候，若其中的特性在Lombok中不支持的话就会受到影响。

还有一个可能带来的问题，就是Lombok自身的升级也会受到限制。

因为一个应用可能依赖了多个jar包，而每个jar包可能又要依赖不同版本的Lombok，这就导致在应用中需要做版本仲裁，而我们知道，jar包版本仲裁是没那么容易的，而且发生问题的概率也很高。

### 4.破坏封装性

以上几个问题，我认为都是有办法可以避免的。但是有些人排斥使用Lombok还有一个重要的原因，那就是他会破坏封装性。

众所周知，Java的三大特性包括封装性、继承性和多态性。

如果我们在代码中直接使用Lombok，那么他会自动帮我们生成getter、setter 等方法，这就意味着，一个类中的所有参数都自动提供了设置和读取方法。

举个简单的例子，我们定义一个购物车类：

```java
@Data

public class ShoppingCart { 

    //商品数目
    private int itemsCount; 
    
    //总价格
    private double totalPrice; 
    
    //商品明细
    private List items = new ArrayList<>();

}
```


我们知道，购物车中商品数目、商品明细以及总价格三者之前其实是有关联关系的，如果需要修改的话是要一起修改的。

但是，我们使用了Lombok的@Data注解，对于itemsCount 和 totalPrice这两个属性。虽然我们将它们定义成 private 类型，但是提供了 public 的 getter、setter 方法。

外部可以通过 setter 方法随意地修改这两个属性的值。我们可以随意调用 setter 方法，来重新设置 itemsCount、totalPrice 属性的值，这也会导致其跟 items 属性的值不一致。

而面向对象封装的定义是：通过访问权限控制，隐藏内部数据，外部仅能通过类提供的有限的接口访问、修改内部数据。所以，暴露不应该暴露的 setter 方法，明显违反了面向对象的封装特性。

好的做法应该是不提供getter/setter，而是只提供一个public的addItem方法，同时去修改itemsCount、totalPrice以及items三个属性。