Optional 是 Java 8 引进的一个新特性，我们通常认为Optional是用于缓解Java臭名昭著的空指针异常问题。

Brian Goetz （Java语言设计架构师）对Optional设计意图的原话如下：

> Optional is intended to provide a limited mechanism for library method return  types where there needed to be a clear way to represent “no result," and using null for such was overwhelmingly likely to cause errors.

这句话突出了三个点：

1. Optional 是用来作为方法返回值的
2. Optional 是为了清晰地表达返回值中没有结果的可能性
3. 且如果直接返回 null 很可能导致调用端产生错误（尤其是NullPointerException）

Optional的机制类似于 Java 的受检异常，强迫API调用者面对没有返回值的现实。

参透Optional的设计意图才能学会正确得使用它。

以下围绕这三个点阐述Optional的最佳实践。





**Optional 是用来作为方法返回值的**

1. **不要滥用 Optional API**

 有的同学知道了一些Optional的API后就觉得找到了一把锤子，看到什么都像钉子。

于是写出了以下这种代码

```java
String finalStatus = Optional.ofNullable(status).orElse("PENDING")
```

这种写法不仅降低了代码可读性还无谓得创建了一个Optional对象（浪费性能）

以下是同等功能但更简洁更可读的实现

```java
String finalStatus = status == null ? "PENDING" : status;
```



**2. 不要使用Optional作为Java Bean实例域的类型**

即避免以下这种代码

```java
// AVOID
public class Customer {
    [access_modifier] [static] [final] Optional<String> zip;
    [access_modifier] [static] [final] Optional<String> telephone = Optional.empty();
    ...
}
```

因为 Optional 没有实现Serializable接口（不可序列化）



**3. 不要使用 Optional 作为类构造器参数**

即避免以下这种代码

```java
// AVOID
public class Customer {
    private final String name;               // cannot be null
    private final Optional<String> postcode; // optional field, thus may be null
    public Customer(String name, Optional<String> postcode) {
        this.name = Objects.requireNonNull(name, () -> "Name cannot be null");
        this.postcode = postcode;
    }
    public Optional<String> getPostcode() {
        return postcode;
    }
    ...
}
```

可以看到这种写法只是无谓地增加了一层包装和样板代码。



**4. 不要使用 Optional 作为Java Bean Setter方法的参数**

即避免以下这种代码

```java
// AVOID
@Entity
public class Customer implements Serializable {
    private static final long serialVersionUID = 1L;
    ...
    @Column(name="customer_zip")
    private Optional<String> postcode; // optional field, thus may be null
     public Optional<String> getPostcode() {
       return postcode;
     }
     public void setPostcode(Optional<String> postcode) {
       this.postcode = postcode;
     }
     ...
}
```

原因除了上面第二点提到的 Optional 是不可序列化的，还有降低了可读性。
既然 setter是用于给Java Bean 属性赋值的， 为什么还无法确定里面的值是不是空 ？ 如果为空，为何不直接赋值 null （或者对应的空值） ？

但相反的是，对于可能是空值Java Bean 属性的Getter 方法返回值使用Optional类型是很好的实践

```java
// PREFER
@Entity
public class Customer implements Serializable {
    private static final long serialVersionUID = 1L;
    ...
    @Column(name="customer_zip")
    private String postcode; // optional field, thus may be null
    public Optional<String> getPostcode() {
      return Optional.ofNullable(postcode);
    }
    public void setPostcode(String postcode) {
       this.postcode = postcode;
    }
    ...
}
```

由于getter返回的是Optional，外部调用时就意识到里面可能是空结果，需要进行判断。

注意：对值可能为 null 的实例域的getter 才需要使用 Optional



**5. 不要使用Optional作为方法参数的类型**

这一点比较有争议，但我支持不使用Optional作为方法参数的类型。

首先，当参数类型为Optional时，所有API调用者都需要给参数先包一层Optional（额外创建一个Optional实例）浪费性能 —— 一个Optional对象的大小是简单引用的4倍。

其次，当方法有多个Optional参数时，方法签名会变得更长，可读性更差。比如：

```java
public void renderCustomer(Optional<Cart> cart, Optional<Renderer> renderer, Optional<String> name) {   
  ...
}
```

最后，Optional参数给方法实现增加了更多检查负担 —— 以上面 renderCustomer 方法为例

cart 参数实际上有三种可能性： 

