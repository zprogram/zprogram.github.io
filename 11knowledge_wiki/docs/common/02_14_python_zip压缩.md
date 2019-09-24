## zipfile zip文件操作
```
	import zipfile
```
zip文件格式是通用的文档压缩标准，在ziplib模块中，使用ZipFile类来操作zip文件，下面具体介绍一下：

zipfile.ZipFile(file[, mode[, compression[, allowZip64]]])
```
	创建一个ZipFile对象，表示一个zip文件。参数file表示文件的路径或类文件对象(file-like object)；参数mode指示打开zip文件的模式，默认值为’r’，表示读已经存在的zip文件，也可以为’w’或’a’，’w’表示新建一个zip文档或覆盖一个已经存在的zip文档，’a’表示将数据附加到一个现存的zip文档中。参数compression表示在写zip文档时使用的压缩方法，它的值可以是zipfile. ZIP_STORED 或zipfile. ZIP_DEFLATED。如果要操作的zip文件大小超过2G，应该将allowZip64设置为True。
```
ZipFile还提供了如下常用的方法和属性：

ZipFile.getinfo(name):
```
	获取zip文档内指定文件的信息。返回一个zipfile.ZipInfo对象，它包括文件的详细信息。将在下面 具体介绍该对象。
```
ZipFile.infolist()
```
	获取zip文档内所有文件的信息，返回一个zipfile.ZipInfo的列表。
```
ZipFile.namelist()
```
	获取zip文档内所有文件的名称列表。
```
ZipFile.extract(member[, path[, pwd]])

	将zip文档内的指定文件解压到当前目录。参数member指定要解压的文件名称或对应的ZipInfo对象；参数path指定了解析文件保存的文件夹；参数pwd为解压密码。下面一个例子将保存在程序根目录下的txt.zip内的所有文件解压到D:/Work目录：
```	
	Python
	
	import zipfile, os
	zipFile = zipfile.ZipFile(os.path.join(os.getcwd(), 'txt.zip'))
	for file in zipFile.namelist():
	    zipFile.extract(file, r'd:/Work')
	zipFile.close()
	
	import zipfile, os
	zipFile = zipfile.ZipFile(os.path.join(os.getcwd(), 'txt.zip'))
	for file in zipFile.namelist():
	    zipFile.extract(file, r'd:/Work')
	zipFile.close()
```
ZipFile.extractall([path[, members[, pwd]]])

解压zip文档中的所有文件到当前目录。参数members的默认值为zip文档内的所有文件名称列表，也可以自己设置，选择要解压的文件名称。

ZipFile.printdir()
```
	将zip文档内的信息打印到控制台上。
```
ZipFile.setpassword(pwd)
```
	设置zip文档的密码。
```
ZipFile.read(name[, pwd])

	获取zip文档内指定文件的二进制数据。下面的例子演示了read()的使用，zip文档内包括一个txt.txt的文本文件，使用read()方法读取其二进制数据，然后保存到D:/txt.txt。
```
	Python

	import zipfile, os
	zipFile = zipfile.ZipFile(os.path.join(os.getcwd(), 'txt.zip'))
	data = zipFile.read('txt.txt')
	(lambda f, d: (f.write(d), f.close()))(open(r'd:/txt.txt', 'wb'), data)  #一行语句就完成了写文件操作。仔细琢磨哦~_~
	zipFile.close()
	
	import zipfile, os
	zipFile = zipfile.ZipFile(os.path.join(os.getcwd(), 'txt.zip'))
	data = zipFile.read('txt.txt')
	(lambda f, d: (f.write(d), f.close()))(open(r'd:/txt.txt', 'wb'), data)  #一行语句就完成了写文件操作。仔细琢磨哦~_~
	zipFile.close()
```
ZipFile.write(filename[, arcname[, compress_type]])
```
	将指定文件添加到zip文档中。filename为文件路径，arcname为添加到zip文档之后保存的名称, 参数compress_type表示压缩方法，它的值可以是zipfile. ZIP_STORED 或zipfile. ZIP_DEFLATED。下面的例子演示了如何创建一个zip文档，并将文件D:/test.doc添加到压缩文档中。
```
ZipFile.writestr(zinfo_or_arcname, bytes)
```
	writestr()支持将二进制数据直接写入到压缩文档。
```

ZipFile.getinfo(name) 方法返回的是一个ZipInfo对象，表示zip文档中相应文件的信息。它支持如下属性：
```
	ZipInfo.filename： 获取文件名称。
	ZipInfo.date_time： 获取文件最后修改时间。返回一个包含6个元素的元组：(年, 月, 日, 时, 分, 秒)
	ZipInfo.compress_type： 压缩类型。
	ZipInfo.comment： 文档说明。
	ZipInfo.extr： 扩展项数据。
	ZipInfo.create_system： 获取创建该zip文档的系统。
	ZipInfo.create_version： 获取 创建zip文档的PKZIP版本。
	ZipInfo.extract_version： 获取 解压zip文档所需的PKZIP版本。
	ZipInfo.reserved： 预留字段，当前实现总是返回0。
	ZipInfo.flag_bits： zip标志位。
	ZipInfo.volume： 文件头的卷标。
	ZipInfo.internal_attr： 内部属性。
	ZipInfo.external_attr： 外部属性。
	ZipInfo.header_offset： 文件头偏移位。
	ZipInfo.CRC： 未压缩文件的CRC-32。
	ZipInfo.compress_size： 获取压缩后的大小。
	ZipInfo.file_size： 获取未压缩的文件大小。
```
下面一个简单的例子说明这些属性的意思：
```
	Python
	
	import zipfile, os
	zipFile = zipfile.ZipFile(os.path.join(os.getcwd(), 'txt.zip'))
	zipInfo = zipFile.getinfo('doc.doc')
	print 'filename:', zipInfo.filename
	print 'date_time:', zipInfo.date_time
	print 'compress_type:', zipInfo.compress_type
	print 'comment:', zipInfo.comment
	print 'extra:', zipInfo.extra
	print 'create_system:', zipInfo.create_system
	print 'create_version:', zipInfo.create_version
	print 'extract_version:', zipInfo.extract_version
	print 'extract_version:', zipInfo.reserved
	print 'flag_bits:', zipInfo.flag_bits
	print 'volume:', zipInfo.volume
	print 'internal_attr:', zipInfo.internal_attr
	print 'external_attr:', zipInfo.external_attr
	print 'header_offset:', zipInfo.header_offset
	print 'CRC:', zipInfo.CRC
	print 'compress_size:', zipInfo.compress_size
	print 'file_size:', zipInfo.file_size
	zipFile.close()
```
