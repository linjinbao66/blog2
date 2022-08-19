---
title: 一个gitlab ci文件
tags: ["gitlab"]
date: 2021-08-12
---

一个golang项目典型的gitlab ci文件

```yaml
image: golang:latest

before_script:
  - export GOPROXY=https://goproxy.io,direct

stages:
    - build
    - release
    - note

build:
    stage: build
    script:
      - go build -o dt-release
    artifacts:
      paths:
        - ./
        
release:
    stage: release
    script:
      - ./dt-release release
    dependencies:
      - build

note:
    stage: note
    script:
      - ./dt-release note
    dependencies:
      - build

```
