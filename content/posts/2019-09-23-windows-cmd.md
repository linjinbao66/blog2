---
title: windows常用批处理命令
tags:
  - bat
  - windows
  - 小工具
categories:
  - 小工具
date: 2019-09-23
---

## Windows批处理

## 常用dos命令

```text
echo     打印
dir 　　　列文件名
cd　　　　改变当前目录
ren 　　　改变文件名
copy　　　拷贝文件
del 　　　删除文件
md　　　　建立子目录
rd　　　　删除目录
deltree　 删除目录树
format　　格式化磁盘
edit　　　文本编辑
type　　　显示文件内容
mem 　　　查看内存状况
##       以下是新增加的命令
help　　　显示帮助提示
cls 　　　清屏
move　　　移动文件，改目录名
more　　　分屏显示
xcopy 　　拷贝目录和文件
```

## 显示篇

1. `echo` `@` 和 `pause` 在`DOS`命令提示符中使用 `echo /?`可以获得对 echo 用法的解释。`echo on` 用于打开命令的回显；`echo off` 用于关闭命令的回显\(默认情况下，`ehco` 是处于打开状态的\)。只输入 `echo` 可以获得当前的回显状态\(是否处于打开状态\)。输入 `echo` 再加一段文字，例如 `echo Hello world!` 可以显示出 Hello world! 这句信息。 `@` ，如果在某一条命令最前面加上 @ ，那么这一行命令就不会显示出来。与 `echo off` 有着相似之处。 `echo off` 以后的所有命令本身都不再显示出来；而 @ 只是将当前那一行的命令不显示出来。然而，至于命令所产生的输出结果，仍然会显示出来。这看起来似乎有些拗口，但我们会通过例子来很容易地理解它们。 `pause` ，从字面上看就是暂停的意思，效果等同于将程序挂起，在按下任意键后才继续。
2. `>` 和 `>>` `>` 表示将输出结果打印到某处。比如： `echo Hello world!>d:\a.txt` 表示将 Hello world! 这句话写入到 D:\a.txt 文件中。如果以前该文件中已经存在，并且有自己的内容，那么以前的内容就被覆盖掉了。比如我们再输入：`echo yo, whats up>d:\a.txt` ，那么文件 a.txt 中以前的 Hello world! 就变成了现在的新例句。 `>>` 与`>` 类似，也可以将输出结果打印到某处。比如我们用 `echo nothin much, and u?>>d:\a.txt` 将例句写到 a.txt 里时，该例句并不会覆盖原有的 `yo, whats up` 这句话，而是加在了原句的后面。
3. `title` 和 `rem` title 后面跟字符串可以改变当前命令提示符的标题名称。输入 title 这是新标题 后，该命令提示符左上角的标题名称已经变为"这是新标题"了。输入中文可以通过 Ctrl+空格 切换出中文输入法；也可以通过复制粘贴的方式输入。 rem 的用法就很简单了，rem 后面跟上一段文字，在批处理中可以作为注释用。rem 和它后面跟的文字在实际运行时并不会起任何作用，只是为了方便人们阅读该批处理时更容易理解而已\(如果您用过C的话，一定会联想到C语言里的 // 或 / __/ 的用法\)。除了 rem 外，两个连续的冒号 :: 也起同样的作用。提示：rem 与 :: 的区别在于，rem 也是一种命令，在 echo on 的情况下会被显示出来，而 :: 却不会
4. 其他命令 prompt ，这就是命令提示符中所谓的"提示符"了。在命令提示符中输入 prompt 加一段文字能够使得提示符不再是以传统的路径名和大于号组成的，而是以我们刚才输入的那段文字开头的。这也许不是很好理解，或者您对 prompt 的含义还不清楚或只知道其字面含义。这并不要紧，如果您只要简单地输入 prompt 提示符 就能很快地明白 prompt 的含义了。此外，要想恢复以前的路径名和大于号为开头的提示符，只需要再输入 prompt $p$g 即可。这里$p 表示当前驱动器和路径， $g 表示大于号。因为一些特殊的格式或符号需要用 $ 加特定的字母来表示。具体的说明可以用 help prompt 来查询。

## 赋值，调用，参数

1. 赋值 赋值指令:`set`

   ```text
   set var=helloljbao
   set var1=%var%
   ```

2. 调用 与很多编程类的东西一样，批处理并不一定非得按照文本中命令的排列顺序一行一行地执行。如果遇到了 `goto`、`call`、`start` 这样的跳转、调用、启动等语句，程序通常会变得多层化，执行起来会更加有效。

   \`\`\`dos @echo off goto :first :second echo then display pause goto :EOF

:first echo first display pause goto :second

```text
```dos
@echo off
echo this is b
pause
call a.bat
echo back now b.bat
pause
```

```text
`start msconfig`用来打开"系统配置应用程序"；`start notepad` 则可以打开一个记事本；当然，start 也可以打开另外一个批处理。这看起来似乎与 call 相仿，却有一些区别。为了对比 call 和 start 之间的效果差异，可以在上一个例子中修改"调用.bat"中的 call 被调用.bat 为 start 被调用.bat 。从"调用.bat"开始执行后，"被调用.bat"也的确能够被调用。但之前的批处理也同时继续执行着。事实上，此时这两个程序已经完全独立开了，是两个独立的进程
```

```text
::::b.bat::::
@echo off
echo thisis  b.bat
pause
call a.bat Helloworld! kk
echo back  b.bat
pause
::::::::::

:::::a.bat:::::
echo this a.bat
echo input1 %1
echo input2 %2
pause
::::::::::
```

1. 参数

```text
::a.bat::
@echo off
pause
set /p num=inputnum
set square=
call b.bat %num% square 
echo %num%,%square%
::::

::b.bat::
@echo off
echo input1 %1
echo input2 %2
set /a %2 = %1 * %1
pause
::::

::cmd::
linjinbao@DESKTOP-I1CQLLJ F:\batLearn     
$ a                                       
请按任意键继续. . .                              
inputnum12                                
input1 12                                 
input2 square                             
请按任意键继续. . .                              
12,144                                    
::::
```

## 条件，循环

1. 条件`if`

   \`\`\`dos

   :::::b.bat::::::

   @echo off

   echo which color do you choose

:retry set /p choice=input your choice R or G or S: if "%choice%"=="R" goto R if "%choice%"=="r" goto R if "%choice%"=="G" goto G if "%choice%"=="g" goto G goto retry

:R color c pause goto retry

:G color a pause goto retry

```text
  2. 循环`for`
```dos
:::::::批量修改文件名.bat:::::::
@echo off
setlocal EnableDelayedExpansion
set /a num=1
for %%i in (D:\test\*.txt) do (
ren "%%i" !num!.txt
set /a num+=1
)
::::::::::::::::::::::::::::::::
```

```text
::::::::批量改文件名.bat::::::::
@echo off
setlocal EnableDelayedExpansion

set /p zpath=请输入目标文件所在的路径：
set /p prefix=请输入文件名前缀(不能包含以下字符\/:*?"<>|)：
set /p ext=请输入文件的扩展名(例如 .txt)：
set /a num=1

for %%i in (%zpath%\*%ext%) do (
ren "%%i" "%prefix%!num!.%ext%"
set /a num+=1
)
:::::::::::::::::::::::::::::::
```

## 组合命令

1. 组合命令 `&` 、`&&` 和 `||` 是一类用于两个或多个命令语句之间起衔接作用的符号。这对于我们想一次性执行两条或多条命令，以及前面命令执行结果的成功与否作为后面命令是否被执行的衡量标准，起着决定性的作用。
2. 管道命令`>` 、`>>`

```text
echo a > f:\exe.txt && dir %windir%\system32\*.exe > f:\exe.txt
```

## 常用实例

1. 批量修改文件名

   \`\`\`dos

   :::::::批量修改文件名.bat:::::::

   @echo off

   title 批量修改文件名

   setlocal EnableDelayedExpansion

   :: 启用延迟变量扩充

:GetPath set zpath=%CD% :: 对变量进行初始化，防止用户不输入而直接跳过。其中%CD%表示当前路径 set /p zpath=请输入目标文件所在的路径： if %zpath:~0,1%%zpath:~-1%=="" set zpath=%zpath:~1,-1% :: 检查变量 zpath 的第一个和最后一个字符是否为 "" ，是的话就去掉 if not exist "%zpath%" goto :GetPath :: 如果 zpath 值的路径不存在，就得跳转回去，要求重新输入

:GetPrefix set prefix=未命名 set /p prefix=请输入文件名前缀\(不能包含以下字符\/:_?"&lt;&gt;\|\)： for /f "delims=\/:_?&lt;&gt;\| tokens=2" %%i in \("z%prefix%z"\) do goto :GetPrefix :: 这里对变量 perfix 进行检查，发现有非法符号便跳转到 :GetPrefix :: 事实上，这里并没有对双引号 " 进行检测，因为双引号无法在此被转义为可用的分隔符 :: 即使是在这个程序里，不正确地使用双引号也会引起程序异常而退出。 :: 因此，想把它做的非常人性化并不是一件容易的事情

:GetExt set ext=. _set /p ext=请输入文件的扩展名\(不输入则表示所有类型\)： if not "%ext:~0,1%"=="." set ext=.%ext% :: 检查变量 ext 的第一个是否为句点 . ，不是的话就加上 :: 建议这里对变量 ext 也检查一下，发现有除_外的非法符号便跳转到 :GetExt

set answer=N echo. echo 您试图将 %zpath% 里的所有 %ext% 类型的文件以 %prefix% 为前缀名进行批量改名，是否继续？ set /p answer=继续请输入 Y ，输入其它键放弃... if "%answer%"=="Y" goto :ReadyToRename if "%answer%"=="y" goto :ReadyToRename

echo 放弃文件改名，按任意键退出... & goto :PauseThenQuit

:ReadyToRename

set /a num=0 echo.

if "%ext%"=="._" \( for %%i in \("%zpath%\_%ext%"\) do \( set /a num+=1 ren "%%i" "%prefix%!num!%%~xi" \|\| echo 文件 %%i 改名失败 && set /a num-=1 \) \) else \( for %%i in \("%zpath%\*%ext%"\) do \( set /a num+=1 ren "%%i" "%prefix%!num!%ext%" \|\| echo 文件 %%i 改名失败 && set /a num-=1 \) \)

if %num%==0 echo %zpath% 里未发现任何文件。按任意键退出... & goto :PauseThenQuit

echo 文件改名完成，按任意键退出...

:PauseThenQuit pause&gt;nul ::::::::::::::::::::::::::::::::

```text
2. 批量备份进程映像列表以及注册表自启动项
```dos

::::::备份进程和自启动.bat::::::
@echo off

set TempFolder=临时文件夹
:: 设置临时文件夹的名称
if not exist %TempFolder% md %TempFolder%
:: 创建临时文件夹
echo.>%TempFolder%\temp.tmp
:: 在临时文件夹里创建一个临时文件 temp.tmp ，并清空其内容

set ExportImageName=%cd%\进程映像列表%date:~0,10%.txt
:: 设置默认导出文件的路径以及文件名
set /p ExportImageName=所要导出的进程映像名称列表备份文件名：
:: 用户自定义输入的路径文件名

for /f "delims=," %%i in ('tasklist /nh /fo csv') do echo %%~i>>%TempFolder%\temp.tmp
:: 将 tasklist 中的进程映像名输出到临时文件 temp.tmp 中 [注1]
sort %TempFolder%\temp.tmp>"%ExportImageName%"
:: 将进程映像名按字母顺序排列
del %TempFolder%\temp.tmp
:: 删除临时文件夹里的文件 temp.tmp
echo 当前进程中的所有映像名已导出到以下文件：%ExportImageName%
pause
:::::::: 以上完成了进程备份，下面开始备份自启动的注册表信息

set ExportRegRunName=%cd%\注册表自启动%date:~0,10%.txt
:: 设置默认导出文件的路径以及文件名
set /p ExportRegRunName=所要导出的注册表自启动备份文件名：
:: 用户自定义输入的路径文件名

reg export HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run %TempFolder%\HKLMrun.reg
reg export HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run %TempFolder%\HKLMexp.reg
reg export HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run %TempFolder%\HKCUrun.reg
reg export HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run %TempFolder%\HKCUexp.reg
:: 将相关的注册表信息导出到各个临时的注册表文件中 [注2]

copy %TempFolder%\*.reg "%ExportRegRunName%">nul
:: 将临时文件夹里所有的注册表文件合并复制到一个文件中
del %TempFolder%\*.reg
:: 删除临时文件夹里所有的临时注册表文件
if exist %TempFolder% rd %TempFolder%
:: 删除临时文件夹
echo 当前自启动的注册表信息已导出到以下文件：%ExportRegRunName%
pause
::::::::::::::::::::::::::::::::
```

1. 批量查看同一子网络下的所有IP在线情况

   \`\`\`dos

   :::::::查看所有子网IP.bat:::::::

   @echo off

   title 查看所有子网IP

set /a Online=0 set /a Offline=0 set /a Total=256 set ExportFile=子网IP在线统计.txt :: 初始化在线IP与不在线IP的个数为零，共扫描256个IP，结果输出的文件名

set StartTime=%time% :: 记录程序的开始时间

for /f "delims=: tokens=2" %%i in \('ipconfig /all ^\| find /i "IP Address"'\) do set IP=%%i :: 获得本机IP \[注1\]

if "%IP%"=="" echo 未连接到网络 & pause & goto :EOF if "%IP%"==" 0.0.0.0" echo 未连接到网络 & pause & goto :EOF :: 当IP为空或 0.0.0.0 时，提示未连接并退出该程序

for /f "delims=. tokens=1,2,3,4" %%i in \("%IP%"\) do \( set /a IP1=%%i set /a IP2=%%j set /a IP3=%%k set /a IP4=%%l \) :: 以句点为分隔符，分别将IP的四个十进制数赋给四个变量

set /a IP4=0 echo 在线的IP：&gt;%ExportFile% :: 初始化IP的第四个数值为零，并创建结果输出文件

:RETRY ping %IP1%.%IP2%.%IP3%.%IP4% -n 1 -w 200 -l 16&gt;nul && set /a Online+=1 && echo %IP1%.%IP2%.%IP3%.%IP4%&gt;&gt;%ExportFile% \|\| set /a Offline+=1 :: ping 目标IP \[注2\]

set /p =\[将本文底部评论4中的退格符替换到此处\]&lt;nul set /a Scanned=%Online%+%Offline% set /a Progress=\(%Online%+%Offline%\)\*100/%Total% set /p =正在扫描：%Scanned%/%Total% 扫描进度：%Progress%%%&lt;nul :: 删除当前行的内容，并重新显示进度信息 \[注3\]

set /a IP4+=1 if %IP4% lss %Total% goto :RETRY :: 当IP的第四个数值小于总数时，跳转回 :RETRY 处，重复执行直到全部 ping 完为止

echo. echo.

set EndTime=%time% :: 记录程序的结束时间

set /a Seconds = %EndTime:~6,2% - %StartTime:~6,2% set /a Minutes = %EndTime:~3,2% - %StartTime:~3,2% if %Seconds% lss 0 set /a Seconds += 60 & set /a Minutes -= 1 if %Minutes% lss 0 set /a Minutes += 60 :: 计算时间差

set /a Percent=%Online%\*100/\(%Online%+%Offline%\) :: 计算在线百分比

echo 在线IP个数: %Online% echo 不在线IP个数: %Offline% echo 在线百分比: %Percent%%% echo 统计耗时: %Minutes%分%Seconds%秒 echo 统计日期: %date% %time:~0,-3% echo.&gt;&gt;%ExportFile% echo 在线IP个数: %Online%&gt;&gt;%ExportFile% echo 不在线IP个数: %Offline%&gt;&gt;%ExportFile% echo 在线百分比: %Percent%%%&gt;&gt;%ExportFile% echo 统计耗时: %Minutes%分%Seconds%秒&gt;&gt;%ExportFile% echo 统计日期: %date% %time:~0,-3%&gt;&gt;%ExportFile% echo 记录已保存到文件"%ExportFile%"中 ::显示结果并将结果保存到文件中 pause ::::::::::::::::::::::::::::::::

```text
4. 坐在家里周游世界
```dos
::::::坐在家里周游世界.bat::::::
@echo off
title 坐在家里周游世界
:: 设置标题
:Start
cls
:: 清屏
set choice=1
set /p choice=请选择经纬度格式(1\. 小数格式 2\. 度分秒格式 3\. 退出)：
:: 选择经纬度的格式 或 退出程序
if %choice%==3 goto :EOF
if %choice%==2 goto :DMSFormat

:DecimalFormat
:: 小数格式的经纬度处理
set latitude=39.906477
set longitude=116.391467
:: 初始化为伟大首都天安门的坐标
set /p latitude=纬度：
set /p longitude=经度：
:: 提示用户输入目标的经纬度 [注1]
goto :LaunchMap

:DMSFormat
:: 度分秒格式的经纬度处理
set "DMSlatitude=39 54 23 N"
set /a degree=39
set /a minute=54
set /a second=23
set NorthOrSouth=N
:: 初始化为美丽首都天安门度分秒格式的纬度
echo.
set /p DMSlatitude=纬度(格式 [度] [分] [秒] [N ^| S])：
:: 提示用户输入目标的纬度 [注2]
for /f "tokens=1,2,3,4,5" %%i in ("%DMSlatitude%") do (
set degree=%%i
set minute=%%j
set second=%%k
set NorthOrSouth=%%l
if "%%l"=="" echo 不正确的格式 & goto :DMSFormat
)
:: 分别获得纬度的度、分、秒，以及南半球或北半球
if %degree% lss 0 echo 纬度必须大于0度 或不正确的格式 & goto :DMSFormat
if %degree% gtr 90 echo 纬度必须小于90度 或不正确的格式 & goto :DMSFormat
if %minute% lss 0 echo 纬度必须大于0分 或不正确的格式 & goto :DMSFormat
if %minute% gtr 60 echo 纬度必须小于60分 或不正确的格式 & goto :DMSFormat
if %second% lss 0 echo 纬度必须大于0秒 或不正确的格式 & goto :DMSFormat
if %second% gtr 60 echo 纬度必须小于60秒 或不正确的格式 & goto :DMSFormat
:: 判断纬度的度、分、秒格式是否有效 [注3]
set /a degree*=3600
set /a minute*=60
set /a second=%degree%+%minute%+%second%
set /a latitude=%second%*2500/9
:: 将度分秒计算并转换为小数格式 [注4]
if %NorthOrSouth%==N goto :LatitudeLockDown
if %NorthOrSouth%==n goto :LatitudeLockDown
if %NorthOrSouth%==S set /a latitude=0-%latitude% & goto :LatitudeLockDown
if %NorthOrSouth%==s set /a latitude=0-%latitude% & goto :LatitudeLockDown
:: 判断南北半球格式是否有效
echo 南北半球标识不明确，请使用 N 或 S 来表示，不区分大小写。 & goto :DMSFormat

:LatitudeLockDown
set "DMSlongitude=116 23 29 E"
set /a degree=116
set /a minute=23
set /a second=29
set EastOrWest=E
:: 初始化为可爱首都天安门度分秒格式的经度
echo.
set /p DMSlongitude=经度(格式 [度] [分] [秒] [E ^| W])：
:: 提示用户输入目标的经度
for /f "tokens=1,2,3,4" %%i in ("%DMSlongitude%") do (
set degree=%%i
set minute=%%j
set second=%%k
set EastOrWest=%%l
if "%%l"=="" echo 不正确的格式 & goto :LatitudeLockdown
)
:: 分别获得经度的度、分、秒，以及东半球或西半球
if %degree% lss 0 echo 经度必须大于0度 或不正确的格式 & goto :LatitudeLockDown
if %degree% gtr 180 echo 经度必须小于180度 或不正确的格式 & goto :LatitudeLockDown
if %minute% lss 0 echo 经度必须大于0分 或不正确的格式 & goto :LatitudeLockDown
if %minute% gtr 60 echo 经度必须小于60分 或不正确的格式 & goto :LatitudeLockDown
if %second% lss 0 echo 经度必须大于0秒 或不正确的格式 & goto :LatitudeLockDown
if %second% gtr 60 echo 经度必须小于60秒 或不正确的格式 & goto :LatitudeLockDown
:: 判断经度的度、分、秒格式是否有效
set /a degree*=3600
set /a minute*=60
set /a second=%degree%+%minute%+%second%
set /a longitude=%second%*2500/9
:: 将度分秒计算并转换为小数格式
if %EastOrWest%==E goto :LongitudeLockDown
if %EastOrWest%==e goto :LongitudeLockDown
if %EastOrWest%==W set /a longitude=0-%longitude% & goto :LongitudeLockDown
if %EastOrWest%==w set /a longitude=0-%longitude% & goto :LongitudeLockDown
:: 判断东西半球格式是否有效
echo 东西半球标识不明确，请使用 E 或 W 来表示，不区分大小写。 & goto :LatitudeLockDown

:LongitudeLockDown
set latitude=%latitude:~0,-6%.%latitude:~-6%
set longitude=%longitude:~0,-6%.%longitude:~-6%
:: 整理纬度和经度

:LaunchMap
echo.
set /a zoom=16
set /p zoom=放缩度(0~18 默认值：%zoom%)：
:: 提示用户输入照片的放缩值
echo.
echo 正在打开 纬度：%latitude% 经度：%longitude% 的卫星照片...
start "正在打开Google Maps..." "http://maps.google.com/?t=k&z=%zoom%&ll=%latitude%,%longitude%"
:: 将放缩值和经纬度值作为 Google Maps 链接参数，打开相应的照片 [注5]

pause
goto :Start
::::::::::::::::::::::::::::::::
```

1. 进程分析者

   \`\`\`dos

   :::::::::进程分析者.bat:::::::::

   @echo off

   setlocal enabledelayedexpansion

title 进程分析者 set SPACE= set /a NumOfTotal=0 set /a NumOfSafe=0 set /a NumOfNasty=0 set /a NumOfUnknown=0 set IconOfSafe=√ set IconOfNasty=× set IconOfUnknown=？

:::::::: 以下定义为可信任的进程，可自定义更多的扩充 :::::::: :: 1. 系统进程 set alg.exe=%IconOfSafe%处理Windows网络连接共享和网络连接防火墙\[系统进程\] set csrss.exe=%IconOfSafe%管理Windows图形相关任务\[系统进程\] set explorer.exe=%IconOfSafe%用于显示系统桌面的图标,任务栏等\[系统进程\] set lsass.exe=%IconOfSafe%用于本地安全和登陆策略\[系统进程\] set services.exe=%IconOfSafe%管理启动和停止服务\[系统进程\] set smss.exe=%IconOfSafe%用于对话管理子系统调用和系统对话操作\[系统进程\] set spoolsv.exe=%IconOfSafe%用于将打印机任务发送到本地打印机\[系统进程\] set svchost.exe=%IconOfSafe%用于执行动态链接库DLL文件\[系统进程\] set System=%IconOfSafe%\[系统进程\] set taskmgr.exe=%IconOfSafe%任务管理器，用于显示系统正在运行的进程\[系统进程\] set winlogon.exe=%IconOfSafe%用于处理系统的登陆和登陆过程\[系统进程\] set winmgmt.exe=%IconOfSafe%用于系统管理员创建WIndows管理脚本\[系统进程\] :: 2. 基本进程 set cmd.exe=%IconOfSafe%Windows系统的命令行程序 set msimn.exe=%IconOfSafe%OutlookExpress相关程序 set mspaint.exe=%IconOfSafe%画图板 set notepad.exe=%IconOfSafe%记事本 set wab.exe=%IconOfSafe%通讯薄，用于储存地址、联系人和Email set ctfmon.exe=%IconOfSafe%提供语音识别、手写识别等 set conime.exe=%IconOfSafe%输入法编辑器相关程序 set SOUNDMAN.EXE=%IconOfSafe%Realtek声卡相关程序 set tasklist.exe=%IconOfSafe%这是本批处理的核心所在-\_-b set wmiprvse.exe=%IconOfSafe%用于通过WinMgmt.exe程序处理WMI操作 :: 3. 工作进程 set EXCEL.EXE=%IconOfSafe%Excel set WINWORD.EXE=%IconOfSafe%Word set XDICT.EXE=%IconOfSafe%金山词霸 set sqlservr.exe=%IconOfSafe%用于SQL基础服务 set wmplayer.exe=%IconOfSafe%Windows Media Player set Mplayerc.exe=%IconOfSafe%暴风影音 set WinRAR.exe=%IconOfSafe%WinRAR :: 4. 防护进程 set 360tray.exe=%IconOfSafe%360安全卫士实时监控程序 set AntiArp.exe=%IconOfSafe%ARP防火墙 set CCenter.exe=%IconOfSafe%瑞星信息中心 set RavMonD.exe=%IconOfSafe%瑞星监控程序 set rfwsrv.exe=%IconOfSafe%瑞星个人防火墙相关程序 set RavStub.exe=%IconOfSafe%瑞星杀毒软件相关程序 set RfwMain.exe=%IconOfSafe%瑞星防火墙主程序 set RavTask.exe=%IconOfSafe%瑞星定时杀毒程序 set RavMon.exe=%IconOfSafe%瑞星监控程序 :: 5. 网络进程 set iexplore.exe=%IconOfSafe%IE浏览器 set Maxthon.exe=%IconOfSafe%傲游浏览器 set BaiduHi.exe=%IconOfSafe%百度Hi set msmsgs.exe=%IconOfSafe%MSN网络聊天工具 set QQ.exe=%IconOfSafe%腾迅QQ set TXPlatform.exe=%IconOfSafe%腾迅平台 set Thunder5.exe=%IconOfSafe%迅雷下载 set Skype.exe=%IconOfSafe%Skype语音聊天 set Contentfilter.exe=%IconOfSafe%Skype的相关程序 set skypePM.exe=%IconOfSafe%Skype语音聊天

:::::::: 以下定义为已知的危险进程，可自定义更多的扩充 :::::::: set a.exe=%IconOfNasty%蠕虫 set av.exe=%IconOfNasty%蠕虫 set blss.exe=%IconOfNasty%木马/拨号器 set cmd32.exe=%IconOfNasty%病毒 set crss.exe=%IconOfNasty%蠕虫 set csrse.exe=%IconOfNasty%病毒/木马 set Desktop.exe=%IconOfNasty%木马/病毒/间谍 set directs.exe=%IconOfNasty%蠕虫 set dllhlp.exe=%IconOfNasty%木马 set dllreg.exe=%IconOfNasty%病毒 set explore.exe=%IconOfNasty%灰鸽子 set explored.exe=%IconOfNasty%蠕虫 set optimize.exe=%IconOfNasty%拨号器/广告 set pcsvc.exe=%IconOfNasty%间谍 set rundll16.exe=%IconOfNasty%木马 set run32dll.exe=%IconOfNasty%间谍 set scvhost.exe=%IconOfNasty%木马/广告 set svchosts.exe=%IconOfNasty%木马 set system32.exe=%IconOfNasty%木马 set updater.exe=%IconOfNasty%蠕虫 set web.exe=%IconOfNasty%病毒/木马 set win32.exe=%IconOfNasty%病毒 set windows.exe=%IconOfNasty%蠕虫 set winlogin.exe=%IconOfNasty%病毒/木马 set winstart.exe=%IconOfNasty%间谍/广告 set wintsk32.exe=%IconOfNasty%病毒 set winupdate.exe=%IconOfNasty%病毒 set winxp.exe=%IconOfNasty%病毒 set winhlep.exe=%IconOfNasty%病毒

:::::::: 以下定义为未知的进程 :::::::: set UnknownTask=%IconOfUnknown%未识别的进程

:: 程序开始 echo 进程名称 分析结果 echo.

for /f "tokens=1" %%i in \('tasklist /NH'\) do \( set TaskName=%%i%SPACE% set TaskName=!TaskName:~0,20! if defined %%i \( echo !TaskName! !%%i! if "!%%i:~0,1!"=="%IconOfNasty%" \( set /a NumOfNasty+=1 \) \) else \( echo !TaskName! %UnknownTask% set /a NumOfUnknown+=1 \) set /a NumOfTotal+=1 \) echo _**\_\_**_ echo. echo 共 %NumOfTotal% 个进程。 if %NumOfNasty% gtr 0 \( echo %NumOfNasty% 个存在安全隐患的进程！ \) if %NumOfUnknown% gtr 0 \( echo %NumOfUnknown% 个未知进程。 \) pause&gt;nul ::::::::::::::::::::::::::::::::

```text
6. 别挤，自启动的排队来

```dos
::::::::::zLauncher.bat:::::::::
@echo off
setlocal enabledelayedexpansion
title Launching...
set /a ShortDelay = 6
set BackSpace=[将本文底部评论3中的退格符替换到此处]

set /p choice=LaunchOptions:
if not "%choice%"=="" goto :CustomLaunch

:BasicLaunch
call :LaunchItem1
call :LaunchItem2
call :LaunchItem3
call :LaunchItem4
call :LaunchItem5
goto :EOF

:CustomLaunch
for /l %%i in (0 1 9) do (
set "choice_=!choice:~%%i,1!"
if "!choice_!"=="" echo All Launched! & goto :EOF
if !choice_! gtr 9 echo Unknown Define & goto :EOF
if !choice_! lss 0 echo Unknown Define & goto :EOF
call :LaunchItem!choice_!
)

:LaunchItem1
start "OE" "C:\Program Files\Outlook Express\msimn.exe"
call :IsItemLaunchedSuccessful OE
goto :EOF
:LaunchItem2
start "Skype" "D:\Program Files\Skype\Phone\Skype.exe" /nosplash /minimized
call :IsItemLaunchedSuccessful Skype
goto :EOF
:LaunchItem3
Start "QQ" "D:\Program Files\Tencent\QQ\QQ.exe"
call :IsItemLaunchedSuccessful QQ
goto :EOF
:LaunchItem4
start "BaiduHi" /min "D:\Program Files\baidu\Baidu Hi\BaiduHi.exe"
call :IsItemLaunchedSuccessful BaiduHi
goto :EOF
:LaunchItem5
start "Thunder" /min "D:\Program Files\Thunder Network\Thunder\Thunder.exe"
call :IsItemLaunchedSuccessful Thunder
goto :EOF
:LaunchItem6
start "Tudou" /min "D:\Program Files\Tudou\iTudou\iTudou.exe"
call :IsItemLaunchedSuccessful Tudou
goto :EOF
:LaunchItem7
start "MSN" /min "C:\Program Files\MSN Messenger\msnmsgr.exe"
call :IsItemLaunchedSuccessful MSN
goto :EOF

:IsItemLaunchedSuccessful
set /p =Launching %1<nul & title Launching %1
if ERRORLEVEL 0 (
for /l %%i in (1,1,4) do (
ping -n %ShortDelay% 127.1>nul
set /p =.<nul
)
)
for /l %%i in (1,1,32) do set /p =%BackSpace%<nul
echo %1 Launched
title %1 Launched
ping -n %ShortDelay% 127.1>nul
goto :EOF
::::::::::::::::::::::::::::::::
```
