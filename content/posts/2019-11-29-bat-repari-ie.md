---
title: 批处理解决IE浏览器设置问题
tags:
  - dos
date: 2019-11-29
---

## 批处理解决IE浏览器设置问题

功能集合.bat

```text
@echo off
title    ★  IEl浏览器设置，请先确保浏览器是IE11 ，否则会有设置无法生效  ★
echo   正在启用IE的 ActiveX控件、JAVA脚本、活动脚本，请稍候...
ping 127.0.0.1 -n 2 >nul 2>nul
set bl=0
:setreg
if "%bl%"=="3" goto ex
set regpath=HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Zones\%bl%
cls
echo   本脚本可快速启用IE的 ActiveX控件、JAVA脚本、活动脚本
echo   正在进行 ZONE%bl% 的设置...
:启用“activeX控件”“JAVE小程序脚本”“活动脚本”
@reg add "%regpath%" /v "1001" /d "0" /t REG_DWORD /f
@reg add "%regpath%" /v "1004" /d "0" /t REG_DWORD /f
@reg add "%regpath%" /v "1200" /d "0" /t REG_DWORD /f
@reg add "%regpath%" /v "1201" /d "0" /t REG_DWORD /f
@reg add "%regpath%" /v "1400" /d "0" /t REG_DWORD /f
@reg add "%regpath%" /v "1402" /d "0" /t REG_DWORD /f
@reg add "%regpath%" /v "1405" /d "0" /t REG_DWORD /f
@reg add "%regpath%" /v "1609" /d "0" /t REG_DWORD /f
@reg add "%regpath%" /v "1804" /d "0" /t REG_DWORD /f
@reg add "%regpath%" /v "2300" /d "0" /t REG_DWORD /f
set /a bl=%bl%+1
goto setreg
:ex
cls

echo IE的 ActiveX控件、JAVA脚本、活动脚本，设置完成

echo 正在添加受信任站点,关闭弹窗功能...
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Ranges" /v @ /t REG_SZ /d "" /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Ranges\Range1" /v "http" /t REG_DWORD  /d 00000002 /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Ranges\Range1" /v ":Range" /t REG_SZ /d "http://172.25.136.242:8080/jkqzwfw/" /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Ranges\Range2" /v "http" /t REG_DWORD  /d 00000002 /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Ranges\Range2" /v ":Range" /t REG_SZ /d "172.25.136.*" /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\he.boss\bd" /v "http" /t REG_DWORD  /d 00000002 /f
reg add "HKCU\Software\Microsoft\Internet Explorer\New Windows" /v "PopupMgr" /t REG_SZ /d "no" /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Zones\2" /v "2201" /t REG_DWORD  /d 00000000 /f
echo   设置完毕，请打开IE测试一下是否正常，若不正常，建议关闭所有IE再次运行本脚本。


ping 127.0.0.1 -n 8 >nul 2>nul


echo 受信任站点已添加,关闭弹窗功能完成

echo  开始执行word预览插件注册程序，请确保OfficeControl.ocx文件在当前目录下

:: BatchGotAdmin  
:-------------------------------------  
REM  --> Check for permissions  
>nul 2>&1 "%SYSTEMROOT%\system32\cacls.exe" "%SYSTEMROOT%\system32\config\system"  

REM --> If error flag set, we do not have admin.  
if '%errorlevel%' NEQ '0' (  
    echo Requesting administrative privileges...  
    goto UACPrompt  
) else ( goto gotAdmin )  

:UACPrompt  
    echo Set UAC = CreateObject^("Shell.Application"^) > "%temp%\getadmin.vbs"  
    echo UAC.ShellExecute "%~s0", "", "", "runas", 1 >> "%temp%\getadmin.vbs"  

    "%temp%\getadmin.vbs"  
    exit /B  

:gotAdmin  
    if exist "%temp%\getadmin.vbs" ( del "%temp%\getadmin.vbs" )  
    pushd "%CD%"  
    CD /D "%~dp0"  
:--------------------------------------  
regsvr32 OfficeControl.ocx

echo ocx插件注册成功

echo 开始将开发区政务中心放进收藏夹

reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\dl.pconline.com.cn" /v http /t REG_DWORD /d 0x00000002 /f
for /f "tokens=2,*" %%i in ('reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" /v "Favorites"') do Set fpath=%%j
for /f "tokens=2,*" %%i in ('reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" /v "Desktop"') do Set dpath=%%j
set uname=开发区在政务中心.url
set uurl=http://172.25.136.242:8080/jkqzwfw
echo [InternetShortcut] > "%fpath%\%uname%"
echo URL=%uurl%>> "%fpath%\%uname%"
echo [InternetShortcut] >  "%dpath%\%uname%"
echo URL=%uurl%>>  "%dpath%\%uname%"
echo "收藏完成"

echo 开始安装360急速浏览器，请确保路径下存在360cse_11.0.2251.0.exe文件，如果不需要安装可以关闭

::安装360急速浏览器
start /wait .\360cse_11.0.2251.0.exe /s

exit
```

安装IE11.bat

```text
@echo off
for /f "skip=2 tokens=3 delims= " %%i in ('reg query "HKLM\SOFTWARE\Microsoft\Internet Explorer" /v Version') do set a=%%i
set b=%a:~0,1%
set c=%a:~2,1%
set d=%a:~2,2%

echo IE Version:%d%

echo 判断IE版本

if %b% EQU 11 (echo IE Version:%b%&& echo 您的浏览器无需升级 && goto next) else goto next1
:next1
if %b% LSS 11 (echo IE Version:%b%&& echo 您的浏览器需要升级，请确认可以升级，如果不可升级请关闭窗口，并上报原因)

echo 请确认Y/N？

set/p option=请输入你的选择:

if "%option%"=="N" (exit) else goto next2 
:next2
if "%option%"=="Y" (echo 开始安装IE11)

echo 开始检测电脑系统位数
if "%PROCESSOR_ARCHITECTURE%"=="x86" goto x86 
if "%PROCESSOR_ARCHITECTURE%"=="AMD64" goto x64 
exit 
:x64 
echo 您的电脑是64位的，需要安装64位的IE11
::安装IE1
start /wait .\IE11-Windows6.1-x64-zh-cn.exe /s /update-no

echo 安装IE11 完成，请关闭窗口

pause 

:x86 

echo 您的电脑是32位的，需要安装32位的IE11
::安装IE11

start /wait .\IE11-Windows6.1-x86-zh-cn.exe /s /update-no
echo 安装IE11 完成，请关闭窗口

pause
```