1) . cart 为 null

2). cart 为 Optional.empty()

3). cart 为 Optional.of(xxx)



如果只是为了设计可选的方法参数，方法重载是个传统的且实用的方案，而且对调用者更友好

```java
public void renderCustomer(Cart cart, Renderer renderer, String name) {   
  ...
}

public void renderCustomer(Cart cart, Renderer renderer) {   
  ...
}

public void renderCustomer(Cart cart) {   
  ...
}
```





**6. 不要在集合中使用 Optional 类**

不要在 List, Set, Map 等集合中使用任何的 Optional 类作为**键，值或者元素**，因为没有任何意义。

对于Map的值，可以使用 getOrDefault() 或者 computeIfAbsent() 方法设置默认值



**7. 不要把容器类型（包括 List, Set, Map, 数组, Stream 甚至 Optional ）包装在Optional中**

即避免

```java
// AVOID
public Optional<List<String>> fetchCartItems(long id) {
    Cart cart = ... ;    
    List<String> items = cart.getItems(); // this may return null
    return Optional.ofNullable(items);
}
```

因为容器类都有自己空值设计，如 Collections.emptyList() Collections.emptySet() Collections.emptyMap()  Stream.empty() 等

```java
// PREFER
public List<String> fetchCartItems(long id) {
    Cart cart = ... ;    
    List<String> items = cart.getItems(); // this may return null
    return items == null ? Collections.emptyList() : items;
}
```



**Optional 是为了清晰地表达返回值中没有结果的可能性**

**8. 不要给Optional变量赋值 null**

```java
// AVOID
public Optional<Cart> fetchCart() {
    Optional<Cart> emptyCart = null;
    ...
}
```

而应该用 Optional.empty() 表达空值

```java
// PREFER
public Optional<Cart> fetchCart() {
    Optional<Cart> emptyCart = Optional.empty();
    ...
}
```



**9.  确保Optional内有值才能调用 get() 方法**

如果不检查Optional是否为空值就直接调用get() 方法，就让 Optional 失去了意义 —— Optional 是为了清晰地表达返回值中没有结果的可能性，强迫API调用者面对没有返回值的现实并做检查。

目前Java 8编译器并不会对这种情况报错，但是 IDE 已经可以识别并警告

![img](C:\Users\tzq\Desktop\Java整理\Optional处理空指针\v2-d72d6b0ee15cfb5256633a42d7ab7338_720w.jpg)

所以避免 

```java
// AVOID
Optional<Cart> cart = ... ; // this is prone to be empty
...
// if "cart"is empty then this code will throw a java.util.NoSuchElementException
Cart myCart = cart.get();
```

而应该

```java
// PREFER
if (cart.isPresent()) {
    Cart myCart = cart.get();
    ... // do something with "myCart"
} else {
    ... // do something that doesn't call cart.get()
}
```



**10. 尽量使用 Optional 提供的快捷API 避免手写 if-else 语句**

在一些场景下， Optional.orElse()  Optional.orElseGet() Optional.ifPresent() 可以避免手写 if-else 语句，使代码更简洁

具体使用方法可以查官方API

简单示例：

```java
// PREFER
public static final String USER_STATUS = "UNKNOWN";
...
public String findUserStatus(long id) {
    Optional<String> status = ... ; // prone to return an empty Optional
    return status.orElse(USER_STATUS);
}


// PREFER
public String computeStatus() {
    ... // some code used to compute status
}
public String findUserStatus(long id) {
    Optional<String> status = ... ; // prone to return an empty Optional
    // computeStatus() is called only if "status" is empty
    return status.orElseGet(this::computeStatus);
}


// PREFER
Optional<String> status ... ;
...
status.ifPresent(System.out::println);
```



如果正在学习或者使用 java 9 甚至 更高版本，Optional 有一些更新的 API （如 ifPresentOrElse ），本文不讨论。



**11. 使用 equals 而不是 == 来比较 Optional 的值**

Optional 的 equals 方法已经实现了内部值比较

```java
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Optional)) {
            return false;
        }

        Optional<?> other = (Optional<?>) obj;
        return Objects.equals(value, other.value);
    }
```

所以

```java
Product product = new Product();
Optional<Product> op1 = Optional.of(product);
Optional<Product> op2 = Optional.of(product);


if (op1.equals(op2)) { 
   ... //expected true
}

if (op1 == op2){
   ... //expected false
}
```



