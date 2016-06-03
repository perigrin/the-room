---
layout: post
Title: How do you do this the right way?  
Author: Chris Prather
Date: 2004-08-06 22:11:47
---

# How do you do this the right way?
So I'm working with Java at work now. A little applet that has several tabbed panes of similar "View" objects. 

The code I'm modeling it off of has several If/Then/Else blocks wrapping cut-n-pasted code which is slightly altered for each class. I want to replace this with something much more intelligent and generic. Something like this:
<pre>
private void loadGenericView (String objClass, String label) {
        String msg;
        try {
            Class c = Class.forName(objClass);
            DataView view = (DataView) c.newInstance();
            tabbedPane.add(view, label);
        }
        catch (Exception e) {
           msg = "Catastrophic Error: " + e.getMessage();
           JOptionPane.showMessageDialog(null, msg, label,
                                         JOptionPane.ERROR_MESSAGE);
       }
    }
</pre>

But that doesn't work. Is there a right way to do this? All the references I've seen to "Factory" objects keep creating objects of a specific pre-known class. I know how to solve it in perl (basically like this but with slightly less verbosity).


