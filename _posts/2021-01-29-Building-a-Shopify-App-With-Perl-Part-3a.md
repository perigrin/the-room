---
layout: post
title: "Building a Shopify App with Perl & JavaScript & Kubernetes (Part 3a)"
author: "Chris Prather"
categories:
tags: shopify, docker, k8s, k3s, lxc
date:  Mon 02 Mar 2020 10:49:17 AM CST
image: https://live.staticflickr.com/3263/2383423665_c1a6b71cba_z.jpg
---

## The Preface

My wife runs an after-school pottery program in various elementary schools
around town. Parents sign their kids up for the classes, and then my wife's
staff go out to the various schools and teach basic ceramics to the kids. All
the existing student registration systems for this are both wildly expensive,
and they suck. She asked me if I could solve at least one of those problems
for her.

I thought maybe I'd write about the process. If you're discovering this for
the first time I refer you to [part 1][1] where I describe the goals and
the process in more detail, and talk about how I set up the integration with
Shopify. There is also [part2][2] where I talk about starting the front end
JavaScript.

[1]: https://chris.prather.org/Building-a-Shopify-App-With-Perl-Part-1.html
[2]: https://chris.prather.org/Building-a-Shopify-App-With-Perl-Part-2.html

## All the world's a stage

So in [part 1][1] I mentioned using a modern deployment stack based around
docker. I presumed that this would eventually lead to me using one of the
container platforms out there that involves using Kubernetes. I've never
actually worked with K8s before so I figured I should set up a staging
environment. (It didn't hurt that my work project was starting to involve K8s
too).

I knew that Rancher had a fairly simple Kubernetes implementation named
[K3s][k3s], that could easily be deployed on Raspberry Pis and other easily
obtainable platforms. So I decided to set one up in my persona dev environment.
One small trick, my personal dev environment ls based around [LXD][lxd], which
can be a little tricky when it comes to things like Docker. It becomes all "Yo
dawg, I heard you liked containers" quickly. I'm writing this blog post so you
know I was successful, but whether this is a cautionary tale of madness and woe
is up to you gentle reader.

[k3s]: https://rancher.com/docs/k3s/latest/en/
[lxd]: https://linuxcontainers.org

## Mewling and puking in the nurses arms

I found [these instructions](k8s-lxc) which seemed exactly what I wanted, well
sort of. I wanted to use ubuntu or better alpine and not Debian, and I expected
to use the [k3s installer][k3s-installer] but I mean basically the same. And Oh
yeah my host system for LXC was a fairly old Ubuntu 16.04 LTS system with
whatever `lxd` came via `apt` (this is foreshadowing pay attention).

So when I followed those instructions, it ended up not working for some reason.
I kept getting errors when k3s would start up about how it couldn't read
`/proc` no matter what I did. Unfortunately in my efforts to debug this, I
somehow decided that upgrading the host to 20.04 would be a good idea, and then
when it *still* didn't work I rebooted the machine and  it didn't come back.
Never do a system upgrade at 2am &hellip; on a machine in a different timezone.

The next morning, when the good folks at the hosting provider had performed a
system wipe and restored to a fresh copy of Ubuntu 20.04 LTS I started again.

[k8s-lxc]: https://github.com/corneliusweig/kubernetes-lxd/blob/master/README-k3s.md
[k3s-installer]: https://get.k3s.io/

## Full of strange oaths

When things had recovered, I had found some more instructions (like
[this one][k3s-lxd]) and with everything I managed to get a working solution.

First create a new profile. In lxc, profiles are basically like configuration
roles, they are all applied in the order specified and merged into the final
container config before it's started.

```bash
lxc profile create k3s
lxc profile edit k3s
```

Then I copied in the following yaml:

