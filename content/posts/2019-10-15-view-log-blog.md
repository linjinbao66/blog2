---
title: 博客加入访问统计
date: 2019-10-15
---

## 2019-10-15-博客加入访问计数

## 博客加入访问统计

使用valine的实现

## 步骤

1. 修改`comments.html`

将以下内容加入：

```markup
<p class="copyright-item">
    <span id="{{ .RelPermalink | relURL }}" class="leancloud_visitors" data-flag-title="{{ .Title }}">
        <span class="post-meta-item-text">Reading: 本文累计被阅读 </span>
        <span class="leancloud-visitors-count">1000000</span>
        <span class="post-meta-item-text">次</span>
    </span>
</p>
```

加入后内容如下：

```markup
{{- if .Site.Params.valine.enable -}}
  <!-- id 将作为查询条件 -->  
<p class="copyright-item">
    <span id="{{ .RelPermalink | relURL }}" class="leancloud_visitors" data-flag-title="{{ .Title }}">
        <span class="post-meta-item-text">阅读次数: 本文累计被阅读 </span>
        <span class="leancloud-visitors-count">1000000</span>
        <span class="post-meta-item-text">次</span>
    </span>
</p>
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
