在JDK8之前，处理日期时间，我们主要使用3个类，`Date`、`SimpleDateFormat`和`Calendar`。

这3个类在使用时都或多或少的存在一些问题，比如`SimpleDateFormat`不是线程安全的，

比如`Date`和`Calendar`获取到的月份是0到11，而不是现实生活中的1到12，关于这一点，《阿里巴巴Java开发手册》中也有提及，因为很容易犯错：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-97e6026bc294ab21164b22f09bd49378_720w.jpg)


不过，JDK8推出了全新的日期时间处理类解决了这些问题，比如`Instant`、`LocalDate`、`LocalTime`、`LocalDateTime`、`DateTimeFormatter`，在《阿里巴巴Java开发手册》中也推荐使用`Instant`、

`LocalDateTime`、`DateTimeFormatter`：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-acec27561dbef056eb8a31e6a2aaf7ef_720w.jpg)

但我发现好多项目中其实并没有使用这些类，使用的还是之前的`Date`、`SimpleDateFormat`和`Calendar`，所以本篇博客就讲解下JDK8新推出的日期时间类，主要是下面几个：

1. Instant
2. LocalDate
3. LocalTime
4. LocalDateTime
5. DateTimeFormatter

> 

## 1. Instant

### 1.1 获取当前时间

既然`Instant`可以代替`Date`类，那它肯定可以获取当前时间：

```java
Instant instant = Instant.now();
System.out.println(instant);
```

输出结果：

>  2020-06-10T08:22:13.759Z

细心的你会发现，这个时间比北京时间少了8个小时，如果要输出北京时间，可以加上默认时区：

```java
System.out.println(instant.atZone(ZoneId.systemDefault()));
```

输出结果：

>  2020-06-10T16:22:13.759+08:00[Asia/Shanghai]

### 1.2 获取时间戳

```text
Instant instant = Instant.now();

// 当前时间戳:单位为秒
System.out.println(instant.getEpochSecond());
// 当前时间戳:单位为毫秒
System.out.println(instant.toEpochMilli());
```

输出结果：

>  1591777752
>  1591777752613

当然，也可以通过`System.currentTimeMillis()`获取当前毫秒数。

### 1.3 将long转换为Instant

1)根据秒数时间戳转换：

```text
Instant instant = Instant.now();
System.out.println(instant);

long epochSecond = instant.getEpochSecond();
System.out.println(Instant.ofEpochSecond(epochSecond));
System.out.println(Instant.ofEpochSecond(epochSecond, instant.getNano()));
```

输出结果：

>  2020-06-10T08:40:54.046Z
>  2020-06-10T08:40:54Z
>  2020-06-10T08:40:54.046Z

2)根据毫秒数时间戳转换：

```java
Instant instant = Instant.now();
System.out.println(instant);

long epochMilli = instant.toEpochMilli();
System.out.println(Instant.ofEpochMilli(epochMilli));
```

输出结果：

>  2020-06-10T08:43:25.607Z
>  2020-06-10T08:43:25.607Z

### 1.4 将String转换为Instant

```java
String text = "2020-06-10T08:46:55.967Z";
Instant parseInstant = Instant.parse(text);
System.out.println("秒时间戳:" + parseInstant.getEpochSecond());
System.out.println("豪秒时间戳:" + parseInstant.toEpochMilli());
System.out.println("纳秒:" + parseInstant.getNano());
```

输出结果：

>  秒时间戳:1591778815
>  豪秒时间戳:1591778815967
>  纳秒:967000000

如果字符串格式不对，比如修改成`2020-06-10T08:46:55.967`，就会抛出`java.time.format.DateTimeParseException`异常，如下图所示：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-981f0a5b6f0f726cdd2de44dcb10d40a_720w.png)

##  2. LocalDate

### 2.1 获取当前日期

使用`LocalDate`获取当前日期非常简单，如下所示：

```text
LocalDate today = LocalDate.now();
System.out.println("today: " + today);
```

输出结果：

>  today: 2020-06-10

不用任何格式化，输出结果就非常友好，如果使用`Date`，输出这样的格式，还得配合`SimpleDateFormat`指定`yyyy-MM-dd`进行格式化，一不小心还会出个bug，比如去年年底很火的1个bug，我当时还是截了图的：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-1bdfa49dc8fe498730f563b8b3cd5747_720w.jpg)

这2个好友是2019/12/31关注我的，但我2020年1月2号查看时，却显示成了2020/12/31，为啥呢？格式化日期时格式写错了，应该是`yyyy/MM/dd`，却写成了`YYYY/MM/dd`，刚好那周跨年，就显示成下一年，也就是2020年了，当时好几个博主写过文章解析原因，我这里就不做过多解释了。

