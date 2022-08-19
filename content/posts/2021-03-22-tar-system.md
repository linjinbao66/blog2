---
title: tar打包现有的系统
date: 2021-03-22
---

选择一个系统，在根目录下将已有系统打包成tar文件：

```code
tar -cvpf /tmp/system.tar --directory=/ --exclude=proc --exclude=sys --exclude=dev --exclude=run --exclude=boot --exclude=/opt/software .
```

导入方式一：

```code
cat system.tar | docker import - redhat:6.5
```

导入方式二：

使用docker build

Dockerfile

```code
FROM scratch
ADD system.tar /
CMD ["sh"]
```

docker build -t linjinbao66/euler:0.0.1 .
