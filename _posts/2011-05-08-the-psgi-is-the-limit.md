---
layout: post
Author: Chris Prather
Date: 2011-05-08T03:34:45

---
# The PSGI is the Limit

Miyagawa has recently linked to a [bunch](http://bit.ly/j4q5oJ) [of](http://perlmonks.org/index.pl?node_id=903569) [forums](http://bit.ly/lk9ZnO) where he has encountered a lot of push back against PSGI. The argument basically goes "We're using CGI/mod_perl/FastCGI just fine and it works for us, why should we change?" Let me tell you a couple of stories.

For the last year and a half I consulted on a large web application. It was a closed application but based on Catalyst and used mod_perl as a deployment. The site had a decent amount of traffic but with a combination of nginx as a front-end proxy, proper caching, and a CDN the systems we had were handling things just fine. 

The lead systems guy was often in the mood to experiment with alternatives to see if we could get better performance out of the cluster of machines that ran the site. Because we happened to be using Catalyst, and thus Catalyst::Engine, we suggested that he try moving to FastCGI and dropping Apache out of the mix. So he took a box out of commission, re-jiggered it and tried it. There was a 30% reduction in the system load. 

30% … nearly 1/3 as much load … that meant by just re-tooling our application stack with no actual changes to the code base we could eliminate 1 out of every 3 servers. We saved the company 30% of our hosting costs by simply changing the deployment scenario all because with Catalyst::Engine we could try a new deployment scenario with no effort.

This is where PSGI saves people money. PSGI is the generalization of Catalyst::Engine and HTTP::Engine and several other projects over the years (yes dating all the way back to Apache::Request and Apache::PerlRun which emulated a CGI environment in mod_perl). It de-couples the application deployment logic from the business logic and allows you to perform trivial changes to the deployment.

The second story I wanted to tell you was this. Way back in the day I was playing around learning POE. Someone had written a POE server, `POE::Component::Server::HTTP` and there was a page on the POE wiki talking about how to run CGI applications under it. The code was incomplete because it didn't *really* implement the CGI environment, it just did the GET and POST stuff.
So I went and read the Specs for CGI (they're ancient-by-web-standards NCSA webserver documents) and sorted out which pieces needed to be glued into where.

I have to tell you it wasn't *difficult* but it certainly isn't the most elegant code I've written. The same basic idea was taken on as HTTP::Request::AsCGI and judging by the 14 releases that package has had over the last five years it's not exactly edge-case free[^1].  

PSGI as a specification is much simpler to implement for a Perl application than the CGI environment. PSGI is at it's core a subroutine that takes a HashRef and returns an ArrayRef.
Now I hear you crying out that CGI is just that easy, but really it's not. CGI is a pollution of environment variables and a system call (or something that fakes system call that by dumping it's output into STDIN). Tell me honestly as a programmer, which is easier: a subroutine dispatch, or faking a capture of the output of a system call?

So while CGI may be "good enough" for you (hopa!) there are cases where it's not "good enough" for the people implementing the tools. While you may never ever ever move off of CGI, FastCGI or mod_perl, that doesn't mean that something better won't ever be invented. It also doesn't mean that what you're using is the best possible solution for your application, but you'll never know that because you didn't see a reason to abstract your deployment in such a way that you can try out alternatives quickly and easily.

[^1]: Think that's not sufficiently bad? Look at the hoops people suggest you jump through for CGI support under nginx [http://wiki.nginx.org/SimpleCGI](http://wiki.nginx.org/SimpleCGI)
