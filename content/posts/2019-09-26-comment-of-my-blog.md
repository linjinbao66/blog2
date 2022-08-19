---
title: 博客加入评论功能
tags:
  - 教程
  - 博客
categories:
  - 教程
date: 2019-09-26
---

## 2019-09-26-个人博客加入评论功能

## 个人博客加入评论功能

实现思路： 使用第三方存储，**白嫖**美滋滋

按照[姑苏流白](https://blog.hgtweb.com/)的教程，我使用[Valine](https://valine.js.org/)的评论系统

## 配置Valine

1. 注册 登录账号 略过
2. 建立应用 配置`应用keys`，配置项：`AppID`， `AppKey`

## 修改博客配置

1. 修改`config.toml`配置

加入以下内容：

```markup
# Valine.
# You can get your appid and appkey from https://leancloud.cn
# more info please open https://valine.js.org
  [params.valine]
    enable = true
    appId = 'V6YqPB0nmoIcjHC9P5CXBMSB-gzGzoHsz'
    appKey = 'hfkA0t8piT3MiYnUx936Msyd'
    notify = false  # mail notifier , https://github.com/xCss/Valine/wiki
    verify = false # Verification code
    avatar = 'mm'
    placeholder = '说点什么吧...'
    visitor = true
```

1. 创建`comments.html`

路径`themes\LeaveIt\layouts\partials\comments.html`

```markup
{{- if .Site.Params.valine.enable -}}
  <!-- id 将作为查询条件 -->  
  <div id="vcomments"></div>
  <script src="//cdn1.lncld.net/static/js/3.0.4/av-min.js"></script>
  <script src='//unpkg.com/valine/dist/Valine.min.js'></script>

  <script type="text/javascript">
    new Valine({
        el: '#vcomments' ,
        appId: '{{ .Site.Params.valine.appId }}',
        appKey: '{{ .Site.Params.valine.appKey }}',
        notify: '{{ .Site.Params.valine.notify }}', 
        verify: '{{ .Site.Params.valine.verify }}', 
        avatar:'{{ .Site.Params.valine.avatar }}', 
        placeholder: '{{ .Site.Params.valine.placeholder }}',
        visitor: '{{ .Site.Params.valine.visitor }}'
    });
  </script>
  {{- end -}}
```

1. 引入`comments.html`

路径`themes\LeaveIt\layout\s_default\single.html`

```markup
<div class="post-comment">
          {{ if ( .Params.showComments | default true ) }}
                 {{ if ne .Site.DisqusShortname "" }}
                     {{ template "_internal/disqus.html" . }}
                 {{ end }}
          {{ end }}

          {{ partial "comments.html" . }}        //加入这一句
</div>
```

1. 效果图如下

![&#x6548;&#x679C;&#x56FE;](https://raw.githubusercontent.com/linjinbao666/resources/master/amromPicture/%E8%AF%84%E8%AE%BA%E5%8A%9F%E8%83%BD.png)
