---
title: VPC
---

# VPC FAQ
---

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


## NAT Gateway

### Why don't you get all 254 allocatable IPs with CIDR /24?

AWS reserved 5 IPs so you get 251. See more details [here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html).

### Does it have to exist in a public subnet?

a NAT gatway requires a route to get to the internet. This is available in 
the public subnet where a default route to IGW entry exists in the route table.

See more details [here](https://serverfault.com/questions/854475/aws-nat-gateway-in-public-subnet-why).

### Summary Notes

- HA only within an AZ.
- Must exist in a public subnet.
- An elastic IP is automatically asssigned.
- Accessible to all subnets within a VPC.


## ELB

### How many public subnets are requires for ELBs?

Atleast 2 public subnets must exist.
