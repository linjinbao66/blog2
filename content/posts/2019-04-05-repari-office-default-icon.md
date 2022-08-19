---
title: 修复Office文件默认图标
tags:
  - Office
  - bat
  - 小工具
categories:
  - 小工具
date: 2019-04-05
---

## 修复office文件默认图标

```text
rem 修复office文件默认图标.bat

:: office图标文件路径，请根据自己电脑上的安装目录设置
set officepath=C:\Program Files\Microsoft Office\root\vfs\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\

:: 修复word图标
reg add "HKCR\Word.Document.8\DefaultIcon" /ve /t REG_SZ /d "%officepath%WORDICON.EXE,1"
reg delete "HKCR\Word.Document.8\DefaultIcon" /v .ksobak

reg add "HKCR\Word.Document.12\DefaultIcon" /ve /t REG_SZ /d "%officepath%WORDICON.EXE,13"
:: 此行为删除wps残留注册表文件，下同
reg delete "HKCR\Word.Document.12\DefaultIcon" /v .ksobak


:: 修复EXCEL图标
reg add "HKCR\Excel.Sheet.8\DefaultIcon" /ve /t REG_SZ /d "%officepath%XLICONS.EXE,28"
reg delete "HKCR\Excel.Sheet.8\DefaultIcon" /v .ksobak

reg add "HKCR\Excel.Sheet.12\DefaultIcon" /ve /t REG_SZ /d "%officepath%XLICONS.EXE,1"
reg delete "HKCR\Excel.Sheet.12\DefaultIcon" /v .ksobak

:: 修复POWERPNT图标
reg add "HKCR\PowerPoint.Show.8\DefaultIcon" /ve /t REG_SZ /d "%officepath%PPTICO.EXE,17"
reg delete "HKCR\PowerPoint.Show.8\DefaultIcon" /v .ksobak

reg add "HKCR\PowerPoint.Show.12\DefaultIcon" /ve /t REG_SZ /d "%officepath%PPTICO.EXE,10"
reg delete "HKCR\PowerPoint.Show.12\DefaultIcon" /v .ksobak


:: 清理图标缓存，重启explorer
taskkill /f /im explorer.exe
CD /d %userprofile%\AppData\Local
DEL IconCache.db /a
start explorer.exe
```
