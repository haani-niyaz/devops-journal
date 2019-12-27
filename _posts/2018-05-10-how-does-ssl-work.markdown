---
layout: post
title:  "How Does SSL Work?"
date:   2019-01-02
author: "Haani Niyaz"
tags: 
 - ssl
 - tutorial
---

## Overview

The sequence diagram below illustrates the different parties involved in setting up SSL certificates and how the encrypted channel is established.

![How Does SSL Work](css/images/how-does-ssl-work.png){:class="img-responsive"}


## Cerificate Generation Process

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
# Make the mysite CSR available to CA - equivant of sending the CSR file to CA
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


## Certificate Structure

High level certificate structure although not comprehesive touches something interesting facts.

{% highlight bash %}
Data
 Version # version of X509 certificate standard (Not SSL version)
 Serial number # unique identifier. Changes with updates.
 Signature Algorithm # used by CA to sign the certificate
 Issuer # who issues the ceritifcate
 Validity # period of validity
 Subject # the organization
 Subject public Key # the org's public key is CSR
 X509 Extensions # additional features of certificate
  SAN: # alternate domain names the certificate is valid for
  Authority Information Access # Acess to CA information if it is not present in the trust store.
  OCSP # Protocol to validate the certificate in the event the owner has revoked it

Signature
 Signature Algorithm
 Signature
{% endhighlight %}

- A ceriticate is made up of a section called **Data** and **Siganture**.
- This Data section is the CSR data (provided by the organization) that is hashed and signed by the CA's private key. 
- The resulting Signature is attached to the certificate.
- Web server provides the certificate to the browser on initial request.


### What is a Digital Signature?


![Digital signature](css/images/digital-signature.png){:class="img-responsive"}


## Troubleshooting

#### View certificates 

{% highlight bash %}

$ openssl s_client -connect google.com:443 < /dev/null

{% endhighlight %}


#### Get readable certificate output

{% highlight bash %}

$ openssl s_client -connect google.com:443 < /dev/null | openssl x509 -in /dev/stdin -text -noout

{% endhighlight %}


