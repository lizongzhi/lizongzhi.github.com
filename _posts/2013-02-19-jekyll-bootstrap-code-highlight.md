---
layout: post
title: " jekyll-bootstrap添加代码高亮 "
category: UI
tags: [jekyll , 代码高亮]
description: |
 在利用jekyll中写博客时难免需要在文章中用到代码，为了让文章排版清晰，代码突出显示，需要将代码部分高亮显示。在jekyll中代码高亮显示的方法有许多种，这篇文章是添加css与js将文章中代码高亮显示。
---
##基本步骤##
1：修改`/includes/themes/twitter/default.html` , 添加几行代码
<pre class="prettyprint linenums">
&lt;link href="{\{ ASSET_PATH }\}/google-code-prettify/prettify.css" rel="stylesheet" type="text/css" media="all"&gt;
&lt;script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js"&gt;&lt;/script&gt;
&lt;script type="text/javascript" src="{\{ ASSET_PATH }\}/google-code-prettify/prettify.js"&gt;&lt;/script&gt;
</pre>

2: 再添加一段js, 让其生效, js代码如下
<pre class="prettyprint linenums">
!function ($) {
    $(function(){
        window.prettyPrint && prettyPrint();
    });
}(window.jQuery);
</pre>

3:写文章的时候而不是用markdown语法, 直接将要高亮的代码用pre标签包围

<pre class="prettyprint linenums">
&lt;pre class="prettyprint linenums"&gt;
    &lt;!-- 包围其中 --&gt;
&lt;/pre&gt;
</pre>

##需要注意几个问题##

1: 所有尖括号都要用html中对应的符号来代替，比如`<`要用`&lt;`代替，`>`要用`&gt;`代替，`&`要用`&amp;`代替。这不光在代码中如此，在blogger正文中好像也是如此。

2: `<pre>`标签无法自动换行，如果需要自动换行需要修改`<pre>`样式
<pre class="prettyprint linenums">
pre{
  white-space:pre-wrap; /* css3.0 */
  white-space:-moz-pre-wrap; /* Firefox */
  white-space:-pre-wrap; /* Opera 4-6 */
  white-space:-o-pre-wrap; /* Opera 7 */
  word-wrap:break-word; /* Internet Explorer 5.5+ */
}
</pre>

