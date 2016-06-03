---
layout: post
Title: FUN with XSLT  
Author: Chris Prather
Date: 2004-03-12 11:41:16
---

# FUN with XSLT
So I created yet another template engine today. But this time I created it in XSLT, and it takes xhtml as it's template format.

You don't get anything fancy, it just parses the HTML and replaces elements with the right id attribute with content from the source XML file. So far it's tested with AxKit and SAWA (both using LibXSLT)  and seems to work perfectly for what it does.

Check it out for yourself:

<ul>
<li> <a href="http://axkit.prather.org/page.xml.txt">The Source XML</a>
<li><a href="http://axkit.prather.org/styles/page.xsl">The Stylesheet</a>
<li><a href="http://axkit.prather.org/main.html">The Template</a>
<li><a href="http://axkit.prather.org/page.xsl">The Output</a>

XSLT is FUN!

