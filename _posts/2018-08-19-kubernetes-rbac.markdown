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

## Prerequisites

For the purpose of this example we use [minikube](https://github.com/kubernetes/minikube) as our Kubernetes cluster. 

## RBAC


*Role Based Access Control*  are roles defined to grant different levels of access to users.


### Terminology

#### Role

> In the RBAC API, a role contains rules that represent a set of permissions. Permissions are purely 
additive (there are no “deny” rules).

- A `Role` provides a level of access that can bound to a single user or a group of users. 
- Roles are set at indiviual Namespace. If rules need to be assigned to an entire cluster you can instead use `ClusterRole`.

#### Rule

> A Rule is a set of operations (verbs) that target resources in an API groups.

#### Verb

> Verbs define activities that can be performed on resources that belong to different API groups.


Here's a prelude to a `Role` definition:

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


#### Subjects

Defined when binding an object to a Role or ClusterRole.

This can either hold a direct API object reference, or a value for non-objects such as user and group names.


### Authentication


#### Create a Private Key (User task)

{% highlight bash %}

# generate private
$ openssl genrsa -out jdoe.key 4096

# create certificate
$ openssl req -new -key jdoe.key -out jdoe.csr -subj "/CN=jdoe/O=devs"

{% endhighlight %}

Where `CN` is the username and `O` is the organization.

#### Sign CSR with Cluster CA (Admin task)

We sign the CSR with the cluster's Certificate Authority (CA). Using the cluster's CA private key we will 
approve the user's request by generting the necessary user certificate.

{% highlight bash %}

$ openssl x509 -req \
     -in keys/jdoe.csr \
     -CA ~/.minikube/ca.crt \
     -CAkey ~/.minikube/ca.key \
     -CAcreateserial \
     -out jdoe.crt \
     -days 365

{% endhighlight %}


#### Add Cluster (User task)

Adding a cluster is as follows:

{% highlight bash %}
$ kubectl config set-cluster NAME [--server=server] [--certificate-authority=path/to/certificate/authority]
{% endhighlight %}

So as per the definition, we will also need to use the CA's public certificate to connect to the Kubernetes API to set the cluster.

{% highlight bash %}

$ kubectl config set-cluster jdoe \
     --certificate-authority \
     ~/.minikube/ca.crt \
     --server https://192.168.99.100:8443

{% endhighlight %}

This will create a new cluster for `jdoe`. You can view the cluster config by running `kubectl config view`.

#### Add credentials (User task)

Set the user credentials with the private key and signed certificate.

{% highlight bash %}

$ kubectl config set-credentials jdoe \
     --client-certificate jdoe.crt \
     --client-key jdoe.key

{% endhighlight %}

#### Create a Context (User task)

A 'context' in Kubernetes allows us to define a relationship between user, cluster and namespace. A user can have access to multiple clusters with multiple namespaces. The context therefore provides a unique associations.


{% highlight bash %}

$ kubectl config set-context jdoe \
     --cluster jdoe \
     --user jdoe

{% endhighlight %}

Access to the `default` namespace is assumed if the namespace is not provided.

### Authorization

If you were to set the context with `kubectl config use-context jdoe` and attempt to access the cluster it will fail with something like:

{% highlight bash %}

$ kubectl get po
Error from server (Forbidden): pods is forbidden: User "jdoe" cannot list pods in the namespace "default"

{% endhighlight %}


#### How to validate a user's access as the cluster admin


{% highlight bash %}

$ kubectl auth can-i get pods --as jdoe
no

{% endhighlight %}

This is expected as the Kubernetes cluster does not come with any predefined roles but it does have ClusterRoles.

##### What is the difference between Role and ClusterRole?

The ClusterRole provides access to all namespaces within a cluster where a Role provides access to a specific namespace.


#### Show available Roles in the cluster


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

If we are to provide view access to a user we can use the existing `view` ClusterRole:

{% highlight bash %}

$ kubectl create rolebinding jdoe \
    --clusterrole view \
    --user jdoe \
    --save-config

# verify
$ kubectl get rolebindings
NAME      AGE
jdoe      34s

{% endhighlight %}

Here we create a RoleBinding to a ClusterRole however note that this is by default limited to the current Namespace if not explicitly provided (in our case `default`).

If we were to now verify access:


{% highlight bash %}

$ kubectl auth can-i get po --as jdoe
yes

$ kubectl auth can-i get po --as jdoe --all-namespaces
no

{% endhighlight %}

#### How does ClusterRole work in RoleBinding?

Although ClusterRole is something that is applied across the entire cluster and RoleBinding only operates within a Namespace, combining the two results in the RoleBinding limiting the ClusterRole operations on resources to that Namespace.


 because we are combining it with RoleBinding, it will limit it to the specified Namespace.


#### Bind user to a Role that provides 'view' access to all Namespaces

Instead of using the command-line we can define the CusterRoleBinding as shown below:

{% highlight yaml %}

# cluster-role-rbac-view.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: view
subjects:
- kind: User
  name: jdoe
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

$ kubectl auth can-i get po --as jdoe --all-namespaces
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
  name: dev
  namespace: dev
subjects:
- kind: User
  name: jdoe
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

$ kubectl auth can-i get po --as jdoe --all-namespaces
yes

$ kubectl auth can-i create  po --as jdoe --namespace dev
yes

$kubectl auth can-i create  po --as jdoe --all-namespaces
no

{% endhighlight %}


It is worth noting that this still prevents the user from running certain cluster operations as the role assigned is `admin` and not `cluster-admin`.


{% highlight bash %}

# access to everything with full rights
$ kubectl auth can-i '*' '*' --as jdoe --namespace dev
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
  name: jdoe

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jdoe
  namespace: jdoe
subjects:
- kind: User
  name: jdoe
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io

{% endhighlight %}


#### Give access to select resources 

In this example we provide full access to Pods and eliminate the `delete` operation for Deployments and ReplicaSets.

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
  name: jdoe
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: operations
  apiGroup: rbac.authorization.k8s.io

{% endhighlight %}


Verify:

{% highlight bash %}

$ kubectl auth can-i '*' pods --as jdoe --namespace default
yes

$ kubectl auth can-i '*' pods/attach --as jdoe --namespace default
yes

$ kubectl auth can-i '*' deployments --as jdoe --namespace default
no

$ kubectl auth can-i create deployments --as jdoe --namespace default
yes

{% endhighlight %}

