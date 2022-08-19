---
title: cloudflare worker使用
date: 2021-08-21
---


部署ariang到cloudflare workers

```code
mkdir -p ariang/public
cd ariang/public
wget https://github.com/mayswind/AriaNg/releases/download/1.1.7/AriaNg-1.1.7.zip
unzip AriaNg-1.1.7.zip
rm -f AriaNg-1.1.7.zip
cd ..
wrangler init --site ariang

wrangler config
wrangler publish
```

wrangler.toml配置如下：

```toml
name = "ariang"
type = "webpack"
account_id = "1e752e20ad8ac6b74a275b64f272ef76"
route = 'ariang.linjinbao.tk/*'
zone_id = '02b23e0f77225f085046a173e2402a25'
usage_model = ''
compatibility_flags = []
workers_dev = true
site = {bucket = "./public",entry-point = "workers-site"}
```

`public`目录是静态文件目录
