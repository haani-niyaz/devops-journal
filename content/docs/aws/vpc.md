---
title: VPC
---

# VPC FAQ
---

## VPC

### What is the relationship between route tables and subnets?

Each subnet in your VPC must be associated with a route table, which controls the routing for the subnet (subnet route table).

A route table can be associated with multiple subnets. However, a subnet can only be associated with one route table at a time. Any subnet not explicitly associated with a table is implicitly associated with the main route table by default.

### What is created by default when a VPC is created?

- NACL
- Security Group
- Route Table

{{< hint info >}}
**Tip**  
You should leave the Main route RT untouched and create a dedicate RT to set an IGW (with default route) because attaching a IGW route to the Main RT will make all subnets public. You can only have 1 RT per subnet.
{{< /hint >}}

### Why don't you get all 254 allocatable IPs with CIDR /24?

AWS reserved 5 IPs so you get 251. See more details [here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html).

In short:

1. Network address (e.g: 10.1.1.0)
2. Network address + 1 (e.g: 10.1.1.1) used by the VPC router
3. Network + 2 used for VPC DNS
4. Network + 3 reserved for future
5. Broadcast address


## VPC Endpoints

Typically you would need to provision an Internet Gateway to access AWS public services like S3, SQS etc. However, AWS provides a way to access public services privately via 2 types of Endpoint services; **Interface** and **Gateway**.

### Gateway

{{< hint info >}}
Provide private access to S3 and DynamoDB.
{{< /hint >}}

{{< tabs "vpc-endpoint-gateway" >}}
{{< tab "Summary" >}}

S3 and DynamoDB are public services to which you can enable a VPC to privately access them instead of assigning a public IP (e.g: to an EC2) or deploying a NAT Gateway.

This creates  prefix list in the route table for the S3 or DynamoDB destination. It is kept upto date by AWS. When you have a request destined for these resources the target is the Gateway Endpoint.
{{< /tab >}}

{{< tab "Resiliency" >}}

Resilient within region.

The Gateway Endpoint is not placed into a subnet but rather an entry is added to the RT.  It is HA across all AZs in a region.
{{< /tab >}}

{{< /tabs >}}

### How do you manage access control from the Gateway Endpoint to resources?

An **Endpoint Policy** can be setup to control which resources (e.g: buckets within S3) can be accessed from the Gateway Endpoint.

See an example [here](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-s3.html#vpc-endpoints-policies-s3).

### Does it support cross region?

No.

### Interface

{{< hint info >}}
Provides private access to all AWS services except for S3 and DynamoDB.
{{< /hint >}}

{{< tabs "vpc-endpoint-interface" >}}
{{< tab "Summary" >}}

It is an ENI with a private IP address from your subnet range that serves as an entrypoint to traffic destined to supported AWS services.

When you create an Interface Endpoint, AWS will generate an endpoint-specific
DNS hostname that you can use to communicate with services. If the private DNS option is enabled (*usually this is enabled by default*) the default DNS name for a service like `ec2.us-east-1.amazonaws.com` will resolve to the private IP addresses of the endpoint network interfaces in your VPC. This means an instance in a private subnet can continue to resolve the service public DNS hostname instead of having to use the endpoint-specific DNS hostnames.

See more details [here](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#vpce-private-dns).

{{< /tab >}}

{{< tab "Resiliency" >}}

Not resilient by default. Requires Interface Endpoint per subnet.
{{< /tab >}}

{{< /tabs >}}

### How do you manage access control from the Gateway Endpoint to resources?

An **Endpoint Policy** can be setup to control which resources (e.g: buckets within S3) can be accessed from the Gateway Endpoint.

See an example [here](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-s3.html#vpc-endpoints-policies-s3).

### Does it support cross region?

No.



## VPC Flow Logs

### Where do you appy this?

1. All interfaces within VPC
2. Interfaces within a subnet
3. Directly to an interface

### Are logs real time?

No. There is a delay between traffic entering and leaving.

### Can you inspect packet content?

No. Only metadata like ports, ip addresses etc.

It also does not record traffic on:

1. The instance metadata IP `169.254.169.254`
2. Time service `169.254.169.123`
3. DHCP
4. AWS DNS server

### Where is it stored?

You have the option of storing it in S3 or CloudWatch Logs.

## Internet Gateway

{{< hint info >}}
**Tip**
An IPv4 public IP is not assigned to the instance/OS but rather exists on the IGW which maintains a table of the instance private IP and public IP. It is a
form of static NAT'ing. This also means a public DNS provided for a public IP will resolve to the private IP within a VPC.
{{< /hint >}}


## NAT Gateway

### Does it have to exist in a public subnet?

a NAT gatway requires a route to get to the internet. This is available in 
the public subnet where a default route to IGW entry exists in the route table.

See more details [here](https://serverfault.com/questions/854475/aws-nat-gateway-in-public-subnet-why).

### Summary Notes

- HA only within an AZ.
- Must exist in a public subnet.
- Must existing in a public subnet so it can be attached an elastic IP and has access to RT.
- Accessible to all subnets within a VPC.


## ELB

### How many public subnets are requires for ELBs?

Atleast 2 public subnets must exist.