```yaml
config:
  boot.autostart: "true"
  linux.kernel_modules: ip_vs,ip_vs_rr,ip_vs_wrr,ip_vs_sh,ip_tables,ip6_tables,netlink_diag,nf_nat,overlay,br_netfilter,vxlan
  raw.lxc: |
    lxc.apparmor.profile=unconfined
    lxc.mount.auto=proc:rw sys:rw cgroup:rw
    lxc.cgroup.devices.allow=a
    lxc.cap.drop=
  security.nesting: "true"
  security.privileged: "true"
description: ""
devices:
  aadisable:
    path: /sys/module/nf_conntrack/parameters/hashsize
    source: /sys/module/nf_conntrack/parameters/hashsize
    type: disk
  aadisable1:
    path: /sys/module/apparmor/parameters/enabled
    source: /dev/null
    type: disk
  aadisable3:
    path: /dev/kmsg
    source: /dev/kmsg
    type: disk
  aadisable4:
    path: /sys/fs/bpf
    source: /sys/fs/bpf
    type: disk
name: k3s
```

This loads some required/recommended kernel modules, it sets some raw lxc
commands including mounting `/proc` and `/sys` as read-write. It disables some
of the container security protections (`security.nesting` and
`security.prividleged` being true *disables* some of the container
separations). Finally it sets up a few devices that k3s expects to exist that
we need to fake in the container.

Now we can create a new container. To save myself some trouble I
went with ubuntu for now, I'll bite off getting alpine to work at a later date.

`lxc launch -p k3s ubuntu: k3s-master`

Next I created a little install script (based on the one from [k3s-on-lxd][k3s-lxd]

```bash
apt update && apt install openssl curl -y
curl -sl get.k3s.io | INSTALL_K3S_EXEC="--snapshotter native" sh -
sleep 30
```

The first time I ran this I used the defaults, but the container had issues
using the overlayfs snapshotter. I ended up adding the configuration manually
after the fact but the environment variable there will set things up the way I
have them working.

We push that script into the container

`lxc file push install_k3s.sh k3s-master/tmp/`

and then tell the container to run it (by default as the container's `root`)

`lxc exec k3s-master -- bash /tmp/install_k3s.sh`

When this is done it should successfully install k3s. You can check by logging
into k3s-master and asking for a list of the current kubernetes nodes.

```bash
lxc exec k3s-master -- bash -l
kubectrl get node
```
[k3s-lxd]: https://github.com/ruanbekker/k3s-on-lxd

## That ends this strange eventful history

This got k3s up and running for me but it was trapped inside a container using
NATed networking on a private bridge. I could access everything from the host
machine but I actually do all my work on my laptop these days[^1]. To access
things from the outside world I needed to expose the container somehow.
Traditionally I've done this with a firewall rule that set up an port forward,
which is fiddly and cumbersome. There has to be a better way.

Googling around I discovered that lxc supports a `macvlan` network stack. This
basically allows a single network card to pretend to be two network devices at
layer 2. (If you understood that without Googling the OSI stack you were doing
better than I was when I started this.) The container would get it's own IP
address and show up on the network like it was a real boy, the downside is that
the host wouldn't be able to communicate with it directly without jumping
through some hoops. Since I rarely use a shell on the host directly that's not
really a problem.

To set this up I created another profile in lxc:

```yaml
config: {}
description: ""
devices:
  eth0:
    nictype: macvlan
    parent: eno1
    type: nic
name: macvlan
```

This sets up a new NIC device in the container, says that it's macvlan and that
it uses the parent's `eno1` device. After that I edited the netplan in the
container to use a static IP address in the range my hosting provider had given
me, but by default it would have grabbed an IP from DHCP.

And like that the container is exposed to the internet.

### Sans teeth, sans eyes, sans taste, sans everything.

So I haven not yet deployed the app to this Kubernetes cluster. I'm still
looking at how that works. I'm investigating how
[blimpup.io](https://blimpup.io) would work with a cluster like this too for
future development and testing. I would have happily traded money for their
services, but they're sadly no longer offering their services for money.

I also ran into a block with the JavaScript a few days after the last post and
I need to circle back around to re-learning meiosis or some other full
application state maintenance. Matt Trout has suggested a few other things to
look at in that realm as well. We shall see.

---

The title image is [G-Wiz](https://flic.kr/p/4CBEPt) by [Steve Jurvetson](url=https://www.flickr.com/photos/jurvetson/).

[^1]: My daily driver laptop is a Pinebook Pro, which only has 4GB hardwired into the SoC. Firefox with more than 6 active tabs tends to bring it to it's knees so I try to do my work in lxc containers remotely.
