## 字符串相关操作

	+ 字符串连接符号


	* 字符复制操作  
		
		'★' × 10  复制10个小星星
		
	[] 通过指定位置（索引）来获取指定位置的字符
	
		str[1]  获取第二个位置的字符
	
	[:] 取片操作
	
		[:] 		获取所有字符
		[开始位置:] 	从开始位置获取到字符串的结尾
		[：结束位置] 从字符串的开头截取到字符串结束位置之前（不包含结束位置）
		[开始位置:结束位置] 从字符串的开始位置截取到结束位置之前（不包含结束位置）
	
	r'字符串'
	
		防止字符串中的转义字符串实现功能，把转义字符当作普通字符使用

## 字符串相关函数  

**注意：以下的str不是模块名称，而是一个字符串变量！**

capitalize() 首字母大写

	格式： str.capitalize()
	返回值：首字母大写的字符串，仅把整个字符串的第一个字符大写

upper() 将所有英文字符变为大写

	格式：str.upper()
	返回值：返回所有英文字符大写的字符串

lower（） 将所有英文字符变为小写

	格式：str.lower()
	返回值：返回所有英文字符小写的字符串

swapcase() 大小写互相转换

	格式：str.swapcase()
	返回值：转换之后的字符串

title() 按照标题格式进行大小写转换（每个单词首字母大写）

	格式：str.title()
	返回值：返回所有英文单词首字母大写的字符串


len() 计算字符串的字符个数,以后也可以用于计算元组列表等序列
​	
​	格式: len(str)
​	返回值:整型

count() 计算一个字符串中出现指定字符串的次数

	格式：str.count('查找的字符串'[,开始位置])
	
	返回值:返回整型

find() 查找字符串中是否具有指定的字符串，查找不到返回-1

	格式：str.find(查找的字符串[，开始位置])
	返回值:第一次出现的位置

index() 查找字符串中是否具有指定的字符串，查找不到直接报错

	格式：str.index(查找的字符串[，开始位置])
	返回值:第一次出现的位置

**注意**：find()和index()功能基本相似


startswith() 检测字符串是否以指定的字符串开头

	格式： str.startswith('查找的字符串'[，开始位置])
	返回值：布尔值

endswith() 检测字符串是否以指定的字符串结尾
​	
​	格式： str.endswith('查找的字符串'[，开始位置])
​	返回值：布尔值



isupper() 检测字符串中字母是否都是大写字母

	格式：str.isupper()
	返回值：布尔值

islower()检测字符串中的字母是否都是小写字母

	格式：str.islower()
	返回值：布尔值

isalnum() 检测一个字符串是否都是有数字或者是否都是有字母组成

	格式：str.isalnum()
	返回值:布尔值

**注意**：空字符串返回False

isalpha() 检测字符串是否都是有字母类型构成，汉字作为字母处理

	格式：str.isalpha()
	返回值：布尔值
**注意**：空字符串返回False

isdigit() 检测字符串是否由纯数字字符组成

	格式：str.isdigit()
	返回值：布尔值
**注意**：空字符串返回False

isspace() 检测字符串是否完全由空白字符组成

	格式：str.isspace()
	返回值：布尔值

**注意：**回车，换行，缩进，空格都可以当作空白字符，空字符串是False

istitle() 检测字符串是否符合title()的结果，每个单词首字母大写

	格式：str.istitle()
	返回值：布尔值

isnumeric() 检测字符串是否有纯数字构成

	格式：str.isnumeric()
	返回值：布尔值

isdecimal(）检测字符串是否完全由十进制字符组成

	格式：str.isdecimal()
	返回值：布尔值


split()  使用指定的字符将字符串卷拆解成多个字符串

	格式： str.split('用于拆解的字符串')
	返回值：列表类型

join() 使用指定的字符串将序列中的内容组成新的字符串
​	
​	格式：连接字符串.join(序列)
​	返回值：组成的新的字符串

zfill() 0填充操作
​	格式： str.zfill(长度)
​	返回值：填充0的字符串

**注意：** 1.填充结果原有内容靠右对齐，前面位数不足使用0来补充，常用于数字类型字符串。
​			


center() 将字符串进行居中操作，并且在空白处进行填充

	格式： str.center(未来字符宽度,空白填充字符)
	返回值：新的字符串

ljust（） 将字符串进行左对齐操作，并且在空白处进行填充

	格式： str.ljust(未来字符宽度,空白填充字符)
	返回值：新的字符串

rjust() 将字符串进行右对齐操作，并且在空白处进行填充

	格式： str.rjust(未来字符宽度,空白填充字符)
	返回值：新的字符串

lstrip() 去掉字符串左侧的空白

	格式： str.lstrip()
	返回值：字符串

rstrip()去掉字符串右侧的空白

	格式： str.rstrip()
	返回值：字符串

strip()去掉字符串两侧的空白

	格式： str.strip()
	返回值：字符串

replace() 字符串替换操作

	格式： str.replace(被替换的字符串,替换字符串)
	返回值：替换之后的字符串、

maketrans() 制作一个字符串映射表，为了给translate函数使用

	格式:str.maketrans（'被替换的字符串','替换字符串'）
	返回值：字典类型，被替换字符串组成键，替换字符串组成值
	
	注意： 两个参数的字符串个数必须一一对应，这里的str不是值字符串，而是真正的str，也可以写空字符串代替str（字符串都是str制作的）

translate() 进行字符串翻译操作，类似转换

	格式：str.translate(映射表)
	返回值：新的字符串
	
	注意:translate用于多字符替换，replace用于长字符串替换
## 字符串模块相关内容 string模块

### 获取所有的空白字符  
result = string.whitespace

print(result)

	\n \r \t   \x0b\x0c

###  获取ascii码的所有字母表（包含大写和小写）
print(string.ascii_letters)

	abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ

### 获取ascii码中的所有大写字母

print(string.ascii_uppercase)

	ABCDEFGHIJKLMNOPQRSTUVWXYZ

### 获取ascii码中所有的小写字母

print(string.ascii_lowercase)

	abcdefghijklmnopqrstuvwxyz

### 获取ascii码中所有10进制数字字符

print(string.digits)

	0123456789

### 获取八进制所有数字字符

print(string.octdigits)

	01234567

### 获取十六进制的所有数字字符

print(string.hexdigits)

	0123456789abcdefABCDEF

### 打印所有可见字符
print(string.printable)

	0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~ 

### 打印所有标点符号
print(string.punctuation)

	!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~



## format格式化字符串

format()  格式化字符串

	格式：str.format()
	返回值：格式化之后的字符串


用法：
​	