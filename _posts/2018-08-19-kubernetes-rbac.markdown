---
layout: post
title:  "Securing Kubernetes Cluster with RBAC"
date:   2018-08-19
author: "Haani Niyaz"
tags: 
 - rbac
 - k8s
 - tutorial
---

* TOC
{:toc}

## Prerequisites

You will need access to a Kubernetes cluster with admin privileges. For the purpose of this example I use [minikube](https://github.com/kubernetes/minikube) to operate.

## RBAC Terminology

*Role Based Access Control* sets different levels of access to users based on predefined roles.


### Role vs. ClusterRole

> In the RBAC API, a role contains rules that represent a set of permissions. Permissions are purely 
additive (there are no “deny” rules).

- A Role provides a level of access that can bound to a single user or a group of users. 
- Roles are set at an indiviual Namespace. If rules need to be assigned to an entire cluster you would instead use ClusterRole.

To drive the point home:

> The ClusterRole provides access to all namespaces within a cluster where a Role provides access to a specific namespace.


### Rule

A Role contains rules.

A Rule is a set of verbs (e.g: get, list, post etc.) that target resources (e.g: Pod, Service etc.) in an API groups.

#### Verb

> Verbs define activities that can be performed on resources that belong to different API groups.

#### API Group

Specifices the API Group resources (e.g: Pods, Services etc.) belongs to.

Here's a prelude to a Role definition:

{% highlight bash %}

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default # limit namespace
  name: pod-reader 
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

{% endhighlight %}


### Binding

Creates a binding between a user/group to a Role or ClusterRole. 

#### RoleBinding vs. ClusterRoleBinding

> A role binding grants the permissions defined in a role to a user or set of users. It holds a list of subjects (users, groups, or service accounts), and a reference to the role being granted. Permissions can be granted within a namespace with a RoleBinding, or cluster-wide with a ClusterRoleBinding.



#### Subjects

This can either hold a direct API object reference or a value for non-objects such as user and group names.


## A Visual Representation  


![RBAC Auth Flow](css/images/rbac.png){:class="img-responsive"}


## User Access

Let's go through examples of: 

1. How to authenticate 
2. How to authorize a user via RBAC

#### User vs. Admin tasks

The section with 'User Task' are actions carried out by an end user requesting access to the cluster. Whereas the 'Admin Task' sections must be peformed by a cluster admin (a user who belongs to the `cluster-admin` ClusterRole).

### Authentication

#### Create a Private Key (User task)

{% highlight bash %}

# generate private
$ openssl genrsa -out gary.key 4096

# create certificate
$ openssl req -new -key gary.key -out gary.csr -subj "/CN=gary/O=devs"

{% endhighlight %}

Where `CN` is the username and `O` is the organization. The generated `gary.csr` should be provided to the cluster admin.

#### Sign CSR with Cluster CA (Admin task)

We sign the CSR with the cluster's Certificate Authority (CA). Using the cluster's CA private key we will approve the user's request by generting the necessary user certificate.

{% highlight bash %}

$ openssl x509 -req \
     -in keys/gary.csr \
     -CA ~/.minikube/ca.crt \
     -CAkey ~/.minikube/ca.key \
     -CAcreateserial \
     -out gary.crt \
     -days 365

{% endhighlight %}

This will produce `gary.crt` which should be provided to the user.


#### Add Cluster (Admin task)

<sub>Note: This step is not required if the cluster already exists.</sub>

Adding a cluster is as follows:

{% highlight bash %}
$ kubectl config set-cluster NAME [--server=server] [--certificate-authority=path/to/certificate/authority]
{% endhighlight %}

So as per the definition, we will also need to use the CA's public certificate to connect to the Kubernetes API to set the cluster.

{% highlight bash %}

$ kubectl config set-cluster final-space \
     --certificate-authority \
     ~/.minikube/ca.crt \
     --server https://192.168.99.100:8443

{% endhighlight %}

This will create a new cluster for `gary`. You can view the cluster config by running `kubectl config view`.

#### Add credentials (User task)

Set the user credentials with the private key and signed certificate.

{% highlight bash %}

$ kubectl config set-credentials gary \
     --client-certificate gary.crt \
     --client-key gary.key

{% endhighlight %}

#### Create a Context (User task)

A 'context' in Kubernetes allows us to define a relationship between user, cluster and namespace. A user can have access to multiple clusters with multiple namespaces. The context therefore provides an unique association.


{% highlight bash %}

$ kubectl config set-context gary \
     --cluster gary \
     --user gary

{% endhighlight %}

Access to the `default` namespace is assumed if the namespace is not provided.

In the configuration above, we created a context called `gary` that maps a relation between user `gary` to cluster `finalspace` in the `default` namespace.

### Authorization

If you were to set the context with `kubectl config use-context gary` and attempt to access the cluster it will fail with something like:

{% highlight bash %}

$ kubectl get po
Error from server (Forbidden): pods is forbidden: User "gary" cannot list pods in the namespace "default"

{% endhighlight %}


#### How to validate a user's access as the cluster admin


{% highlight bash %}

$ kubectl auth can-i get pods --as gary
no

{% endhighlight %}

This is expected since there is no Role associated to the user 'gary' granting him any privileges.

#### How to check available Roles in a cluster


{% highlight bash %}

$ kubectl get roles
No resources found.

{% endhighlight %}

By default Kubernetes cluster does not define any Roles. However there are default ClusterRoles:


{% highlight bash %}

$ kubectl get clusterRoles | grep -v system
NAME                                                                   AGE
admin                                                                  27d
cluster-admin                                                          27d
edit                                                                   27d
view                                                                   27d

{% endhighlight %}


#### Bind user to a Role that provides 'view' access to current Namespace

If you wanted to provide just view access to a user you can use the existing `view` ClusterRole:

{% highlight bash %}

$ kubectl create rolebinding gary-view \
    --clusterrole view \
    --user gary \
    --save-config

# verify
$ kubectl get rolebindings
NAME           AGE
gary-view      34s

{% endhighlight %}

Here we create a RoleBinding to a ClusterRole however note that this is by default limited to the current Namespace if not explicitly provided (in our case `default`).

If we were to now verify access:


{% highlight bash %}

$ kubectl auth can-i get po --as gary
yes

$ kubectl auth can-i get po --as gary --all-namespaces
no

{% endhighlight %}

#### How does ClusterRole work in RoleBinding?

Although ClusterRole is something that is applied across the entire cluster and RoleBinding only operates within a Namespace, combining the two results in the RoleBinding limiting the ClusterRole operations to resources in that Namespace.


> Because we are combining it with RoleBinding, it will limit it to the specified or default Namespace.


#### Bind user to a Role that provides 'view' access to all Namespaces

Instead of using the command-line we can define the CusterRoleBinding in yaml configuration:

{% highlight yaml %}

# cluster-role-rbac-view.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: gary-view
subjects:
- kind: User
  name: gary
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io

{% endhighlight %}


Go ahead and appply it:

{% highlight bash %}

$ kubectl apply -f cluster-role-rbac-view.yml

{% endhighlight %}

If we were to now verify access:

{% highlight bash %}

$ kubectl auth can-i get po --as gary --all-namespaces
yes

{% endhighlight %}


#### Bind user to a Role that provides full access to a specific Namespace


{% highlight yaml %}

#cluster-role-rbac-ns-admin.yml

apiVersion: v1
kind: Namespace
metadata:
  name: dev

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-view
  namespace: dev
subjects:
- kind: User
  name: gary
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: admin
  apiGroup: rbac.authorization.k8s.io

{% endhighlight %}

Go ahead and appply it:

{% highlight bash %}

$ kubectl apply -f cluster-role-rbac-view.yml

{% endhighlight %}

If we were to now verify access:

{% highlight bash %}

$ kubectl auth can-i get po --as gary --all-namespaces
yes

$ kubectl auth can-i create  po --as gary --namespace dev
yes

$kubectl auth can-i create  po --as gary --all-namespaces
no

{% endhighlight %}


It is worth noting that this still prevents the user from running certain cluster operations as the role assigned is `admin` and not `cluster-admin`.


{% highlight bash %}

# access to everything with full rights
$ kubectl auth can-i '*' '*' --as gary --namespace dev
no

# 'admin' role has limitations compared to 'cluster-admin'
$ kubectl get clusterrole | grep -v system
NAME                                                                   AGE
admin                                                                  28d
cluster-admin                                                          28d
..

{% endhighlight %}


#### Giving full access to a user within a Namespace


{% highlight yaml %}

#cluster-role-rbac-ns-admin.yml

apiVersion: v1
kind: Namespace
metadata:
  name: galaxy-one

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: gary-admin
  namespace: gary
subjects:
- kind: User
  name: gary
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io

{% endhighlight %}


#### Give access to select resources 

Say you wanted to provide full access to Pods and block `delete` operations for Deployments and ReplicaSets.

If you were to look at the APIGroups for Deployments you will notice that it is not limited to the core APIGroup:


{% highlight bash %}

$ kubectl describe clusterrole admin | grep deployments
  Resources                                       Non-Resource URLs  Resource Names  Verbs
  ---------                                       -----------------  --------------  -----
  deployments.apps                                []                 []              [create delete deletecollection get list patch update watch]
  deployments.apps/rollback                       []                 []              [create delete deletecollection get list patch update watch]
  deployments.apps/scale                          []                 []              [create delete deletecollection get list patch update watch]
  deployments.extensions                          []                 []              [create delete deletecollection get list patch update watch]
  deployments.extensions/rollback                 []                 []              [create delete deletecollection get list patch update watch]
  deployments.extensions/scale                    []                 []              [create delete deletecollection get list patch update watch]

{% endhighlight %}

We have an `apps` and `extensions` APIGroup. 


Similarly, if we look at Pods, you will notice that there are sub resources such as `pods/attach` and `pods/status` which must be explicitly defined in the resource rules if access is required.

{% highlight bash %}

$ kubectl describe clusterrole admin | grep pods
  Resources                                       Non-Resource URLs  Resource Names  Verbs

  pods                                            []                 []              [create delete deletecollection get list patch update watch]
  pods/attach                                     []                 []              [create delete deletecollection get list patch update watch]
  pods/exec                                       []                 []              [create delete deletecollection get list patch update watch]
  pods/log                                        []                 []              [get list watch]
  pods/portforward                                []                 []              [create delete deletecollection get list patch update watch]
  pods/proxy                                      []                 []              [create delete deletecollection get list patch update watch]
  pods/status                                     []                 []              [get list watch]

{% endhighlight %}


Based on the above, we will now have the following:


{% highlight yaml %}

#role-rbac-ops.yml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: operations
rules:
- resources: ["pods", "pods/attach", "pods/exec", "pods/log", "pods/status"]
  verbs: ["*"]
  apiGroups: [""]
- resources: ["deployments", "replicasets"]
  verbs: ["create", "get", "list", "update", "watch"]
  apiGroups: ["", "apps", "extensions"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: operations
  namespace: default
subjects:
- kind: User
  name: gary
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: operations
  apiGroup: rbac.authorization.k8s.io

{% endhighlight %}


We have our first Role definition followed by binding to the user `gary`.

Verify:

{% highlight bash %}

$ kubectl auth can-i '*' pods --as gary --namespace default
yes

$ kubectl auth can-i '*' pods/attach --as gary --namespace default
yes

$ kubectl auth can-i '*' deployments --as gary --namespace default
no

$ kubectl auth can-i create deployments --as gary --namespace default
yes

{% endhighlight %}


## Group Access

If we wanted to provide access to a group of users (instead of creating a Role per user), you can setup a Role or ClusterRole per group.


### How to setup a group

You already setup a group called `devs` when we created the user certificate earlier with the following command:


{% highlight bash %}

# create certificate
$ openssl req -new -key gary.key -out gary.csr -subj "/CN=gary/O=devs"

{% endhighlight %}

As you may have noticed organization `O=devs` is how the Kubernetes verifies if a request belongs to a group.


### Give group access to a specific Namepace

In the example below you have a `dev` Namespace that has a RoleBinding to a group called `devs` with `cluster-admin` permissions. The only difference between a user or group RoleBinding (or ClusterRoleBinding) definition is the `subjects.kind` value is now set to `Group` and the name is the certificate organization name we set earlier. 


{% highlight yaml %}

#cluster-role-rbac-ns-admin-group.yml

apiVersion: v1
kind: Namespace
metadata:
  name: dev

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devs
  namespace: dev
subjects:
- kind: Group
  name: devs
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io

{% endhighlight %}