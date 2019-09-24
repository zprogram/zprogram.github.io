## calendar日历模块

在python中与日期时间相关的模块主要有4个：
```
	calendar 	日历模块
	time		时间模块
	datetime  	日期时间模块（综合模块）
	timeit   	时间检测模块
```

### 日历模块


calendar() 获取一年的日历字符串
```
	格式：calendar.calendar(年份,w=2,l=1,c=6)
	返回值：整年的日历字符串
	
	参数： 年份:整数
	
		w = 每个日期之间的间隔字符数
		l = 每周所占用的函数
		c = 每个月之间的间隔字符数
```
isleap() 检测是否是闰年
```
	格式：calendar.isleap(年份)
	返回值：布尔值
```

leapdays() 获取指定年份之间闰年的个数
```
	格式:calendar.leapdays(开始年份，结束年份)
	返回值：整数
```
month（） 获取某个月的日历字符串
```
	格式:calendar.month(年，月)
	返回值：月日历的字符串
```
monthrange（） 获取一个月的周几开始即总天数
```
	格式：calendar.monthrange(年,月)
	返回值：元组(周几开始,总天数)
	
	注意：周默认 0 -6 表示周一到周天
```
monthcalendar() 返回一个月每天的矩阵列表
```
	格式：calendar.monthcalendar(年，月)
	返回值：二级列表
	
	注意：矩阵中没有天数用0表示
```
prcal() 直接打印整年的日历
```
	格式：calendar.prcal(年份)
	返回值：无
```
prmonth() 直接打印整个月的日历
```
	格式：calendar.prmonth(年，月)
	返回值：无
```
firstweekday() 获取每周开始的天数
```
	格式：calendar.firstweekday()
	返回值： 0 - 6  默认数值，在不改变的情况
```
setfirstweekday() 设置每周开始的天数
```
	格式：calendar.setfirstweekday(开始值)
	返回值：无
```
weekday() 获取周几
```
	格式:calendar.weekday(年，月，日)
	返回值:周几对应的数字
```
timegm（）获取一个指定时间的时间戳
```
	格式：calendar.timegm(时间元组)
	返回值：时间戳
```
