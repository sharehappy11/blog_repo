---
title: Next主题-文章目录锚点问题修复
date: 2017-12-09 11:03:27
tags: [博客建站,Hexo,Next主题]
---

Next主题在浏览文章时，会在侧边栏展示文章目录，能帮助我们快速回忆，快速定位文章内容，是一个很实用的功能。文章目录功能默认是开启的，可以在/themes/next/\_config.yaml中进行配置。

**软件版本**：hexo-v3.4.2, Next-v5.1.3.

但在个人配置Next主题时，目录中 **【只有数字编号有锚点，而文字上无锚点】**，使用起来非常的不方便，如下图：
![锚点错误展示](http://p0mqjeixa.bkt.clouddn.com/markdown-img-paste-20171211112752513.png)

在官网没有找到解决方案，最后从源码上解决了这个问题，简单记录一下修复方法。

next主题的侧边栏渲染源码在：/next/layout/\_macro/sidebar.swig中142行：
```html
{% if display_toc and toc(page.content).length > 1 %}
<!--noindex-->
  <section class="post-toc-wrap motion-element sidebar-panel sidebar-panel-active">
    <div class="post-toc">

      {% if page.toc_number === undefined %}
        {% set toc = toc(page.content, { "class": "nav", list_number: theme.toc.number }) %}
      {% else %}
        {% set toc = toc(page.content, { "class": "nav", list_number: page.toc_number }) %}
      {% endif %}
      {% if toc.length <= 1 %}
        <p class="post-toc-empty">{{ __('post.toc_empty') }}</p>
      {% else %}
        <div class="post-toc-content">{{ toc }}</div>
      {% endif %}

    </div>
  </section>
<!--/noindex-->
{% endif %}
```
发现文章目录是利用toc()方法直接生成的，方法在 blog/node_modules/hexo/lig/plugins/helper/toc.js中进行定义：
```js
function tocHelper(str, options) {
  options = options || {};
  var headingsSelector = ['h1', 'h2', 'h3', 'h4', 'h5', 'h6'].slice(0, headingsMaxDepth).join(',');
  var headings = $(headingsSelector);
  ...
  headings.each(function() {
    var level = +this.name[1];
    var id = $(this).attr('id');
    var text = $(this).html();   

//问题就在这个地方，获取到的html内容是<a href="xxx" class="headerlink" title=xxx></a>标题, 直接拼接在html中，导致出现问题。
//增加下一行代码，将html标签过滤掉，只留下文本内容，即可
text = text.replace(/<[^>]+>/g, '');

    lastNumber[level - 1]++;
    for (i = level; i <= 5; i++) {
      lastNumber[i] = 0;
    }
    ...
    result += '<span class="' + className + '-text">' + text + '</span></a>';
```

按照如上方法修改后，在运行hexo g重新生成网页，就可以看到修改后的效果啦。