------

**总结**

1. Optional 尽量只用来作为方法返回值类型
2. 调用了返回值为Optional的方法后，一定要做空值检查
3. 不要过度使用 Optional 避免降低代码可读性和性能
4. 查阅并适当使用 Optional API

**实例演示**

请大家赏析下面这段代码：

```java
public class NullPointerTest {
    /**
     * 需求：根据用户名查找该用户所在的部门名称
     *
     * @param args
     */
    public static void main(String[] args) {
        String departmentNameOfUser = getDepartmentNameOfUser("test");
        System.out.println(departmentNameOfUser);
    }

    /**
     * 假设这是A-Service的服务
     * 这一步很烦！！！
     *
     * @param username
     * @return
     */
    public static String getDepartmentNameOfUser(String username) {
        ResultTO<User> resultTO = getUserByName(username);
        if (resultTO != null) {
            User user = resultTO.getData();
            if (user != null) {
                Department department = user.getDepartment();
                if (department != null) {
                    return department.getName();
                }
            }
        }
        return "未知部门";
    }

    /**
     * 假设这是B-Service的服务（不用关注具体逻辑，就是随机模拟返回值，可能为null）
     *
     * @param username
     * @return
     */
    public static ResultTO<User> getUserByName(String username) {
        if (username == null || "".equals(username)) {
            return null;
        }

        Department department;
        User user;

        if (ThreadLocalRandom.current().nextBoolean()) {
            department = new Department("总裁办", 10086);
        } else {
            department = null;
        }
        if (ThreadLocalRandom.current().nextBoolean()) {
            user = new User("周董", 18, department);
            user.setDepartment(department);
        } else {
            user = null;
        }

        return ResultTO.buildSuccess(user);
    }


    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    static class User {
        private String name;
        private Integer age;
        private Department department;
    }

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    static class Department {
        private String name;
        private Integer code;
    }

    @Getter
    @Setter
    static class ResultTO<T> implements Serializable {

        private Boolean success;
        private String message;
        private T data;

        public static <T> ResultTO<T> buildSuccess(T data) {
            ResultTO<T> result = new ResultTO<>();
            result.setSuccess(true);
            result.setMessage("success");
            result.setData(data);
            return result;
        }

    }

}
```

你会发现，如果一个POJO的**层级过深**而且**恰好作为返回值**返回时，调用者将苦不堪言，为了避免空指针不得不写一大堆的if判断，也就是空指针探测。稍微好看点的写法是：

```java
/**
 * 假设这是A-Service的服务
 * 这一步很烦！！！
 *
 * @param username
 * @return
 */
public static String getDepartmentNameOfUser(String username) {
    ResultTO<User> resultTO = getUserByName(username);
    if (resultTO == null) {
        return "ResultTO为空";
    }
    
    User user = resultTO.getData();
    if (user == null) {
        return "User为空";
    }
    
    Department department = user.getDepartment();
    if (department == null) {
        return "Department为空";
    }
    
    return department.getName();
}
```

虽然避免了过深的if嵌套，逻辑稍微清晰一点，但还是很啰嗦。

**封装NullWrapper**

我们来尝试封装一个工具类，希望能简化NullPointerException的探测工作。

**核心思想**

设计一个Wrapper，内部有一个T value字段，用来接收返回值，这样就能把null包裹在内部，稍微安全了一些，因为Wrapper肯定不为null：new Wrapper(value).getXxx()。

但这还不够！因为如果这个value是null，而外界getValue()后再次调用的话，仍然会发生NullPointerException。

怎么处理？

其实答案已经出现过了：再次把返回值包装成Wrapper即可。

比如：

```java
public static void main(String[] agrs) {
    Wrapper resultWrapper = new Wrapper(getDepartmentnameByName(username));
    Wrapper userWrapper = new Wrapper(resultWrapper.get()); // resultTO可能为null，所以再次包装
    Wrapper departmentWrapper = new Wrapper(userWrapper.get());
    Wrapper departmentNameWrapper = new Wrapper(departmentWrapper.get());
    // ...
}
```

但上面的代码有个问题：

> 如果wrapper.get()表示从wrapper中取出value，那么取出后又塞进新的wrapper是没有意义的！

我们应该对value进行判断，当value不为null时**往下**取一层，这样才能最终一层层剥开value，得到最终的值：

