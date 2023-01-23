---
layout: post
title: "How I set up a social media site from an iPad for $50 a month without managing servers."
author: "Chris Prather"
categories:
tags: docker, akkoma, render
date: 2023-01-22
image: https://live.staticflickr.com/65535/51548431435_f2dd2d084f_5k.jpg
---

## The Pledge

Over the last month I've set up a Mastodon compatible service for about $50 per
month without a single server to manage.

The easiest solution is a hosted solution. There are several decent hosting
providers out there. Check out [Join Mastodon's
list](https://docs.joinmastodon.org/user/run-your-own/), not all of them are
the same. They start out at well under our price but a few of them cap out at
nearly double our target. Several of them are limited to Europe or their
websites are entirely in Japanese, which for my monoglot self is a non-starter.

I don't know how things at nerdfight.online will scale and how soon I'd go from
starter tier to "call us for a quote". I also wanted direct access to the
codebase, or at least the database for some of the features I'm planning on. So
I decided to host my own.

Most people who choose to host their own use a VPS or server of some kind.
Hachyderm.io for example is famously hosted on a rack in Nova's house, and
Digital Ocean has a one click startup service. For reasons I talked about
[three years
ago](https://chris.prather.org/Building-a-Shopify-App-With-Perl-Part-1.html)
and because when it comes to systems work I have the attention span of an
autist with ADHD (hello late diagnosis), I wanted something serverless.

Since that post last three years ago I've  found
[render.com](https://render.com). I really like how quickly and easily it is to
set up something and get it onto the web. This isn't paid advertising for them,
but as of right now I'm very happy with what they've provided.

I'll save you the month or so of reading, testing, throwing away, and
restarting effort I went through and just describe the fastest way, in my
opinion, to get started with a setup like mine.

Before we start I should stress that setting up a website that publishes user
generated content is *much* more complicated than the technical infrastructure.
Denise at [dreamwidth](https://denise.dreamwidth.org/91757.html) has a _really_
good post just covering the bare basics of what's necessary. Suffice to say if
you're planning on going past a single user, you will _need_ to do some
research and probably have a lawyer you feel comfortable calling if you get
into trouble.

I've spent a decade running my little company, my risk tolerances are different
than someone else. I'm fully aware there's a lot I don't know that could
potentially make my life harder, and I have a lot of privilege I can lean
on. I'm still being very cautious as I move forward.

If you're just wanting a single user instance, well you're the only one who can
get yourself into trouble. Things should be moderately okay, but forewarned is
forearmed.

## The Turn

### What you'll need

    * 1 device with a web browser and internet access

### Instructions

First Register a domain name. [Render](https://render.com) has
[instructions](https://render.com/docs/configure-namecheap-dns) specifically
for namecheap they also have instructions for
[cloudflare](https://render.com/docs/configure-cloudflare-dns). I use
[namecheap](https://namecheap.com) for this because I've been using them for
years already. You can follow [Render's
instructions](https://render.com/docs/configure-namecheap-dns) configuring the
domain name.

Next on [GitHub](https://github.com) we're going to create new repo to hold
your instance's configurations and files. We'll be using one of the templates
I've set up to make things easier.

I tried [GoToSocial](https://github.com/superseriousbusiness/gotosocial)
([template](https://github.com/Tamarou/gotosocial-render)) first. The
documentation says that it's in an alpha state, and the documentation is not
wrong, but it's a very solid alpha. If I were planning on staying a single user
instance I might have stayed with GoToSocial. I was able to use the starter
size service for both the web service and database with GoToSocial without
issue, keeping my costs to around $14 per month.

Having run up against GoToSocial's edges a few too many times, I switched to
[Akkoma](https://akkoma.social)
([template](https://github.com/Tamarou/akkoma-render)). It had a similar
reputation for minimal resource consumption. Once I'd gotten myself set up and
followed a couple hundred accounts, I started bumping up against the half gig
limit of the starter level. Upgrading the database and the web services to
standard fixed that issue and I haven't encountered anything like it again
since.

We will be focusing on Akkoma, if you want to go with GoToSocial the steps are
similar but not identical. You're welcome to reach out to me for help.

Clone the template repository. You can do this from the browser by just
clicking `Use this Template` and selecting `Create a new repository`. When it
asks, name your repository after the domain name you picked in step one. After
you have forked the template you'll need to edit a few files, you can click the
file and then the pencil icon to open it in an editor in the browser.

Edit the `render.yaml` file and change the domain `akkoma.example.com` to your
own domain. You only *need* to change it in `domains:` and the `DOMAIN`
environment variable, but I recommend changing it everywhere, including the
place like the name `db.example.com`. This way everything is grouped together
in the Render dashboard. You'll also want to change the `repo:` to the URL for
your GitHub repo.

Save and commit the changes to `render.yaml` and make sure they're pushed to
the GitHub repo. If you're editing in the browser this is all one step. Now we
will tell Render about our repository.

Sign up for an account with
[render.com](https://dashboard.render.com/register). If you use GitHub
authentication it will automatically associate your Render account with your
GitHub account and make the next steps easier. Click on
[Blueprints](https://dashboard.render.com/blueprints) at the top of the
dashboard and click [`New Blueprint
Instance`](https://dashboard.render.com/select-repo?type=blueprint). Connect to
the GitHub repository you created in the previous steps.

Render will take a few minutes to parse the `render.yaml` file and create the
services that host your instance. After a few minutes you can check the
dashboard, everything should be deployed. You'll need to manually create your
first user before you can log in.

On the dashboard click on the your akkoma service, mine is named
`ack.nerdfight.online`. Click the `Shell` tab on the left side. This will bring
up a window that allows you type directly into the docker container's command
prompt. Type the following, be sure to replace `USER` with the username you
prefer and `email@exmample.com` with your email address:

```shell
  cd /opt/akkoma
  bin/pleroma_ctl user new USER email@example.com --admin
```

Now you should be able to go to the domain you've configured in your browser
and log in. You can Toot your success.

For nerdfight.online I've also configured S3 media storage and a static site
for documentation. Check the configuration for
[nerdfight.online](https://github.com/tamarou/nerdfight.online) or reach out to
me if you'd like to see how either of these work.

In the future I would like to expand things a few ways. Eventually I suspect
I'll want to add a CDN for static assets, and a dedicated search engine to
improve the speed of everything. I also have a hand full of custom services I'd
like to install, like a nightly digest and the Fake Nerd Fight Friday script.

## The Prestige

What about price? How does it compare for the all-in-one hosted solutions?
Pretty well I think. In theory you could stand up a server for free … but it
wouldn't really be very useful. Render's free tier Postgres service is
automatically suspended after 90 days, they give you 14 days after that to take
a backup before deleted the service. They allow you to create a new free tier
database so in theory you could restore from the backups every 90 days … but
while testing your backups is important, building your entire app around it
every three months seems suboptimal.

I paid $20.17 for December and I'm currently projected to spend $37.72 for
January. I expect my costs to level out around $40 - $50 per month, but I'm
still in the middle of testing. There isn't a lot of hard data out there about
scaling and prices, but I'm pretty sure that I'll be able to hit a several
dozen active users before I have to look at scaling up the services further.

Notice that because every website we've worked with here has a web based data
entry, we never needed anything more than a web browser. You could easily
deploy a new service from an iPad in the middle of the ocean. During initial
testing I set up a second instance while debugging what turned out to be a
firewall issue with on the cruise ship I was on. I had only taken my iPad with
me.
