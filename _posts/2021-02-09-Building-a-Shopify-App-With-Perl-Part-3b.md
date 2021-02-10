---
layout: post
title: "Building a Shopify App with Perl & JavaScript & Kubernetes (Part 3b)"
author: "Chris Prather"
categories:
tags: shopify, docker, k8s, k3s, lxc
date:  Tue 09 Feb 2021 10:00:00 PM EST
image: https://live.staticflickr.com/65535/49985572657_01cb2a78a0_b.jpg
---

## The Preface

My wife runs an after-school pottery program in various elementary schools
around town. Parents sign their kids up for the classes, and then my wife's
staff go out to the various schools and teach basic ceramics to the kids. All
the existing student registration systems for this are both wildly expensive,
and they suck. She asked me if I could solve at least one of those problems
for her.

I thought maybe I'd write about the process. If you're discovering this for the
first time I refer you to [part 1][1] where I describe the goals and the
process in more detail, and talk about how I set up the integration with
Shopify; [part 2][2] where I talk about starting the front end JavaScript; and
[part 3][3] where I discuss setting up a Kubernetes cluster in my LXC based
environment.

[1]: https://chris.prather.org/Building-a-Shopify-App-With-Perl-Part-1.html
[2]: https://chris.prather.org/Building-a-Shopify-App-With-Perl-Part-2.html
[3]: https://chris.prather.org/Building-a-Shopify-App-With-Perl-Part-3a.html

## Wherever anything lives, there is, open somewhere, a register in which time is being inscribed.

