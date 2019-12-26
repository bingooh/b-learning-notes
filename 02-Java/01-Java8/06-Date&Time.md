## 旧版日期
`java.util.Date/Calendar`缺陷：
- 类的对象是可变的，线程不安全
- 年份以`1990`开始,月份以`0`开始
- `DateFormat`类线程不安全

## 新版日期
`java.time`提供了新版日期：
- 类的对象不可变，调用相关方法**总是返回1个新的实例**

#### 新版日期主要接口和类
- `TemporalAmount` 定义两个时间点之间的时间段，如`2 hours`
    - `Duration` `time-based`时间总和，如`34.5 seconds`
    - `Period`   `date-based`时间总和，如`2 years,3 monthes and 4 days`
- `Temporal` 定义1个(暂存的)时间点
    - `Instant`        机器使用的`time-line`上的1个时间点
    - `LocalTime`      人类可读的1个时间点，不含时区
    - `LocalDate`      人类可读的1个日期点，不含时区
    - `LocalDateTime`  人类可读的1个日期时间点，不含时区
    - `ZonedDateTime`  人类可读的1个时间点，包含时区
    - `Year`           人类可读的1个年时间点，如`2017`
    - `YearMonth`      人类可读的1个年月时间点，如`2017-01`
    - `OffsetTime`     人类可读的1个偏移时间点，如`10:15:30+01:00`
    - `OffsetDateTime` 人类可读的1个偏移日期时间点，如`2007-12-03T10:15:30+01:00` 
- `TemporalAdjuster` 定义调整时间点的策略接口
    - `TemporalAdjusters` 工具类 
- `ZoneId`      定义时区，使用`zh/cn`
    - `ZoneOffset`  定义时区，使用`UTC(+00:00)`偏移量定义时区
- `TemporalField` 定义时间字段，如`day-of-month`
    - `ChronoField`，常用时间字段枚举类
- `TemporalUnit` 定义时间单位，如`days`
    - `ChronoUnit` 常用时间单位枚举类
- `ValueRange` 所有`TemporalField`都有1个有效值范围，如`hour_of_day`的有效值范围为`0-23`
- `DateTimeFormatter` 格式化，线程安全
    - `DateTimeFormatterBuilder` 帮助类 

#### 年历(Chronology)
- 年历规定了元年起始，月份划分等
- 国际年历使用`ISO-8601`,如`2017-01-01`
- `Chronology` 定义年历接口
- `ChronoLocalDate` 定义使用那种年历
    - `LocalDate`  使用`ISO-8601`
    - `HijrahDate` 使用`Hijrah`
- `ChronoLocaltDateTime`
    - `LocalDateTime`  使用`ISO-8601` 
- `ChronoZonedDateTime`
    - `ZonedDateTime`  使用`ISO-8601`

## 时间段
- `Duration`底层存储为秒，存储格式同`Instant`
- `Duration.get()`仅支持`SECONDS,NANOS`(底层存储为秒)
- `Duration.of(3,ChronoUnit.MINUTES)` 3分钟时间段
- `Duration.between(t1,t2)` `t1-t2`时间段
- `Duration.from(period)`   转换`period`为`duration`
- `Period.get()`仅支持`YEARS,MONTHS,DAYS`
- `Period.ofDays(2)`   2天时间段
- `Period.withDays(2)` 复制当前时间段，使用指定的天数
- `Period.addTo(t1)`   把当前时间段加到`t1`时间点

## 时间点
#### `Temporal`
- `Temporal.isSupported(field)` 是否支持传入的`TemporalField`
    - 调用`plus(),range(),with()`等，如果传入不支持的`TemporalField`将抛错
- `Temporal.plus()` 增加指定的时间段
- `Temporal.with(adjuster)` 调整到指定的时间点
- `Temporal.util(t2)` 计算两个时间点之间的时间段

#### `Instant`
- 使用`long`保存秒数，精确解析到纳秒(1秒=10E9纳秒)
- 以`1970-01-01T00:00:00Z`为原点
    - 大于此原点的时间点使用整数
    - 小于此原点的时间点使用负数
- `Instant`仅支持秒相关字段
- `Instant.ofEpochSecond(1,10)`   `1秒+10纳秒`
- `Instant.ofEpochSecond(-1,10)` `-1秒+10纳秒`

#### 举例
- 互相转换
```
LocalTime time=LocalTime.now();
LocalDate date=LocalDate.now();

//转换为LocalDateTime
LocalDateTime dateTime=time.atDate(date);
LocalDateTime dateTime1=date.atTime(time);
LocalDateTime dateTime2=LocalDateTime.of(date,time);

//转换为LocalDate，LocalTime
LocalTime time1=dateTime.toLocalTime();
LocalDate date1=dateTime.toLocalDate();

//与Instant互相转换
Instant instant=Instant.from(dateTime);
LocalDateTime dateTime3=LocalDateTime.from(instant);
```

- 计算时间
```
LocalTime time1=...;
LocalTime time2=...;

Instant instant=Instant.between(time1,time2);

LocalTime time3=instant.addTo(time1);
LocalTime time3=instant.subtractFrom(time1);
```
- 使用时区
```
ZoneId zone=ZoneId.of("zh/cn");
ZoneId zone1=systemDefault();
ZoneId zone2=TimeZone.getDefault().toZoneId();//TimeZone为旧API

ZonedDateTime z1=localDate.atStartOfDay(zone);
ZonedDateTime z2=localDateTime.atZone(zone);
ZonedDateTime z3=instant.atZone(zone);

//传入时区计算偏移量，如果不需要计算时区偏移量可使用from()
Instant instant=localDateTime.toInstant(zone);
LocalDateTime localDateTime=LocalDateTime.ofInstant(instant,zone);

//使用时区偏移量
ZoneOffset cnZone=ZoneOffset.of("+08:00");
OffsetDateTime dateTime=OffsetDateTime.of(localDateTime,newYorkOffset);
```

- 使用`TemporalAdjuster`
```
import static java.time.temporal.TemporalAdjusters.*;

LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));
LocalDate date3 = date2.with(lastDayOfMonth());
```