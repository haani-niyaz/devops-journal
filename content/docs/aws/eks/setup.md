---
title: Setup
weight: 1
---

# Setup

The following articles are worth reading before you begin:

- https://aws.amazon.com/blogs/containers/de-mystifying-cluster-networking-for-amazon-eks-worker-nodes/

## Create VPC 

Begin my creating a VPC to launch worker nodes. We also create a control plane security group as per [guide](https://docs.aws.amazon.com/eks/latest/userguide/sec-group-reqs.html).

See cloudformation template [here](https://gitlab.com/haani-niyaz/cloud-engineering/-/blob/master/eks/1-amazon-eks-vpc-sample.yaml).

## Create cluster

See cloudformation template [here](https://gitlab.com/haani-niyaz/cloud-engineering/-/blob/master/eks/2-control-plane.yaml).

## Create worker nodes

See cloudformation template [here](https://gitlab.com/haani-niyaz/cloud-engineering/-/blob/master/eks/3-amazon-eks-nodegroup.yaml).

## Join worker nodes to cluster

Update the `aws-auth` ConfigMap. See an example [here](https://gitlab.com/haani-niyaz/cloud-engineering/-/blob/master/eks/4-aws-auth-cm.yaml).

## Example app

See [here](https://gitlab.com/haani-niyaz/cloud-engineering/-/blob/master/eks/5-sample-app.txt).