![img](https://pic4.zhimg.com/80/v2-0f2bfdec6c2199bc208f69591b6962a7_720w.jpg)

```java
public <U> NullWrapper<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (value是否为null)
        // 如果为null，用Wrapper包装null，避免直接暴露导致空指针
        return new Wrapper(null);
    else {
        // 如果不为null，那么调用传入的映射规则，剥掉一层嵌套，把下一级取出来重新包装为Wrapper
        return new Wrapper(mapper.apply(value));
    }
}
```

由于不论value是否为null，最终返回的都是Wrapper，而Wrapper有map()，所以这里可以形成链式调用：

![img](https://pic1.zhimg.com/80/v2-38a8e1b2b928342c2baa1728dad82584_720w.jpg)

也就是说，每一次map()其实都**有可能**剥掉一层嵌套，而且过程中不会发生空指针异常！

```java
// 初始化包裹，得到Wrapper
Wrapper wrapper = new Wrapper(firstLevelValue);
// 调用Wrapper#map()尝试向下剥开每一级的嵌套，由于map()返回的也是Wrapper对象，可以链式调用
wrapper
.map(firstLevelValue -> firstLevelValue.getSecondLevelValue)
.map(secondLevelValue -> secondLevelValue.getThirdLevelValue)
.map(...)
```

为什么是**有可能？**

我们来考虑两种极端的情况：

- 如果每一级value**都为null，**其实每次map()都是对null进行包装传递、取出、再包装，并没有剥开任何嵌套（null值不存在嵌套）
- 如果每一级value**都不为null，**每次map()都剥开一层，这是最理想的效果，离最终需要的value越来越近

第一种情况，其实也没什么，无非就是多new几个Wrapper对象（似乎有点浪费？）。第二种情况，符合我们的预期，但也不能每次剥开又给套上Wrapper吧，丑媳妇最终还是要见公婆。所以，除了map()方法，Wrapper还需要额外提供一个最终获取**真实value**的方法，比如Wrapper#orElse(T other)，它允许调用者得到未包装的value，但是！有个条件：调用orElse()时必须传一个备用的值，如果value真的为null，则返回备用值代替null（但你如果作死，备用值传null，那也没办法）。

上面讲述的就是Wrapper工具类的核心思想，接着让我们一起来封装一下！



**工具类封装**

```java
/**
 * 工具类，用于简化空指针的探测工作
 * 核心思想：将可能为null的value包装成NullWrapper对象，那么调用nullWrapper.xxx()方法肯定就不会发生NullPointerException
 * 核心方法：map()
 * <p>
 * map()方法由NullWrapper对象调用，实际操作的是nullWrapper内部的value，将value映射为指定的值（比如某个字段）
 * 如果value为null，调用empty()方法，返回包装了null的NullWrapper对象
 * 如果value不为null，执行mapper.apply(value)得到结果并调用ofNullable(T newValue)方法，返回包装了newValue的NullWrapper对象
 *
 * @param <T>
 */
public final class NullWrapper<T> {
    /**
     * 配合{@link NullWrapper#empty()}使用，返回一个包装了null的NullWrapper
     */
    private static final NullWrapper<?> EMPTY = new NullWrapper<>();

    /**
     * 实际值
     */
    private final T value;

    /**
     * 构造器，包装null
     */
    private NullWrapper() {
        this.value = null;
    }

    /**
     * 构造器，包装指定的【非空值】
     * 如果传入null会抛异常NullPointerException
     *
     * @param value
     */
    private NullWrapper(T value) {
        this.value = Objects.requireNonNull(value);
    }

    /**
     * 静态方法，返回一个包装了null的NullWrapper
     *
     * @param <T>
     * @return
     */
    public static <T> NullWrapper<T> empty() {
        @SuppressWarnings("unchecked")
        NullWrapper<T> t = (NullWrapper<T>) EMPTY;
        return t;
    }

    /**
     * 静态方法，包装指定的【非空值】
     * 如果传入null会抛异常NullPointerException
     *
     * @param value
     * @param <T>
     * @return
     */
    public static <T> NullWrapper<T> of(T value) {
        return new NullWrapper<>(value);
    }

    /**
     * 静态方法，包装指定值，允许null。当传入null时，会调用empty()方法返回EMPTY对象
     *
     * @param value
     * @param <T>
     * @return
     */
    public static <T> NullWrapper<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }

    /**
     * 是否非null
     *
     * @return
     */
    public boolean isPresent() {
        return value != null;
    }

    /**
     * 核心方法
     * 调用者：NullWrapper对象A，内部含有value，可能为null，也可能不为null
     * 如果value为null，调用empty()方法，返回包装了null的NullWrapper对象
     * 如果value不为null，执行mapper.apply(value)得到结果并调用ofNullable(T newValue)方法，返回包装了newValue的NullWrapper对象
     * 简而言之，每次调用map()，都会剥掉一层外壳，也意味着躲过了一次潜在的NullPointerException
     *
     * @param mapper
     * @param <U>
     * @return
     */
    public <U> NullWrapper<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return NullWrapper.ofNullable(mapper.apply(value));
        }
    }

    /**
     * 终端操作，当最终结果还是为null时，返回默认值other
     *
     * @param other
     * @return
     */
    public T orElse(T other) {
        return value != null ? value : other;
    }

    // ---------- 对Object的方法重写，不用关注 -----------

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof NullWrapper)) {
            return false;
        }
        NullWrapper<?> other = (NullWrapper<?>) obj;
        return Objects.equals(value, other.value);
    }

    @Override
    public int hashCode() {
        return Objects.hashCode(value);
    }

    @Override
    public String toString() {
        return value != null
                ? String.format("Optional[%s]", value)
                : "Optional.empty";
    }
}
```

不知道大家有没有发现一个小心思：

> 对于连续为null的情况，其实并不会每次都重新new Wrapper()，而是直接调用empty()返回EMPTY，也就是**空Wrapper复用。**

**Demo重构**

```java
/**
 * 假设这是A-Service的服务
 *
 * @param username
 * @return
 */
public static String getDepartmentNameOfUser(String username) {
    // 请求远程方法
    ResultTO<User> resultTO = getUserByName(username);
    // 包装为Wrapper
    NullWrapper<ResultTO<User>> resultTONullWrapper = NullWrapper.ofNullable(resultTO);
    // 链式包装
    return resultTONullWrapper
            .map(ResultTO::getData)
            .map(User::getDepartment)
            .map(Department::getName)
            .orElse("未知部门");
}
```

简写成：

```java
/**
 * 假设这是A-Service的服务
 *
 * @param username
 * @return
 */
public static String getDepartmentNameOfUser(String username) {
    return NullWrapper.ofNullable(getUserByName(username))
            .map(ResultTO::getData)
            .map(User::getDepartment)
            .map(Department::getName)
            .orElse("未知部门");
}
```

是不是很简洁、很优雅呢~

**Optional**

其实上面的工具类就是模仿Java8 Optional写的，也已经演示了Optional最最核心的用法，所以这里不打算再对Optional进行长篇大论。

**Optional不难，难的是很多人不知道什么时候该用Optional、怎么用Optional。**



**常见错误写法**

举几个常见的滥用Optional的案例：

```java
public static String getDepartmentNameOfUser(String username) {
    
    // 错误用法1，这样还是会出现空指针，因为get()后返回的就是裸露的值
    String result1 = Optional.ofNullable(getUserByName("test")).get().getData().getDepartment().getName();

    /**
     * 错误用法2，把isPresent()当做判断空指针的方法，又回归以前的if嵌套，毫无意义
     * 个人觉得isPresent()根本不应该暴露出来，只有在Optional内部会使用，普通调用者根本不需要
     */
    Optional<ResultTO<User>> result2 = Optional.ofNullable(getUserByName("test"));
    if (result2.isPresent()) {
        User user = result2.get().getData();
        if (user != null) {
            // ...继续if判断空指针
        }
    }
}
```

应该把Optional当做一个简单的工具类，仿佛是你身边的同事封装的，不要对它抱有过分的期待，它的使命就一个：简化冗余的空指针探测。

**API梳理**

这里帮大家梳理一下Optional的API，标出几个重点且适合我们使用的方法，其他的都可以不理会，因为真的没用：



![img](https://pic2.zhimg.com/80/v2-07c3e006eb508df9f3f88dd9d887a925_720w.jpg)

最重要的就是学会3个方法：

- 如何包装value：Optional.ofNullable()
- 逐层安全地拆解value：map()
- 最终返回：orElse()

**这三个搭配就是最佳实践。**

其他可能会用的方法就是orElseGet()、orElsethrow()，其他的不常用或者说不好用。