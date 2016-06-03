---
layout: post
Title: Wakeup, we're here...  
Author: Chris Prather
Date: 2005-10-15 17:55:36
---

# Wakeup, we're here...
I mentioned to Nacho that I would post this.

I got Alice (my iBook running Tiger) updating it's Network Location based off the ssid of the network it joins to when it wakes up from suspend.

I used a script (update-location.sh) from <a title="macosxhints - Change location automatically based on network" href="http://www.macosxhints.com/article.php?story=2005010613401823">this page</a>.

I just had to munge the script a little, the following line:
<pre>
#look up the status 
location=`grep -i "IT Airport" .wlans | awk -F"\t" '{print $2}'`
</pre>
should read
<pre>
#look up the status 
location=`grep -i $wlan .wlans | awk -F"\t" '{print $2}'`
</pre>

It's all very neat.
