---
title: Route53
---

# Route53 
---




## Health Checks

### Types

- Simple endpoint
- Calculated summary checks 
- State of Cloudwatch alarms 

## Routing Policies

### Simple

- You can provide a list of endpoint IPs but it is not load balanced but rather spread. This is because it is determined by the TTL and caching of the DNS record on the client side. 
- If you use an Alias record that points to an AWS resource, you can only use it with a single AWS resource.