### 2.2 获取年月日

```text
LocalDate today = LocalDate.now();

int year = today.getYear();
int month = today.getMonthValue();
int day = today.getDayOfMonth();

System.out.println("year: " + year);
System.out.println("month: " + month);
System.out.println("day: " + day);
```

输出结果：

>  year: 2020
>  month: 6
>  day: 10

获取月份终于返回1到12了，不像`java.util.Calendar`获取月份返回的是0到11，获取完还得加1。

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-da85767d1025662cd871bad212e925a4_720w.jpg)

### 2.3 指定日期

```text
LocalDate specifiedDate = LocalDate.of(2020, 6, 1);
System.out.println("specifiedDate: " + specifiedDate);
```

输出结果：

>  specifiedDate: 2020-06-01

如果确定月份，推荐使用另一个重载方法，使用枚举指定月份：

```text
LocalDate specifiedDate = LocalDate.of(2020, Month.JUNE, 1);
```

### 2.4 比较日期是否相等

```text
LocalDate localDate1 = LocalDate.now();
LocalDate localDate2 = LocalDate.of(2020, 6, 10);
if (localDate1.equals(localDate2)) {
    System.out.println("localDate1 equals localDate2");
}
```

输出结果：

>  localDate1 equals localDate2

### 2.5 获取日期是本周/本月/本年的第几天

```text
LocalDate today = LocalDate.now();

System.out.println("Today:" + today);
System.out.println("Today is:" + today.getDayOfWeek());
System.out.println("今天是本周的第" + today.getDayOfWeek().getValue() + "天");
System.out.println("今天是本月的第" + today.getDayOfMonth() + "天");
System.out.println("今天是本年的第" + today.getDayOfYear() + "天");
```

输出结果：

>  Today:2020-06-11
>  Today is:THURSDAY
>  今天是本周的第4天
>  今天是本月的第11天
>  今天是本年的第163天

### 2.6 判断是否为闰年

```text
LocalDate today = LocalDate.now();

System.out.println(today.getYear() + " is leap year:" + today.isLeapYear());
```

输出结果：

>  2020 is leap year:true

## 3. LocalTime

### 3.1 获取时分秒

如果使用`java.util.Date`，那代码是下面这样的：

```text
Date date = new Date();

int hour = date.getHours();
int minute = date.getMinutes();
int second = date.getSeconds();

System.out.println("hour: " + hour);
System.out.println("minute: " + minute);
System.out.println("second: " + second);
```

输出结果：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-e97944691aa7b2eb31e1834f471b478c_720w.jpg)


**注意事项：这几个方法已经过期了，因此强烈不建议在项目中使用：**

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-27b9c212e5694e5771f2f8f5dbab7137_720w.jpg)


如果使用`java.util.Calendar`，那代码是下面这样的：

```text
Calendar calendar = Calendar.getInstance();

// 12小时制
int hourOf12 = calendar.get(Calendar.HOUR);
// 24小时制
int hourOf24 = calendar.get(Calendar.HOUR_OF_DAY);
int minute = calendar.get(Calendar.MINUTE);
int second = calendar.get(Calendar.SECOND);
int milliSecond = calendar.get(Calendar.MILLISECOND);

System.out.println("hourOf12: " + hourOf12);
System.out.println("hourOf24: " + hourOf24);
System.out.println("minute: " + minute);
System.out.println("second: " + second);
System.out.println("milliSecond: " + milliSecond);
```

输出结果：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-2c2b36fa0dd67748a4908cd87f4ec3ef_720w.jpg)


**注意事项：**获取小时时，有2个选项，1个返回12小时制的小时数，1个返回24小时制的小时数，因为现在是晚上8点，所以`calendar.get(Calendar.HOUR)`返回8，而`calendar.get(Calendar.HOUR_OF_DAY)`返回20。

如果使用`java.time.LocalTime`，那代码是下面这样的：

```text
LocalTime localTime = LocalTime.now();
System.out.println("localTime:" + localTime);

int hour = localTime.getHour();
int minute = localTime.getMinute();
int second = localTime.getSecond();

System.out.println("hour: " + hour);
System.out.println("minute: " + minute);
System.out.println("second: " + second);
```

输出结果：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-a1c0bbbf1c2b3acbbfba1cb551da99d8_720w.jpg)

可以看出，LocalTime只有时间没有日期。

## 4. LocalDateTime

### 4.1 获取当前时间

```text
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println("localDateTime:" + localDateTime);
```

输出结果：

>  localDateTime: 2020-06-11T11:03:21.376

### 4.2 获取年月日时分秒