So while going about learning how to use kubernetes, the fact that it requires
a registry didn't escape my notice. For my personal stuff I don't mind just
using [Docker Hub](https://hub.docker.com) but for projects that are a bit more
sensitive, like this one it would be nice to just have a private registry.
Docker makes it pretty easy to setup a registry but I figured I should write
down the process as we go along.

First I create a new container for the registry. I have a profile (`docker`)
that sets the permissions on the LXC containers so that docker can happily run
inside the container.

```yaml
config:
  security.nesting: "true"
  user.user-data: |
    #cloud-config
    package_upgrade: true
    apt:
      preserve_sources_list: true
      sources:
        docker.list:
          source: deb [arch=amd64] https://download.docker.com/linux/ubuntu $RELEASE stable
          keyid: 7EA0A9C3F273FCD8
    packages:
        - docker-ce
        - docker-ce-cli
        - containerd.io
description: Docker LXD profile
devices:
  eth0:
    nictype: bridged
    parent: lxdbr0
    type: nic
  root:
    path: /
    pool: default
    type: disk
name: docker
```

A few tricks to this, first `security.nesting` needs to be enabled so that
docker has permissions to run. Second the `user.user-data` is a cloud-init yaml
script that installs docker. By default the container is attacked to the
private bridged network that lxc sets up. Using `lxc profile edit docker` will
let you paste the above file into your lxc settings.

With that setup we're ready to launch the registry.

```shell
lxc launch -p docker ubuntu: docker-registry
```

We'll need to wait a few minutes for lxc to finish running the cloud-init
script, but when we're done we should have a new lxc container with docker
installed. We can test this with:

```shell
lxc exec docker-registry -- docker run hello-world
```

If that outputs successfully then we're good to move on to installing the
registry. Installing a registry is pretty simple, really at it's base it's
`docker run -d --restart=always --name registry registry`, but that leaves us
with a bare container running in a private network inside your LXC. If all of
your other services are on the same LXC bridge network, then that may be enough
for you.

## History is only the register of crimes and misfortunes

If however you need to access it from outside the LXC network we'll need to do
a few things. First let's start with exposing the LXC container to the public.
We're gonna use the `macvlan` profile we setup in  [part 3][3] to expose our
container to a public IP.

```shell
lxc profile add docker-registry macvlan
lxc restart docker-registry
```

We will need to update the guest OS in the container to use the new IP. I'm
using the latest LTS of ubuntu so this means editing the netplan files to look
something like:

```yaml
network:
    version: 2
    ethernets:
        eth0:
            dhcp4: no
            addresses:
                - 192.168.0.3/24 # USE YOUR REAL IP
            gateway: 192.168.0.1 # USE YOUR REAL GATEWAY
            nameservers:
                addresses:
                    - 1.1.1.1
                    - 8.8.8.8
```

Getting this configuration into *my* container just required the following.

```shell
lxc exec docker-registry -- bash -l
vi /etc/netplan/50-cloud-init.yaml # paste in the above YAML
netplan generate
netplan apply
```

If everything worked correctly, you should be able to ping from the above host,
and ping the above host from an outside address. Next we'll need to create an A
record for this address, I configured `docker.tamarou.com` via the Web GUI at
my domain registrar and it's beyond my ability to advise you on how to do it
yourself.

Assuming that the A record is live, and you can now setup an SSL certificate
for the docker registry using Letsencrypt.

```shell
lxc exec docker-registry -- bash -l
docker run -it --rm --name certbot -p 80:80 \
    -v "/etc/letsencrypt:/etc/letsencrypt" \
    -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
    certbot/certbot certonly
```

If this works correctly it will kick off an interactive session asking you a
few questions. You want to choose `1` for `Spin up a temporary webserver` and
provide it with the domain you set up with the A record.

## I don't know any celebrated people that register in a big way who aren't unique

Securing communication is great, but still anybody can push anything into our
repo. By default docker only allows `htpasswd` authentication, requiring you to
drop to a web-server like Nginx to get anything fancier. That's fine for our
purposes here, integrating with your own system is left as a lemma for the
reader.

Create a new `htpasswd` file somewhere,

```shell
htpasswd -cB /etc/htpasswd perigrin
```

If we need to add another user (let's say for our [scrum master](https://twitter.com/trg404) it's as simple as running the following and supplying a password:

```shell
lxc exec docker-registry -- htpasswd -B /etc/htpasswd trog
```

## I don't express a lot of things that I feel; I kind of register things

Now we can setup the docker registry to point at this. I went and created a
startup script so I was sure to get all these arguments correct, you're welcome
to attempt to type it all correctly or just copy-and-paste.

```shell
docker run -d --name registry --restart=always \
	-p 443:5000 \
	-e REGISTRY_HTTP_ADDR=0.0.0.0:5000 \
	-e REGISTRY_HTTP_TLS_CERTIFICATE=/etc/letsencrypt/live/docker.tamarou.com/fullchain.pem \
	-e REGISTRY_HTTP_TLS_KEY=/etc/letsencrypt/live/docker.tamarou.com/privkey.pem \
    -e REGISTRY_AUTH=htpasswd \
    -e REGISTRY_AUTH_HTPASSWD_REALM="Docker Registry Realm" \
    -e REGISTRY_AUTH_HTPASSWD_PATH="/etc/htpasswd"
	-v /etc/letsencrypt:/etc/letsencrypt \
	-v /var/lib/docker/registry:/var/lib/registry \
    -v /etc/htpasswd:/etc/htpasswd \
	registry
```

## Carpe per diem &ndash; seize the check

Assuming everything worked we can push a test image to the repo this way. On a
machine that docker is installed on (could be the lxc container we just built,
`lxc exec docker-registry -- bash -l` is your friend) do the following:

```bash
docker run hello-world
docker tag hello-world docker.tamarou.com/hello-world # replace with your domain
docker push docker.tamarou.com/hello-world
```

This should report that the image was pushed. From anywhere with `curl` you can do the following:

```bash
curl -X GET https://docker.tamarou.com/v2/_catalog # again make this match your own domain
```

It will return a JSON listing of the repositories, hopefully now including `hello-world`.


## A good accountant is a debit to is profession

Much of this post was based on a series of  posts by [exoscale][exoscale]. I've
just re-organized thins a bit so that the process is cleaner and discussed how
all of this interacts with my LXC setup.

[exoscale]: https://exoscale.com/syslog/securing-private-docker-registry/

---
The title image is [Container](https://flic.kr/p/2ja48he) by [Johannes](https://www.flickr.com/photos/p1hde/).


