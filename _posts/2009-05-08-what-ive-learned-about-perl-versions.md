---
layout: post
Title: What I've Learned about Perl Versions  
Author: Chris Prather
Date: 2009-05-08 21:33:15
---

# What I've Learned about Perl Versions
So in a discussion about the version numbers in `MooseX::POE` the following wisdom was imparted upon me:
<pre>
14:35 <@lbr> DBIC had: $VERSION = '0.08010';
14:35 <@rafl> just always quote version numbers
14:36 <@rafl> and don't make them shorter than a previous version
14:37 <@lbr> and people with weird package-systems are happy
14:50  * perigrin will make sure to start quoting the version number;
14:51 <@perigrin> lbr: note that once 0.2x settles down I'm gonna release it as 1.0
14:51 <@perigrin> or at least 1.000
14:51 <@rafl> also $VERSION = eval $VERSION if you do dev releases
14:51 <@perigrin> rafl: I haven't really yet.
14:51 <@perigrin> but someone should write this down
14:51 <@rafl> well volunteered
</pre>

Consider this written down. Now to clean up the modules I have that don't follow this practice.