```text
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println("localDateTime: " + localDateTime);

System.out.println("year: " + localDateTime.getYear());
System.out.println("month: " + localDateTime.getMonthValue());
System.out.println("day: " + localDateTime.getDayOfMonth());
System.out.println("hour: " + localDateTime.getHour());
System.out.println("minute: " + localDateTime.getMinute());
System.out.println("second: " + localDateTime.getSecond());
```

输出结果：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-3dae40b0616c7beb5c9fffadd83d4319_720w.jpg)

###  4.3 增加天数/小时

```text
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println("localDateTime: " + localDateTime);

LocalDateTime tomorrow = localDateTime.plusDays(1);
System.out.println("tomorrow: " + tomorrow);

LocalDateTime nextHour = localDateTime.plusHours(1);
System.out.println("nextHour: " + nextHour);
```

输出结果：

>  localDateTime: 2020-06-11T11:13:44.979
>  tomorrow: 2020-06-12T11:13:44.979
>  nextHour: 2020-06-11T12:13:44.979

`LocalDateTime`还提供了添加年、周、分钟、秒这些方法，这里就不一一列举了：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-2e8115ee4ae1c2361f14dc6b27a632b0_720w.jpg)

### 4.4 减少天数/小时

```text
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println("localDateTime: " + localDateTime);

LocalDateTime yesterday = localDateTime.minusDays(1);
System.out.println("yesterday: " + yesterday);

LocalDateTime lastHour = localDateTime.minusHours(1);
System.out.println("lastHour: " + lastHour);
```

输出结果：

>  localDateTime: 2020-06-11T11:20:38.896
>  yesterday: 2020-06-10T11:20:38.896
>  lastHour: 2020-06-11T10:20:38.896

类似的，`LocalDateTime`还提供了减少年、周、分钟、秒这些方法，这里就不一一列举了：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-f1e519701766d01b568e3e0d8bcf5c14_720w.jpg)

### 4.5 获取时间是本周/本年的第几天

```text
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println("localDateTime: " + localDateTime);

System.out.println("DayOfWeek: " + localDateTime.getDayOfWeek().getValue());
System.out.println("DayOfYear: " + localDateTime.getDayOfYear());
```

输出结果：

>  localDateTime: 2020-06-11T11:32:31.731
>  DayOfWeek: 4
>  DayOfYear: 163

## 5. DateTimeFormatter

```
//自定义格式转LocalDateTime
ateTimeFormatter dateTimeFormatter=DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime parse = LocalDateTime.parse("2018-05-29 13:52:50", dateTimeFormatter);

ateTimeFormatter dateTimeFormatter=DateTimeFormatter.ofPattern("yyyy-MM-dd");
LocalDate parse = LocalDate.parse("2018-05-29", dateTimeFormatter);
```

JDK8中推出了`java.time.format.DateTimeFormatter`来处理日期格式化问题，《阿里巴巴Java开发手册》中也是建议使用`DateTimeFormatter`代替`SimpleDateFormat`。

### 5.1 格式化LocalDate

```text
LocalDate localDate = LocalDate.now();

System.out.println("ISO_DATE: " + localDate.format(DateTimeFormatter.ISO_DATE));
System.out.println("BASIC_ISO_DATE: " + localDate.format(DateTimeFormatter.BASIC_ISO_DATE));
System.out.println("ISO_WEEK_DATE: " + localDate.format(DateTimeFormatter.ISO_WEEK_DATE));
System.out.println("ISO_ORDINAL_DATE: " + localDate.format(DateTimeFormatter.ISO_ORDINAL_DATE));
```

输出结果：

![img](C:\Users\tzq\Desktop\Java整理\jdk8日期类\v2-fd539cfa8060acd051f7c43012c100f7_720w.jpg)

如果提供的格式无法满足你的需求，你还可以像以前一样自定义格式：

```text
LocalDate localDate = LocalDate.now();

System.out.println("yyyy/MM/dd: " + localDate.format(DateTimeFormatter.ofPattern("yyyy/MM/dd")));
```

输出结果：

>  yyyy/MM/dd: 2020/06/11

### 5.2 格式化LocalTime

```text
LocalTime localTime = LocalTime.now();
System.out.println(localTime);
System.out.println("ISO_TIME: " + localTime.format(DateTimeFormatter.ISO_TIME));
System.out.println("HH:mm:ss: " + localTime.format(DateTimeFormatter.ofPattern("HH:mm:ss")));
```

输出结果：

>  14:28:35.230
>  ISO_TIME: 14:28:35.23
>  HH:mm:ss: 14:28:35

### 5.3 格式化LocalDateTime

