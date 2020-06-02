# cmd 命令学习

## 1.	基础设置

```shell
color  1 #颜色 
title cmd demo # 左上角修改名字
mode 80,50  #列,宽
```

## 2.	ip设置

```shell
ping www.baidu.com  #ttl 操作系统
ping /?  #帮助文档
ping -t  #指定主机,直到停止
ping -n 2 www.baidu.com #发送次数
ping -l 1024 ip #指定大小
```

## 3.	cmd判断网络故障

```shell
ping localhost #丢失lv %d0 正常
```

## 4.	goto跳转命令

```shell
@echo off  #关闭回显
rem 以下是正文 #等同于注释 
echo hello world
exit  #跳出 
pause  #使窗口暂停
```

```shell
@echo off 
rem 以下是正文 
echo hello world
goto prat1 #跳转
exit  #不会执行
:prat1
echo i am ready to exit
pause
```

## 5.	start命令

```shell
start /? #man
start f: #打开f盘
start /max f:   #最大化方式打开
start www.baidu.com  #打开网址
```

## 6.	call命令

```shell
@echo off 
call demo.bat #同一个路劲下 调用文件
echo 引用完成
pause
```

## 7.	sort命令

```shell
sort demol.txt #排序 升序 
sort /+3 demol.txt #从每行的第3个字母排序
sort /r demol.txt #倒序
type demol.txt #打印文件信息
sort demol.txt > 1.txt #生成排序后的文件
sort demol.txt /o 1.txt #和上面功能一样
```

## 8.	重定向操作符

```shell
ping baidu.com >f:\a.txt #将内容写入到a.txt 覆盖
ping github.com >>f:\a.txt # 添加新内容 不覆盖
sort <f:\a.txt  #将内容显示到终端
```











