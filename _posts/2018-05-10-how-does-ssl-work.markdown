---
layout: post
title:  "How Does SSL Work?"
date:   2019-01-02
author: "Haani Niyaz"
tags: 
 - ssl
 - tutorial
---

### Overview

The sequence diagram below illustrates the different parties involved in setting up SSL and how the encrypted channel is established.

![How Does SSL Work](css/images/how-does-ssl-work.png){:class="img-responsive"}


### Steps to Generate Keys

#### Generate Server Private Key

{% highlight bash %}
$ mkdir -p /etc/ssl/mysite.com && cd /etc/ssl/mysite.com
$ openssl genrsa -out mysite.key 4096
Generating RSA private key, 4096 bit long modulus
.............................++
.......................................................++
e is 65537 (0x10001)
{% endhighlight %}

The `mysite.key` is the private key  which contains both the `private` and `public` key pairs.

#### Generate the CSR from Private Key

{% highlight bash %}
$ openssl req -new -key mysite.key -out mysite.csr
ou are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:AU
State or Province Name (full name) []:VIC
Locality Name (eg, city) [Default City]:MELBOURNE
Organization Name (eg, company) [Default Company Ltd]:Acme
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:acme.dev
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
{% endhighlight %}

The information provided above will be used by the CA to verify your details.


#### CA Signs CSR

Let's assume the role of the CA and produce a self signed certificate instead.

{% highlight bash %}
$ mkdir /etc/ssl/ca.com && cd /etc/ssl/ca.com
# Generate CA private key
$ openssl genrsa -out ca.key 4096
# Make the mystie csr available to CA
$ sudo cp /etc/ssl/mysite.com/mysite.csr .
# Create the certificate
$ openssl x509 -req -days 365 -in mysite.csr -signkey ca.key -out server.crt
{% endhighlight %}


#### Installing the Certificate

With some web servers or proxy servers you cannot load the certificate and private key seperately. In these instance you can create a `pem` file which contains the certificate and the private key.

{% highlight bash %}
# Get self signed certificate
$ cp server.crt /etc/ssl/mysite.com
$ cd /etc/ssl/mysite.com
$ cat server.crt mysite.key > server.pem
{% endhighlight %}