```text
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println(localDateTime);
System.out.println("ISO_DATE_TIME: " + localDateTime.format(DateTimeFormatter.ISO_DATE_TIME));
System.out.println("ISO_DATE: " + localDateTime.format(DateTimeFormatter.ISO_DATE));

//自定义格式转LocalDateTime
ateTimeFormatter dateTimeFormatter=DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime parse = LocalDateTime.parse("2018-05-29 13:52:50", dateTimeFormatter);
```

输出结果：

>  2020-06-11T14:33:18.303
>  ISO_DATE_TIME: 2020-06-11T14:33:18.303
>  ISO_DATE: 2020-06-11

## 6. 类型相互转换

### 6.1 Instant转Date

JDK8中，`Date`新增了`from()`方法，将`Instant`转换为`Date`，代码如下所示：

```text
Instant instant = Instant.now();
System.out.println(instant);

Date dateFromInstant = Date.from(instant);
System.out.println(dateFromInstant);
```

输出结果：

>  2020-06-11T06:39:34.979Z
>  Thu Jun 11 14:39:34 CST 2020

### 6.2 Date转Instant

JDK8中，`Date`新增了`toInstant`方法，将`Date`转换为`Instant`，代码如下所示：

```text
Date date = new Date();
Instant dateToInstant = date.toInstant();
System.out.println(date);
System.out.println(dateToInstant);
```

输出结果：

>  Thu Jun 11 14:46:12 CST 2020
>  2020-06-11T06:46:12.112Z

### 6.3 Date转LocalDateTime

```text
Date date = new Date();
Instant instant = date.toInstant();
LocalDateTime localDateTimeOfInstant = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
System.out.println(date);
System.out.println(localDateTimeOfInstant);
```

输出结果：

>  Thu Jun 11 14:51:07 CST 2020
>  2020-06-11T14:51:07.904

### 6.4 Date转LocalDate

```text
Date date = new Date();
Instant instant = date.toInstant();
LocalDateTime localDateTimeOfInstant = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
LocalDate localDate = localDateTimeOfInstant.toLocalDate();
System.out.println(date);
System.out.println(localDate);
```

输出结果：

>  Thu Jun 11 14:59:38 CST 2020
>  2020-06-11

可以看出，`Date`是先转换为`Instant`，再转换为`LocalDateTime`，然后通过`LocalDateTime`获取`LocalDate`。

### 6.5 Date转LocalTime

```text
Date date = new Date();
Instant instant = date.toInstant();
LocalDateTime localDateTimeOfInstant = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
LocalTime toLocalTime = localDateTimeOfInstant.toLocalTime();
System.out.println(date);
System.out.println(toLocalTime);
```

输出结果：

>  Thu Jun 11 15:06:14 CST 2020
>  15:06:14.531

可以看出，`Date`是先转换为`Instant`，再转换为`LocalDateTime`，然后通过`LocalDateTime`获取`LocalTime`。

### 6.6 LocalDateTime转Date

```text
LocalDateTime localDateTime = LocalDateTime.now();

Instant toInstant = localDateTime.atZone(ZoneId.systemDefault()).toInstant();
Date dateFromInstant = Date.from(toInstant);
System.out.println(localDateTime);
System.out.println(dateFromInstant);
```

输出结果：

>  2020-06-11T15:12:11.600
>  Thu Jun 11 15:12:11 CST 2020

### 6.7 LocalDate转Date

```text
LocalDate today = LocalDate.now();

LocalDateTime localDateTime = localDate.atStartOfDay();
Instant toInstant = localDateTime.atZone(ZoneId.systemDefault()).toInstant();
Date dateFromLocalDate = Date.from(toInstant);
System.out.println(dateFromLocalDate);
```

输出结果：

>  Thu Jun 11 00:00:00 CST 2020

### 6.8 LocalTime转Date

```text
LocalDate localDate = LocalDate.now();
LocalTime localTime = LocalTime.now();

LocalDateTime localDateTime = LocalDateTime.of(localDate, localTime);
Instant instantFromLocalTime = localDateTime.atZone(ZoneId.systemDefault()).toInstant();
Date dateFromLocalTime = Date.from(instantFromLocalTime);

System.out.println(dateFromLocalTime);
```

输出结果：

>  Thu Jun 11 15:24:18 CST 2020

## 7. 总结

JDK8推出了全新的日期时间类，如`Instant`、`LocaleDate`、`LocalTime`、`LocalDateTime`、`DateTimeFormatter`，设计比之前更合理，也是线程安全的。

《阿里巴巴Java开发规范》中也推荐使用`Instant`代替`Date`，`LocalDateTime` 代替 `Calendar`，`DateTimeFormatter` 代替 `SimpleDateFormat`。

因此，如果条件允许，建议在项目中使用，没有使用的，可以考虑升级下。