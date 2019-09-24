## shutil模块
```
	import shutil 
```

copy() 复制文件
```
	格式：shutil.copy(来源路径，目标路径)
	返回值：返回目标路径
```
copy2() 复制文件，保留元数据（文件信息）
```
	格式：shutil.copy2(来源路径，目标路径)
	返回值：返回目标路径
	
	注意：copy和copy2的唯一区别在于copy2复制文件时尽量保留元数据
```

copyfileobj() 将一个文件中的内容复制到另外一个文件当中
```
	格式：shutil.copyfileobj(open('源路径','r'),open('目标路径','w'))
	
	返回值：无
```
copyfile()将一个文件中的内容复制到另外一个文件当中
```
	格式：shutil.copyfile（'源路径','目标路径')
	
	返回值：无
	
	注意：如果源路径和目标路径相同，会报错误！
```
copytree（） 直接复制整个目录树中的所有内容
```
	格式：shutil.copytree（源文件夹，目标文件夹）
	返回值：返回目标路径
```
copymode()  复制权限  仅供参考

copystat() 复制状态   仅供参考

rmtree() 删除整个目录树，即使非空目录也可以删除
```
	格式：shutil.rmtree(路径)
	返回值：无
```
move() 移动文件/文件夹
```
	格式：shutil.move(源路径，目标路径)
	返回值：目标路径！
```
which() 获取可执行程序的位置
```
	格式：shutil.which('命令字符串')
	返回值:命令程序位置的路径字符串
```
disk_usage() 获取磁盘使用情况
```
	格式：shutil.usage('盘符')
	返回值：元组（总计,使用，剩余）
```
----

### 归档和压缩

归档：将多个文件或者文件夹合并到一个文件当中
```
	减少文件数目，方便发送邮件等操作
```
压缩：使用算法将多个文件或者文件夹无损或者有损的合并到同一个文件当中
```
	减少磁盘占用空间，也方便发送邮件等操作。
```
make_archive() 归档操作
```
	格式:shutil.make_archive('归档之后的目录和文件名','后缀','需要归档的文件夹')
	
	返回值：归档之后的地址
```
unpack_archive() 解包操作
```
	格式：shutil.unpack_archive('归档文件地址','解包之后的地址')
	返回值：解包之后的地址
```
get_archive_formats() 获取系统已注册的归档格式和信息
```
	格式：shutil.archive_formats()
	返回值：列表
```
get_unpack_foramts() 获取系统已经注册的解包格式和信息
```
    格式：shutil.unpack_formats()
    返回值：列表
```