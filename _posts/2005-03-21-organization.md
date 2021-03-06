---
layout: post
Title: Organization  
Author: Chris Prather
Date: 2005-03-21 17:43:58
---

# Organization
So I got mentioned on planet RDF via <a title="jo's journal" href="http://frot.org/devlog/0015_gtd.html">jo's journal</a>

<blockquote>I recalled how highly dngnand had spoken of [Getting Things Done] last year and sworn by the Life Balance PDA version; we've talked with perigrin about writing an irc lifebalance bot based on dngnand's description of the operations, and sketched prototypes...</blockquote>

Which leads me to wanting to actually talk about what I want/need to organize and why I'm writing a lifebalance bot (and wiki) thing. My requirements aren't quite as comprehensive as dngnands. I simply need:
<ul>
<li> A Hierarchical List of all Tasks/Ideas.

Something to keep track of tasks in an organized fashion. ShadowPlan is great for this on my palm, and there are a thousand plus organizers out there for building hierarchical lists. (Vim|Notepad) probably being the simplest and fastest. 

<li> A Simple List of what I should do next.

LifeBalance has a reccomendation system for what I should be doing now, based upon where I'm located, what I've been working on, and how important that task is to over all goals. I found this list to be the most appealing part of LifeBalance and why I started using it over ShadowPlan. 

<li> Multiple Points of Entry

Having used LifeBalance on both the Desktop and Palmtop I found that being able to quickly add tasks/ideas wherever I happened to be when I thought of them was immeasurably useful. At work I could just flip to the desktop app and add an idea/task whatever, and when I was out I could use the slower entry on the palm knowing that it would all be synced up when I was at work again. It would be nice to have access to this process no matter where I was.

</ul>

These three are must haves in any system I use. LifeBalance satisfied them, but the cost was gonna be about a hundred dollars (USD) in semi-proprietary software (LifeBalance has an XML output, but that's not entirely useful). Also there are some features I'd like to have that neither LifeBalance nor ShadowPlan have. 

<ul>
<li> Ability to share my Todo list with others.

I found several occassions where I wanted to show other people portions of my ToDo list, but that simply wasn't possible with desktop applications. There was no easy way to sync them with my website. Granted both Shadow and LifeBalacne export XML, and I run an AxKit server ... but I'd have to write XSLT transformations, and upload the files. Wouldn't it be better if the application itself were internet accessible?

<li> Automatic notification of events. 

Often I found I wouldn't update my LifeBlance often enough, and it wouldn't alert me to events that needed to happen *now*. Palm Desktop has alerts with the calendar, and I could export events that way or use Outlook or something. I may do that eventually, but none of those things happen by default with timed events, and I couldn't set them up to happen by default.  Also if I happen to forget my Palm at home, or I'm not at the office ... I'm out of luck. With an internet accessible appliance, anything that is remotely connected to the net can work. If I'm not on a computer, the Bot can (page|SMS) me (Work provides me with a pager. Why not use the bloody thing.)


<li>Integration with things I haven't thought of yet

Having my stuff in a easily accessible format means I can come up with new ways later to integrate my ideas and be able to patch them into the old system (or re-write the old system to allow that). Closed sources like ShadowPlan and LifeBalance are locked-in. Without help from Jeff, or the people at Llamagraphics, I can't really extend their functionality. Now Jeff is one of the coolest people ever, and if you have a good idea that would help lots' of shadow users ... he'll try to implement it. But what if I needed somethign that only benefitted me? There's no reason why Jeff should write that.

</ul>

So that's sorta a brain dump of what I've been thinking about in space of Organization and writing Yet Another Organizer App. The sad irony is right now I haven't had time to finish my Organizer app, because my organization tools have slid (I let LifeBalance expire, and haven't gone back to Shadow yet).

So we'll see where we end up from here. Hopefully I'll have a Organizer app to post about soon.
