---
layout: post
title: Yet Another mTLS Tutorial
excerpt: "Is the internet full of mTLS tutorials? I'm not sure. Anyway, here's another one."
modified: 2021-03-04T21:19:25-04:00
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

Alright, it's time to get into some hands-on learning. Let's start with our backend.

## Reflect Thine Headers

```go
package main
import (
	"fmt"
	"net/http"
)
func main() {
	http.HandleFunc("/headers", headers)
	http.ListenAndServe(":6969", nil)
}
func headers(w http.ResponseWriter, req *http.Request) {
	for name, headers := range req.Header {
		for _, h := range headers {
			fmt.Fprintf(w, "%v: %v\n", name, h)
		}
	}
}
```

If you're unfamiliar with [Go](https://golang.org/) here's what the above code does:
- starts a http server on port [6969](https://i.pinimg.com/474x/32/c6/db/32c6db68576a9535537cdc0ce82a3a1f.jpg)
- responds with any headers sent to it

Feel free to implement this in your language of choice if you don't have the Go toolchain set up. Once you have your backend running, let's move on to the frontend.

## Curl It

```bash
curl -H "heyo: mayo" http://localhost:6969/headers
```

What, did you think we were going to write some HTML and JS code? Do I look like I'm in a masochistic mood? [Curl](https://curl.se/) is good enough for our needs. If you don't have it installed, google (or your choice of search engine) is your friend.

If you managed to run that curl command and your backend has been running, you should get a response like this:
```
User-Agent: curl/7.64.1
Accept: */*
Heyo: mayo
```

As expected, we've had our headers reflected back to us. So far so good.

We _could_ go ahead and implement mTLS right there in our backend but in a robust production system you want your concerns to be separated. Our backend shouldn't have to worry too much about TLS protocol stuff, so let's move all of that to our [reverse proxy](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/).

For your choice of reverse proxy there's always goodl old [Nginx](https://www.nginx.com/) which has a straightforward way to [configure mTLS](https://smallstep.com/hello-mtls/doc/server/nginx). If you're especially old you might even find yourself reaching for [Apache](https://httpd.apache.org/) where you can also [configure mTLS](https://blog.behrang.org/2019/08/11/ssl-mututal-authentication-apache-http-client.html).

But we're hip and cool so we're gonna be using the (relatively) new kid on the block [Traefik Proxy](https://traefik.io/traefik/). I'm sure you can figure out how to [install it](https://doc.traefik.io/traefik/getting-started/install-traefik/), by yourself. Go do that and come back once you've read up on [Traefik's configuration](https://doc.traefik.io/traefik/getting-started/configuration-overview/) management.

## Traefiking

Back? Figured out how to load [static](https://doc.traefik.io/traefik/getting-started/configuration-overview/#the-static-configuration) and [dynamic](https://doc.traefik.io/traefik/getting-started/configuration-overview/#the-dynamic-configuration) configuration in Traefik? Good, so here's what we'll be using.

> **Note**: I'm using version 2.2.1, the latest (at the time of this writing) is 2.4.x, but I don't think there should be a big difference in our experiences if you use 2.4.x

### Static Config

```toml
## Static configuration
[log]
level = "DEBUG"

[api]
dashboard = true
debug = true
insecure = true

[entryPoints]
[entryPoints.web]
address = ":8089"

[entryPoints.websecure]
address = ":443"

[providers]
[providers.file]
directory = "path/to/dynamic-config/directory"

[accessLog]
[accessLog.fields]
defaultMode = "keep"

[accessLog.fields.names]
defaultMode = "keep"

[accessLog.fields.headers]
defaultMode = "keep"
```

Here's what's happening:
- We're starting a dashboard we can view all our configurations
- We're starting a web listener (regular http) at port 8089
- We're starting a secure listener (https) at port 443
- We're providing the _directory_ where our dynamic config is stored (you can have a ton of dynamic config files and they all get merged into the config)

Alright, straightforward. Let's look at the dynamic config, now.

### Dynamic Config

```toml
[http]

[http.routers]

[http.routers.traeifk-api]
rule = "Host(`traefik.local`)"
service = "api@internal"

[http.routers.mtls-tut]
rule = "Host(`server.mtls-tut.local`)"
service = "mtls-tut"

[http.services]
[http.services.mtls-tut.loadBalancer]
[[http.services.mtls-tut.loadBalancer.servers]]
url = "http://127.0.0.1:6969/"
```

Here's what's happening:
- We're defining a [_router_](https://doc.traefik.io/traefik/routing/routers/) to route any requests that match the [_rule_](https://doc.traefik.io/traefik/routing/routers/#rule) _Host(`server.mtls-tut.local`)_. Any requests that arrive with the host value set to `server.mtls-tut.local` will get picked up bu the router `mtls-tut`. The same is done for the traefik api (which is used by its dashboard).
- We're defining a [_service_](https://doc.traefik.io/traefik/routing/services/) called `mtls-tut` which load-balances to our backend server running at port [6969](https://i.imgflip.com/4k39g9.jpg). In our router configuration we've defined this service as the one that should be serving all requests for `server.mtls-tut.local`

Now start Traefik however you prefer (docker, binary, whatever), make sure it can access both the static and dynamic files and you've pointed it to pick up the static config. If you did everything right Traefik will start up and print logs.

```
INFO[0000] Configuration loaded from file: /Users/shak/installs/traefik/traefik.toml
INFO[2023-02-04T10:01:47+01:00] Traefik version 2.2.1 built on 2020-04-29T18:09:41Z
DEBU[2023-02-04T10:01:47+01:00] Static configuration loaded {"global":{"checkNewVersion":true},"serversTransport":{"maxIdleConnsPerHost":200},"entryPoints":{"traefik":{"address":":8080","transport":{"lifeCycle":{"graceTimeOut":10000000000},"respondingTimeouts":{"idleTimeout":180000000000}},"forwardedHeaders":{},"http":{}},"web":{"address":":8089","transport":{"lifeCycle":{"graceTimeOut":10000000000},"respondingTimeouts":{"idleTimeout":180000000000}},"forwardedHeaders":{},"http":{}},"websecure":{"address":":443","transport":{"lifeCycle":{"graceTimeOut":10000000000},"respondingTimeouts":{"idleTimeout":180000000000}},"forwardedHeaders":{},"http":{}}},"providers":{"providersThrottleDuration":2000000000,"file":{"directory":"/Users/shak/projects/mtls-example/traeifk/dynamic","watch":true}},"api":{"insecure":true,"dashboard":true,"debug":true},"log":{"level":"DEBUG","format":"common"},"accessLog":{"format":"common","filters":{},"fields":{"defaultMode":"keep","names":{"defaultMode":"keep"},"headers":{"defaultMode":"keep"}}}}
...
...
INFO[2023-02-04T10:01:47+01:00] Starting provider *file.Provider {"directory":"/Users/shak/projects/mtls-example/traeifk/dynamic","watch":true}
INFO[2023-03-04T10:01:47+01:00] Starting provider *traefik.Provider {}
DEBU[2023-03-04T10:01:47+01:00] Configuration received from provider file: {"http":{"routers":{"mtls-tut":{"service":"mtls-tut","rule":"Host(`server.mtls-tut.local`)"},"traeifk-api":{"service":"api@internal","rule":"Host(`traefik.local`)"}},"services":{"mtls-tut":{"loadBalancer":{"servers":[{"url":"http://127.0.0.1:6969/"}],"passHostHeader":null}}}},"tcp":{},"udp":{},"tls":{}}  providerName=file
```

Traefik started with the static config we provided and then loaded the dynamic config. Let's make sure the dashboard is up. By default Traefik starts the dashboard on port [8080](http://localhost:8080)

If you see this page

![image](https://user-images.githubusercontent.com/5138570/110094067-af563f00-7d9b-11eb-96ce-12d82cadaf9f.png)

Traefik is up and running. If we go to `/dashboard/#/http/routers/mtls-tut@file`

![image](https://user-images.githubusercontent.com/5138570/110095621-656e5880-7d9d-11eb-96d8-54aa3e8dc790.png)

We can confirm that our desired configurations were picked up by Traefik. Note the empty **TLS** and **Middlewares** sections. We'll be filling those soon.

The final piece of this first part of the puzzle is modifying our `/etc/hosts` file and adding a new entry:
```
127.0.0.1 server.mtls-tut.local
```

Let's try curl again.
```bash
curl -H "heyo: mayo" http://server.mtls-talk.local:8089/headers
```

Note that we are now accessing the http entrypoint of our reverse proxy and not our backend. Your response should look something like:
```
X-Real-Ip: 127.0.0.1
Accept-Encoding: gzip
Heyo: mayo
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: server.mtls-tut.local:8089
X-Forwarded-Port: 8089
X-Forwarded-Proto: http
User-Agent: curl/7.64.1
Accept: */*
```

All those extra headers are telling us that Traefik is successfully routing our request to our backend at port [6969](https://camo.githubusercontent.com/42cef8be0203e843fc1c25b4be92bb6314721e180e3c23cfd1019387d2de628e/687474703a2f2f69302e6b796d2d63646e2e636f6d2f70686f746f732f696d616765732f6f726967696e616c2f3030302f3435302f3135342f3832302e6a706567).

Phew! All that work and we haven't even stared on the TLS part of things.

## TLS Time

First we're going to need a CA. Now, you _could_ go to the trouble of getting certificates online. If you have two domain names you own, getting certificates from [Let's Encrypt](https://letsencrypt.org/) is an inexpensive solution. But for our tutorial we'll stick with out local resoueces. Let's make a CA!

### Break Open that Bottle of openssl

Making your own CA is [surprisingly simple](https://gist.github.com/fntlnz/cf14feb5a46b2eda428e000157447309) (getting others to trust it is a different matter). You'll need [openssl](https://github.com/openssl/openssl) which _should_ already be installed on your platform of choice but if it isn't, you can do that bit yourself.

Once you have it, it's time to make the most important decision so far: what are you going to call your CA? I'll keep things simple for myself and call mine `mtls-tut.ca`. Not too imaginative but it's been a long week. The first thing we need to do now that we've decided a name is to create the _[Root](https://www.thesslstore.com/blog/root-certificates-intermediate/) [Certificate](https://www.kinamo.be/en/support/faq/what-are-root-and-intermediate-ssl-certificates)_. The _Root Certificate_ is a **Big Deal™**. This is the certificate that will be acting as-- you guessed it-- the root of all certificates issued by our CA. Other certificates will draw their authority _from_ this Root Certificate. So let's begin.

First, we need a private key.
```bash
openssl genrsa \
    -out mtls-tut.ca.key 4096
```
The [private key](https://sectigo.com/resource-library/public-key-vs-private-key) is basically a really big random number. We need to keep this number secret. Most CAs don't even keep these keys online. If it can't be accessed via a network it (probably) can't be hacked. CAs take the security of their private keys _[very seriously]_(https://security.stackexchange.com/questions/24896/how-do-certification-authorities-store-their-private-root-keys). In our case, we can choose to be somewhat lackadaisical about the security of our private key. I'm going to store it in a directory called `mtls-tut` and pretend it is guarded by digital pitbulls.

![timothy-perry-G9_4owLR9b4-unsplash (1)](https://user-images.githubusercontent.com/5138570/110125634-d32d7b00-7dc3-11eb-9c2f-c8f27adb255d.jpg)

If you ran the comman you should see output like this:
```
Generating RSA private key, 4096 bit long modulus
..............................................................................................................++
.........................++
e is 65537 (0x10001)
❯ ls
mtls-tut.ca.key
~/mtls-tut ❯
```

With a key in hand we need to derive a [X509 certificate](https://blog.keyfactor.com/what-is-x509-certificate). I will let you read up on what the X509 format looks like. For our purposes we need to know that the certificate is a collection of fields such as _country_, _city_, _region_, some more, and most importantly
```bash
openssl req -x509 -new -nodes \
    -key mtls-tut.ca.key \
    -sha256 -days 1024 \
    -out mtls-talk.ca.crt
```
You will get an interactive prompt asking you to enter values for the fields I mentioned above.
```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) []:BE
State or Province Name (full name) []:Flanders
Locality Name (eg, city) []:
Organization Name (eg, company) []:
Organizational Unit Name (eg, section) []:
Common Name (eg, fully qualified host name) []:mtls-tut.ca
Email Address []:
```
Most of it is informational except the _Common Name_ which is _very important_. Get that one right.

```
❯ ll
-rw-r--r--  1 shak  staff   1.7K  15 Mar 25:08 mtls-talk.ca.crt
-rw-r--r--  1 shak  staff   3.2K  15 Mar 25:03 mtls-tut.ca.key
```
We now have a _private key_ and a _x509 certificate_ for our CA. And that's enough to get us started on the next step: creating the server-side certificate. The process is _very similar_ to what we did for the CA certificates with one key difference. We'll start the same as before by generating a private key.

```bash
openssl genrsa -out server.mtls-tut.local.key 2048
```

And now we're going to derive a X509 certificate from this private key, _however_, we will sign this certificate with the CA key and cert we generated earlier. To do this we first need a [CSR (Certificate Signing Request)](https://www.sslshopper.com/what-is-a-csr-certificate-signing-request.html). A CSR is a _request_ that is signed by the private key of the server and submitted to the CA. If the CA is happy with it, they will issue a _certificate_ based on it.

```bash
openssl req -new -sha256 -key server.mtls-tut.local.key \
    -subj "/C=BE/ST=AN/O=Sentiance/CN=server.mtls-tut.local" \
    -out server.mtls-tut.local.csr
```

The CSR includes the same fields we used to generate the certificate of our CA. In essence they serve the same function: identification of an entity, be it a server or organization. In the CA's case there isn't any higher authority to authenticate them. They _are_ the highest digital authority. For a server certificate, the CA is the higher authority that authenticates them. Since we have our own CA, we can sign our own certificate.

```bash
openssl x509 -req -in server.mtls-tut.local.csr \
    -CA mtls-talk.ca.crt -CAkey mtls-tut.ca.key \
    -CAcreateserial -out server.mtls-tut.local.crt \
    -days 500 -sha256
```
If we break down the command a bit, we'll see that we're taking the CSR as an input, accept the CA key and cert and apart from a few options, outputing a X509 certificate.

Phew again! We have our CA and we managed to get some server certificates out of it. Can we see some TLS action already?!
