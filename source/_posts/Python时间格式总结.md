---
title: Python时间格式总结
date: 2020-09-20 23:14:57
tags:
	- Python
---



## time模块

1. time模块下时间主要有三种表现形式：

   1. 时间戳(timestamp)；
   2. 时间元组(struct_time);
   3. 格式化时间，格式化时间还包括自定义格式和固定格式；

2. 以上表现形式生成和相互转换的方式：

   |            | 时间戳(ts)      | 时间元组(st)                          | 自定义格式(ft)         | 固定格式         |
   | ---------- | --------------- | ------------------------------------- | ---------------------- | ---------------- |
   | 时间戳     | time.time()     | time.localtime(ts)<br>time.gmtime(ts) | NA                     | time.ctime(ts)   |
   | 时间元组   | time.mktime(st) | time.localtime()<br>time.gmtime()     | time.strptime(fmt, st) | time.asctime(st) |
   | 自定义格式 | NA              | NA                                    | time.strftime(fmt)     | NA               |
   | 固定格式   | NA              | NA                                    | NA                     | NA               |

   

## datetime.date类

1. date类，表示日期的类：
   1. 创建方法：datetime.date(year, month, day)
   2. 主要属性：year，month，day
   3. 主要方法：
      1. date.timetuple(date) —— 返回日期的时间元组；
      2. date.replace(year, month, day) —— 替换date对象中的year/month/day，并返回替换后的date对象；
      3. date.weekday(date) —— 返回日期对应的星期，星期一返回0，星期二返回1，以此类推；
      4. date.isoweekday(date) —— 返回日期对应的星期，星期一返回1，星期二返回2，以此类推；
      5. date.iscalendar(date) —— 返回日期对应的元组 ；
      6. date.isoformat(date) —— 返回日期对应格式的字符串；
      7. date.fromtimestamp(timestamp) —— 根据给定的时间戳返回一个date对象；
      8. date.today() —— 返回当地日期的date对象，注意这里返回的不是timestamp对象；

## datetime.time类

1. time类，表示时间的类：
   1. 创建方法：datetime.time(hour, minute, second, microsecond, tzinfo)
   2. 主要属性：hour, minute, second, microsecond, tzinfo
   3. 主要方法：
      1. time.replace(hour, minute, second, microsecond, tzinfo) —— 替换time对象中的时分秒微秒和时区，并返回替换后的time对象；
      2. time.isofomr(time)，根据time对象返回 hour:minute:second格式的时间；
      3. time.strftime(fmt)，根据指定的格式，返回time对象的自定义格式化时间的字符串；

## datetime.datetime类

1. datetime类，饱含date和time的所有信息。
2. 主要属性：year、month、day、hour、minute、second、microsecond、tzinfo；
3. 主要方法：
   1. datetime.date()/time()，获取date/time对象；
   2. datetime.today()，获取表示当前本地时间的datetime对象；
   3. datetime.now(tz)，获取表示当前本地时间的datetime对象，如果提供时区参数tz，则获取指定时区的datetime对象；
   4. datetime.utcnow()，获取表示当前utc时间的datetime对象；
   5. datetime.fromtimestamp(timestamp, tz)，根据时间戳创建一个datetime对象，如果指定了tz参数，则返回指定时区的对象；
   6. datetime.utcfromtimestamp(timestamp)，根据时间戳创建一个datetime对象；
   7. datetime.combine(date, time)，根据date和time对象创建一个datetime对象；
   8. datetime.strptime(date_string, format)，将格式字符串转换为datetime对象；
   9. datetime.replace(year, month, day, hour, minute, second, microsecond, tzinfo)，替代datetime中的年月日时分秒微秒和时区，并返回替换后的datetime对象；
   10. datetime.timetuple(datetime)，根据datetime对象，返回对应的时间元组；
   11. datetime.utctimetuple(datetime)，根据datetime对象，返回对应的utc时间的时间元组；
   12. datetime.weekday(datetime)，返回datetime对象所对应的星期，星期一对应于0，星期二对应1，以此类推；
   13. datetime.isoweekday(datetime)，返回datetime对象所对应的星期，星期一对应于1，星期二对应2，以此类推；
   14. datetime.isocalendar (datetime)，返回datetime对象说对应的iso日历；
   15. datetime.isoformat(datetime)，返回datetime对象所对应的iso格式的时间；
   16. datetime.ctime()，返回datetime对象所对应的c格式时间的字符串；
   17. datetime.strftime(format)，返回datetime对象所对应的指定格式时间的字符串；

### 参考文献：

https://www.cnblogs.com/tkqasn/p/6001134.html

https://my.oschina.net/whp/blog/130710