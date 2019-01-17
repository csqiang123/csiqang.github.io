# Date, DateFormat

## 1.Date类方法

​	以毫秒作为标记,即距离1970年1月1日00:00:00所经过的毫秒值,再通过毫秒值计算出对应的年、月、日、时、分、秒、星期等时间信息 

A：构造方法：

​    public Date()                     //返回当前时间

​    public Date(long date)   //返回指定毫秒值的日期对象

 B：普通方法：

   public long getTime()   //获取当前时间对象的毫秒值

   public void setTime(long time)  //设置时间毫秒值



## 2.DateFormat

​	DateFormat 是日期/时间格式化子类的抽象类，它以与语言无关的方式格式化并解析日期或时间。

​	日期/时间格式化子类（如SimpleDateFormat类）允许进行格式化（也就是日期 -> 文本）、解析（文本-> 日期）和标准化。



 DateFormat:日期格式化类

SimpleDateFormat:实际使用的日期格式化子类

​	在创建SimpleDateFormat对象时,可以指定生成字符串的模板,规则见API帮助文档

​	模板是一个字符串,代表转换规则:特殊字母代表一定的时间组成部分

public final String format(Date date)  将日期格式化成字符串

例子1:

​	//定义日期规则字符串

​       String rule = "yyyy年MM月dd日HH:mm:ss";

​       //使用日期转换规则,创建日期格式化类对象

​       DateFormat format = **new** SimpleDateFormat(rule);

​       //准备要转换的日期对象

​       Date dNow = **new** Date();

​       //转换Date对象为String字符串

​       String sNow = format.format(dNow);



例子2:

//定义日期规则字符串

​       String rule = "yyyy年MM月dd日HH:mm:ss";

​       //创建日期格式化类对象

​       DateFormat format = **new** SimpleDateFormat(rule);

​       //准备日期字符串

​       String sTime = "2017年11月06日  14:10:28";

​       //转换生成Date对象

​       Date dTime = format.parse(sTime);

​       //打印Date对象

​       System.**out**.println(dTime);

​    }

