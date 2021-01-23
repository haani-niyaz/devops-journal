---
title: IAM
---

# IAM FAQ
---

### Which region does IAM belong to?

IAM is across all regions (global)

### What level of permissions do new users created within IAM have?

- No access to any AWS service which is the **explicit deny** rule set for all new users. This conforms to the *Principle of Least Privileges*. 

#### Best practice
Create an admin account for daily activity. Do not use your `root` account.

### What is a Policy?

- Represents a permission scheme.
- An IAM policy attached to a user is called an 'Identity Policy' whereas when it is attached to a resource (e.g: S3) it is a 'Resource Policy'.
- If an EXPLICIT DENY and an EXPLICIT ALLOW is used, the deny rule will override the allow.
- An IAM policy can specify a 'Principle' to denote who can access the resource. See more [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html#intro-structure-principal).
- Policies are instantaneously applied.
- Policies can be appplied directly to an identity which is known as an 'Inline Policy' or they can be created independantly as a 'Managed Policy' and attached. 

{{< hint info >}}
**Tip**  
If multiple identities require a set of permissions it is best practice to create a Manage Policy and attache it rather than creating individual Inline Policies with the same permissions.
{{< /hint >}}

- Groups cannot be referenced with an ARN. Therefore they cannnot be attached to a Resource Policy (e.g: s3) as a Principal to allow access like you can do with Users or Roles.
- A Statement can contain many policies. Each Policy is made up of:


{{< columns >}} 
### Effect
The ability to allow or deny the action on that resource 
<---> 
### Action
The operation on that resource 
<---> 
### Resource
AWS resource you are targetting
{{< /columns >}}


{{< hint info >}}
**Tip**  
If you wish to block a user due to a breach you can add the **explicit deny** Policy to override all access rights.
{{< /hint >}}


#### Example policies:

An example of the `AdministratorAccess` Policy which allows any action to be taken on all AWS resouces or services.

{{< highlight bash >}}
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
{{< / highlight >}}


Another example of the `AmazonS3ReadOnlyAccess` Policy which provides readonly access to all buckets. The resources will be limited to S3 despite the `Resource` field specifying access to all resources. This is enforced by the `Action` entries limiting it to S3.


{{< highlight bash >}}
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:List*"
            ],
            "Resource": "*"
        }
    ]
}
{{< / highlight >}}

### What is a Role?

- AWS resources can "assume" Roles to gain specific privileges defined via a 'Trust Policy'. This is what tells you **who can assume the Role**. This followed by a 'Permissions Policy' which defined **what you can do when you assume the Role**. There is nothing special about the Permissions Policy, it is the same type of policy applied to a group or a user.
- Roles can be changed after an instance is created.
- Roles are also used for non-IAM users who have some form of AD account.
- A good use case for a Role is when you don't know who will need to have the permissions in advance. Unlike a user where you have a clear 1-to-1 mapping between the identity and the permissions, Role take on a more indirect way to
give something access on a need-only basis.

{{< hint info >}}
**Tip**  
You can setup a Trust Relationships within a Role to allow users from a different account to assume a Role in your acccount to provide the necessary permissions.
{{< /hint >}}


Further information can be found [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html).

#### How does this work?

For a detailed look see [this](https://aws.amazon.com/blogs/security/now-create-and-manage-aws-iam-roles-more-easily-with-the-updated-iam-console/) article.

For example, if you want to allow EC2 instances to have read access to S3 buckets:

- Create a trust Policy to define what resource(s) can assume the Role.

{{< highlight bash >}}


# trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Principal": {"Service": "ec2.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }
}



# Create the role and attach the trust policy to allow EC2 instances to assume this role
$ aws iam create-role --role-name EC2S3Access --assume-role-policy-document trustpolicy.json

{{< / highlight >}}


- Create a permission Policy to attach to the Role:

{{< highlight bash >}}

# permission-policy.json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": [
        "s3:Get*",
        "s3:List*"
     ],
    "Resource": "*"
  }
}



# Embed the permissions policy (in this example an inline policy) to the role to specify what it is allowed to do.
$ aws iam put-role-policy --role-name EC2S3Access --policy-name S3ReadOnly --policy-document policy.json


{{< / highlight >}}


- Create an Instance Profile (this is automatically created via the console and will be set to the Role name)


{{< hint info >}}
**Note**  
An Instance Profile is a container for an IAM role that you can use to pass role information to an EC2 instance when the instance starts.
{{< /hint >}}


{{< highlight bash >}}

# Create the instance profile required by EC2 to contain the role
$ aws iam create-instance-profile --instance-profile-name EC2-ReadBucket-S3

{{< / highlight >}}

- Add the Role to the profile


{{< highlight bash >}}

# Finally, add the role to the instance profile
$ aws iam add-role-to-instance-profile --instance-profile-name EC2-ReadBucket-S3 --role-name EC2S3Access

{{< / highlight >}}


See full example [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html).

In summary:

1. Create a Role (e.g: `EC2S3Access`) with a trust Policy (e.g: `trust-policy.json`) that enables the resource to assume a Role
2. Define the permissions Policy to be granted (e.g: `permission-policy.json`)
3. Attach the Policy to the Role 
4. Create the instance profile (e.g: `EC2-ReadBucket-S3`)
5. Assign Role to Instance Profile













