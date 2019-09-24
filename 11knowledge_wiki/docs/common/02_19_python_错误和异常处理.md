#错误和异常处理

在python中只有2种错误：一种是语法错误，另一种就是异常

#语法错误

python中语法错误也叫做解析错误，语法不符合规定导致的错误，初学者常见！

#异常

异常是指在语法正确的前提下，运行时发生的错误！

在python中，语法正确是无法运行时所报的错误都是异常。

异常是一个对象，在程序的思维中。

所以异常时可以进行处理的！

#异常的分类（标准异常分类）

	AssertError	 		断言语句（assert）失败
	AttributeError	 		尝试访问未知的对象属性
	EOFError				用户输入文件末尾标志EOF（Ctrl+d）
	FloatingPointError	 	浮点计算错误
	GeneratorExit	 		generator.close()方法被调用的时候
	ImportError				导入模块失败的时候
	IndexError	 			索引超出序列的范围
	KeyError	 			字典中查找一个不存在的关键字
	KeyboardInterrupt	 	用户输入中断键（Ctrl+c）
	MemoryError	 			内存溢出（可通过删除对象释放内存）
	NameError	 			尝试访问一个不存在的变量
	NotImplementedError	 	尚未实现的方法
	OSError	 				操作系统产生的异常（例如打开一个不存在的文件）
	OverflowError	 		数值运算超出最大限制
	ReferenceError			弱引用（weak reference）试图访问一个已经被垃圾回收机制回收了的对象
	RuntimeError	 		一般的运行时错误
	StopIteration	 		迭代器没有更多的值
	SyntaxError	 			Python的语法错误
	IndentationError	 	缩进错误
	TabError	 			Tab和空格混合使用
	SystemError	 			Python编译器系统错误
	SystemExit	 			Python编译器进程被关闭
	TypeError				不同类型间的无效操作
	UnboundLocalError		访问一个未初始化的本地变量（NameError的子类）
	UnicodeError			Unicode相关的错误（ValueError的子类）
	UnicodeEncodeError		Unicode编码时的错误（UnicodeError的子类）
	UnicodeDecodeError		Unicode解码时的错误（UnicodeError的子类）
	UnicodeTranslateError	Unicode转换时的错误（UnicodeError的子类）
	ValueError				传入无效的参数
	ZeroDivisionError	 	除数为零

#常见的异常案例：

IndexError   
  
	list1 = ['冯巩','牛群','马三立','侯宝林']
	
	print(list1[30])#使用了不存在的索引值
	
KeyError

	dict1 = {'niu':'穹','zhu':'升','yang':'磊','ma':'星辉'}

	print(dict1['lv'])#使用了不存在的键



nameError

	print(diao)#使用了为定义的变量


ZeroDivisionError

	print(99/0)#被除数为0

KeyboardInterrupt
	while True:
    	print('大爷来玩啊～')

	#用于按ctrl+C终止shell执行的错误

AssertionError

	assert len([1,2,3,4]) > 1 #断言成功，不报错

	assert len([1,2,3,4]) > 10 #断言失败时被报错


#错误异常处理

语法格式：

	try:
		尝试实现某个操作，
		如果没出现异常，任务就可以完成
		如果出现异常，将异常从当前代码块扔出去尝试解决异常

	except 异常类型1:

		解决方案1：用于尝试在此处处理异常解决问题

	except 异常类型2：

		解决方案2：用于尝试在此处处理异常解决问题

	except (异常类型1,异常类型2...)

		解决方案：针对多个异常使用相同的处理方式

	excpet:

		解决方案：所有异常的解决方案

	else:

		如果没有出现任何异常，将会执行此处代码

	finally:

		管你有没有异常都要执行的代码



##try语法
	
1.无任何异常情况下的处理顺序

	1.执行try区域内的代码
	2.如果没有任何异常发生
	3.执行else区域的代码
	4.执行finally区域的代码

2.发生异常的处理顺序

	1.执行try区域内的代码
	2.如果出现异常，将异常从当前try区域抛出
	3.第一个【except 异常类型1】，会查询抛出的异常是否和我指定的类型一致
		如果异常和的类型一致，则进入当前except区域进行处理
		如果异常和类型不一致，则跳过当前except区域将异常送到下一个except匹配

	4.第二个【except 异常类型2】，会查询抛出的异常是否和我指定的类型一致
		如果异常和的类型一致，则进入当前except区域进行处理
		如果异常和类型不一致，则跳过当前except区域将异常送到下一个except匹配
	...

	5.如果except区域执行完毕或者没有任何except区域能够接受异常对象，继续运行finally区域


#用户自定义异常
如果要用户自己操作异常对象需要使用到raise语句，抛出系统的内置异常类型或者自定义的异常类型

raise语法

	raise	错误异常对象

自定义异常类型

	class 异常类名(RuntimeError):
		pass

	try:

		raise 自定义异常类() #生成并抛出异常类对象

	except 异常类 as 接收对象变量:

		可以使用接收对象的变量

---

自定义异常类中成员可以根据需要添加或者不添加

	常用的自定义类中的异常属性

		自定义发生异常代码
		自定义发生异常文字提示
		自定义发生异常的行数
		等


#with语法

	try:

		with open('文件','模式') as 变量：#变量接收了open的返回值
			使用变量

	except:
		#错误异常处理区域
		pass


使用该语法操作文件时不需要关闭文件，with会监控文件的使用情况，在适当的时机自动关闭

	