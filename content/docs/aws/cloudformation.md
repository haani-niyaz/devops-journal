---
title: Cloudformation
---


# Cloudformation FAQ
---

### How to omit a property if a value does not exist?

Use `AWS::NoValue`.

Example:

{{< highlight bash >}}
# If condition (true); value else value;
DBSnapshotIdentifier: !If [isRestore, !Ref SnapToRestore, !Ref "AWS::NoValue"]
{{< / highlight >}}


### Can you change a stack name after it is created?

No.

### How to create a stack?

{{< highlight bash >}}

$ aws cloudformation create-stack --stack-name mynetwork --template-body file://infra.yml 
{{< / highlight >}}

### How to check for progress?

{{< highlight bash >}}
$ aws cloudformation wait stack-create-complete --stack-name mynetwork 
{{< / highlight >}}


### How to check status?

{{< highlight bash >}}
aws cloudformation describe-stacks --stack-name mynetwork 
{{< / highlight >}}


### How to update stack?

{{< highlight bash >}}
$ aws cloudformation update-stack --stack-name mynetwork --template-body file://infra.yml
{{< / highlight >}}

 
### How to delete stack?

{{< highlight bash >}}
$ aws cloudformation delete-stack --stack-name mynetwork --profile la-temp
{{< / highlight >}}