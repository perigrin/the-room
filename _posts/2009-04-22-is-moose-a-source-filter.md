---
layout: post
Title: Is Moose a Source Filter?  
Author: Chris Prather
Date: 2009-04-22 16:12:26
---

# Is Moose a Source Filter?
No. 

<div class="thumbnail"><a href="http://skitch.com/perigrin/bcsk7/moose-is-not-a-source-filter"><img src="http://img.skitch.com/20090422-fy43ua53hw9dddk9b41h3nbsru.preview.jpg" alt="Moose is Not a Source Filter" /></a><br /><span style="font-family: Lucida Grande, Trebuchet, sans-serif, Helvetica, Arial; font-size: 10px; color: #808080">Uploaded with <a href="http://plasq.com/">plasq</a>'s <a href="http://skitch.com">Skitch</a>!</span></div>

The only thing in Moose's dist that *is* a source filter is `oose.pm` that I wrote to make writing Moose apps on the command line as easy as:

`perl -Moose -e'has foo => (is => q[ro]); Class->new(foo=>q[bar])->dump'`
