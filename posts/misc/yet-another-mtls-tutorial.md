---
layout: post
title: Yet Another mTLS Tutorial
excerpt: "Is the internet full of mTLS tutorials? I'm not sure. Anyway, here's another one."
modified: 2020-03-04T21:19:25-04:00
categories: misc
tags: [tls, ssl, mtls]
comments: true
share: true
---

I've come across many mTLS tutorials on the internet but none that took me end-to-end in a satisfying fashion. That's what I hope to do in this tutorial. Although we'll be using a very simple golang server and traefik as our reverse proxy. Why those? Because that's what I had to work with and learn about, recently, and nothing quite cements an idea into your head than blogging about it.

# Let's Start with the Barebones

Okay, that's a bit of a lie. I won't explain what [TLS](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls) is or how it works. I am going to assume you already know some of the basics. Now if you know what TLS is, mTLS gets really easy to understand. So let's quickly recap how TLS works.

In a traditional TLS setup, used every time you see that green padlock icon in your browser's address bar, the browser is verifying the server's identity. When you go to a website you access it by using an address such as [badgerbadgerbadgerbadger.dev](https://badgerbadgerbadgerbadger.dev). If the website has TLS support (and if you encounter one in 2021 that _does not_ have TLS support, seriously, don't use it), the browser will ask it to send over its certificate in order to verify that the website is actually who it says it is. The website sends back such a certificate which has a [_CommonName_ (also knwown as a _Fully Qualified Domain Name_, or _FQDN_)](https://www.godaddy.com/garage/whats-a-fully-qualified-domain-name-fqdn-and-whats-it-good-for/) field which _should_ match the address you entered in the address bar. For `https://badgerbadgerbadgerbadger.dev` the returned certificate _should_ have a _CommonName_ of *badgerbadgerbadgerbadger.dev*. Your browser will see that the _CommonName_ and the name in the address matches and it will be satisfied that you are indeed visiting a website which says who it is.

"But wait!" I hear you say, "Couldn't anyone send back a certificate with a _CommonName_ field set to whatever?" Astute observation. Yes, indeed, anyone can create a certificate, attach whatever _CommonName_ they choose, and send it back to your browser...

...except, they can't.

That's where [_Certificate Authorities (CAs)_](https://support.dnsimple.com/articles/what-is-certificate-authority/) come in. These are organizations that have (allegedly) gone through rigorous audits and whose trustworthiness has been verified to an extent that if they say that a website is who it says it is, that's good enough for browsers to trust that website. And how does this process work, you say? We'll get to that when we get to the more hands-on section of this post.

# Ti(m)e to Reciprocate

Alright, so web browsers know who the server is. The server can be trusted. But can the browser be trusted? Maybe, maybe not. But to make our usecase more practical let's move away from the example of an end-user using a browser and move to two backend machines talking to each other. Let's imagine we have a traditional client-server architecture where _Machine A_ needs to get data from _Machine B_ at random intervals, and the way that _Machine B_ has decided it wants _Machine A_ to authenticate is Mutual TLS.

If you understood how a server indetified itself to the client by using a certificate trusted by a third party, you're already most of the way to understanding mTLS: it's kind of in the name.

In mTLS the server sends its certificate as usual, but the client has to send one too. Let's say _Machine B_ (the server) identifies itself as `machineb.com` and has a certificate with a matching _CommonName_, that's one side taken care of. In order for the other side (_Machine A_, the client) to do its part, it also needs to obtain a certificate. Now this certificate doesn't have to have a specific _CommonName_. It doesn't _have_ to be `machinea.com`. But if we're only using public CAs, the easiest thing for _Machine A_ to do is to prove to a CA that they own `machinea.com` and then be issued a certiciate for that _CommonName_. _Machine B_, on its end will know to expect requests from _Machine A_ authenticated by a certificate which has the _CommonName_ of `machinea.com`.

Are there other ways to do this? Sure. _Machine B_ could host its own CA and issue a certificate to _Machine A_. That way _Machine B_ knows exactly what to expect since it's the one that issued the certificate.

I'll end this section a link to a [really nice article which has some informative diagrams](https://medium.com/sitewards/the-magic-of-tls-x509-and-mutual-authentication-explained-b2162dec4401).

# Let's Get Our Hands Dirty
