---
title: EC2
---

# EC2 FAQ
---

{{< tabs "ec2" >}}

{{< tab "Resiliency" >}}

Resilient within AZ.

{{< /tab >}}

{{< /tabs >}}

## General

### What are the two types of storage available for EC2 instances?

1. EBS
2. Instance storage (ephemeral)
    - Local to the AWS EC2 host the EC2 is spun up on.
    - EC2 instance stopped/started or a host failure can cause the instance relocate to another host.

### How is the primary network interface setup?

When an instance is provisioned into the subnet, the primary network interface that is created for the instance is actually mapped to physical hardware on the AWS EC2 host.

### What are the types of virtualization types available?

- **HVM AMIs**: Guest OS is run on hypervisor.
- **PV AMIs**: Guest is run on host hardware that does not have support for virtualization.

### Can I use an AMI from any region?

No, AMIs are region specific.


### What is an elastic IP?

With an EIP you can attach a **static** public IP address to an EC2 instance that was originally created only with a private IP. EIPs can be leveraged for fail over of your application by rapidly mapping the EIP to another instance.

#### What if the instance already had a publc IP?

It will replace the public IP for as long as the elastic IP is attached.

### What is Enhanced Networking?

Typically a host has a physical network card which
is shared as virtual NICs for each EC2 instance. 
This results in the OS having to translate communication between the instances
and physical NIC. When the host is under load it can experience
poor performance.

#### How is this fixed?
Present the physical NIC with a logical NIC per instance. There is minimal OS
involvement where the translation takes place between the logical
and physical NIC.

### Difference between Launch Configurations (LC) and Launch Templates (LT)?

**Background**

- LT & LC are responsible for determining "what" is launched not the 
"where" or "how"

- Define configuration for an instance in advance such as type, storage,
keypair, SG, userdata, role etc.

- Cannot be edited although LT has versions.

- AWS recommends using LT.


> LC is used for auto scaling groups. 
LT can also be used for ASGs and in 
addition, it can be used to launch EC2
instances via CLI or console.


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


### How to allow TCP `src` address when using AWS Classic LB?

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
