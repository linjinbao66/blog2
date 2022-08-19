---
title: golang交叉编译
date: 2021-08-20
---



Golang 支持在一个平台下生成另一个平台可执行程序的交叉编译功能。

## 1、Mac下编译Linux, Windows平台的64位可执行程序

```go
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build test.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build test.go
```

## 2、Linux下编译Mac, Windows平台的64位可执行程序

```go
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build test.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build test.go
```

## 3、Windows下编译Mac, Linux平台的64位可执行程序

```go
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build test.go
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build test.go
```

GOOS：目标可执行程序运行操作系统，支持 darwin，freebsd，linux，windows GOARCH：目标可执行程序操作系统构架，包括 386，amd64，arm
