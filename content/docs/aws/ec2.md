---
title: EC2
---

# EC2 FAQ
---

### What are the two types of storage available for EC2 instances?

- EBS
- Instance storage (ephemeral)

### How are you charged for EC2 instances?

If it is running, you will be charged.

### What is a *reserved* instance?

An EC2 instance that runs for a specific period of time. This is paid upfront unlike an on-demand instance.

### What are the types of virtualization types available?

- **HVM AMIs**: Guest OS is run on hypervisor.
- **PV AMIs**: Guest is run on host hardware that does not have support for virtualization.

### Can I use an AMI from any region?

No, AMIs are region specific.

### By default what type of IP address does an EC2 instance get?

Private IP. This is used for internal comms inside the VPC.

### What is a public IP?

An IP that is reachable from the internet.

### What is an elastic IP?

With an EIP you can attach a **static** public IP address to an EC2 instance that was originally created only with a private IP. EIPs can be leveraged for fail over of your application by rapidly mapping the EIP to another instance.

### What if the instance already had a publc IP?

It will replace the public IP for as long as the elastic IP is attached.

### What is a quick way to introduce bootstrapping commands via the AWS console?

Within the *Advanced Detatils* section of the EC2 instance GUI configuration, you can add shell scripts.

### How to view the commands used to bootstrap the EC2 instance?

{{< highlight bash >}}
$ curl http://169.254.269.254/latest/user-data
{{< / highlight >}}

### How to view the metadata of the  the EC2 instance?

{{< highlight bash >}}
$ curl http://169.254.269.254/latest/meta-data
{{< / highlight >}}


## Load Balancing FAQ

### What types of load balancers are available?

#### Classic LB (Legacy)

Simple balancing to ec2 instances. This is much like a VIP that routes to a pool of reverse proxies.


#### Network LB (OSI L4)

- Requests are recieved and sent at the Network layer resulting in higher performance
- Supports static IP addressing for the LB
- dynamic host port mapping

#### Application LB (OSI L7)

- path based routing
- host based routing
- route to multiple ports
- dynamic host port mapping

Configuration requires setting up **listeners** for client requests and a **target group** of target instances.


### How to allow TCP `src` address when using AWS ELB?

Amazon ELB supports Proxy Protocol. This will allow a **TCP header** that can later be read by applications which support it.

This must be done via the API. First, create the policy as shown below:

{{< highlight bash >}}
$ aws elb create-load-balancer-policy --load-balancer-name YOUR-ELB-HERE --policy-name EnableProxyProtocol  --policy-type-name ProxyProtocolPolicyType --policy-attributes AttributeName=ProxyProtocol,AttributeValue=True
{{< / highlight >}}

Then, apply it like so:

{{< highlight bash >}}
$ aws elb set-load-balancer-policies-for-backend-server --load-balancer-name YOUR-ELB-HERE --instance-port 25 --policy-names EnableProxyProtocol
{{< / highlight >}}

You can verify by running:

{{< highlight bash >}}
$ aws elb describe-load-balancers --load-balancer-name YOUR-ELB-HERE
{{< / highlight >}}

You should see `EnableProxyProtocol` listed under policies.

Read more [here](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/enable-proxy-protocol.html#proxy-protocol)
