---
layout: post
title:  "Getting Started with Kubernetes"
date:   2016-10-16
author: "Haani Niyaz"
tags: 
 - docker 
 - kubernetes
---

* TOC
{:toc}


## Objective

This article is mostly a walkthrough in an attempt to learn Kubernetes. It is still in a work-in-progress state and the content may change in the future. I recommend doing your own research to support the material below.

### What Problem does Kubernetes Solve?

Multiple containers talking to each other on a single host is trivial and can be solved with native docker networking capabilities. But as you scale, you will find that you end up with containers scattered across multiple hosts. At this point a number of concerns arise:

1. How do containers talk to each other on different hosts?
2. How to manage communication between containers and the outside world?
3. How do you maitain state such as IP addresses?
4. etc.


To solve the problems highlighted above kuberenetes provides *Orchestration* for container deployment and management. A *Scheduler* to place containers on one or more available hosts. A mechanism to discover these containers that span across multiple hosts known as *Service Discovery*.

To understand the problem further and really get into the concepts I recommend reading [this](http://www.oreilly.com/webops-perf/free/docker-networking-and-service-delivery.csp) free e-book.


### Topics Covered

* Nodes
* Pods
* Labels
* Selectors
* Controllers
* Services
* Control Pane
* API


## Kubernetes Architecure

We will cover the key topics lightly before diving into the command-line. First, let's visualize the topics of interest:


{% highlight  ruby%}
+-----------------------+
|                       |
|             +-------+ |     xN     +--------------------+
|    Node     | Docker| |  <---------+  Master Controller |
|             +-------+ |            +---------+----------+
|   +------+ +-------+  |                      
|   |Pod   | |Pod    |  |                      
|   |c1 c2 | |c3 c4  |  |                      
|   +------+ +-------+  |                      
|  +------------------+ |                      
|  | Kubernetes Proxy | |                      
|  +------------------+ |                      
|  +------+             |
|  | Etcd |             |
|  +------+             |
+-----------------------+

{% endhighlight %}

       						 									                                                                                        
### Texual Breakdown

A *Master Controller* has N number of *Nodes* i.e: virtual machines.  
Each node has N number of *Pods* and the the docker daemon instantiates containers within the pods.


### Components

Here is a high-level overview of each component so we have a frame of reference when we ecounter them in the walk throughs. 


#### Nodes

Your nodes are the individual hosts (VMs) that act as 'container clients' via docker. Each node runs *etcd* which is a key value management store. Kubernetes uses etcd to exchange messages and report on cluster status . An excerpt from this [article](https://www.digitalocean.com/community/tutorials/an-introduction-to-kubernetes) describes it in an easy to digest manner:

> For example a job scheduler needs to notify a machine that it has work to do. Once that work has been completed machines may need to communicate that fact to some other component in the cluster. A distributed system needs a reliable coordination mechanism, so it’s important that this communication happen in a timely and reliable manner to keep everything running smoothly. In essence something has to manage the state of the cluster – the source of truth. This is where etcd comes in.


In order to deal with individual host subnetting and to make services available to external parties, a *kubernetes proxy* service is run on each node server.

Nodes are also sometimes referred to as *Minions*.

#### Pods

Pods contain one or more containers. These containers are guaranteed to be on the same node so that they can share resources like volumes, services etc.

Pods are assigned unique IPs within each cluster allowing you to have multiple pods running on the same port and not have to worry about port conflict.


#### Labels

You can attach key/value pairs to any component like a 'pod' or a 'node' in the kubernetes eco- system. However this is more than just way to tag components. The labels allow you to identify them in configuration and take certain certain actions on them.


#### Selectors 

Selectors are queries made on *labels*. 


>Labels and Selectors are the primary way kubernetes determines which components a given operation
should run on when indicated.


#### Controllers

Manages the desired configuration state of the cluster. So this means it manages a set of pods and can engage other controllers to perform different tasks. For example, to handle replication and scaling you have the *Replcation Controller*.


#### Services
*Services* enable pods to interact with what is called a 'service' rather the pods directly.  If you think about it as pods scale up and down you cannot guarantee that one set of pods can directly access another set via IP. At this point you run into issues with discovering pods and keeping track of them. A service groups a logical set of pods and a policy by which to access them.


## Installing Kubernetes

We will be setting up a master controller and 2 nodes.

**Note:** *All commands and actions are based on CentOS 7.*


Its important to ensure that time is synchronized across all nodes so let's do that first.

{% highlight bash %}
{% raw %}

# Install ntp package
$ yum install -y ntp

# Manage service
$ systemctl enable ntpd && systemctl start ntpd

{% endraw %}
{% endhighlight %}


Now, to make our life just a little easier, I'm going to add the following to all of our node's `etc/hosts` file.

{% highlight bash  %}
{% raw %}

172.31.33.47 master-controller
172.31.32.100 minion-1
172.31.33.244 minion-2

{% endraw %}
{% endhighlight %}


### RTFM

The Kubernetes site has pretty decent tutorial on how to install and configure the master controller and nodes so I won't replicate it here but [this](http://kubernetes.io/docs/getting-started-guides/centos/centos_manual_config/) is the link for CentOS 7.


## Nodes

{% highlight bash %}
{% raw %}

# Show nodes
$ kubectl get nodes


# Show verbose information on nodes
$ kubectl describe nodes

{% endraw %}
{% endhighlight %}


## Pods

We will start by creating a *pod*. The pod definition is below in `nginx.yaml`:

{% highlight yaml %}
---
apiVersion: v1
kind: Pod
metadata: 
	name: nginx
spec:
	containers: 
	- name: nginx
	  image: nginx:1.7.9
	  ports:
	  - containerPort: 80
{% endhighlight %}


The pod definition above is written in `yaml` format. Firstly, you need to specify the api version as `apiVersion`. You can find the available api versions by running:

{% highlight bash %}
{% raw %}

$ kubectl api-versions
autoscaling/v1
batch/v1
extensions/v1beta1
v1

{% endraw %}
{% endhighlight %}


Next, you specify what you're creating by specifying the `kind`. In this case we want to create a `Pod`. We name the pod `nginx` and proceed to specifying the container details in the `spec` section.

Once we're done, we can create the pod by running:

{% highlight bash %}
{% raw %}
$ kubectl create -f nginx.yaml
{% endraw %}
{% endhighlight %}

You can check your pod by running:

{% highlight bash %}
{% raw %}
$ kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          39m
{% endraw %}
{% endhighlight %}


#### How do we find which node the pod is running on?

{% highlight bash %}
{% raw %}
$ kubectl describe pod nginx
Name:		nginx
Namespace:	default
Node:		minion-2/172.31.33.244
...
{% endraw %}
{% endhighlight %}

The above command provides a lot of useful configuration information such as labels used, image ids, environment variables used etc. 

**Note:** In the current configuration only pods or containers within the pods (that reside on the node) can communicate with eache other.

We can jump to `minion-2` and run a `docker ps` which will show the container created:

{% highlight bash %}
{% raw %}
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS               NAMES
b9f8c97ec246        nginx:1.7.9                          "nginx -g 'daemon off"   57 minutes ago      Up 57 minutes                           k8s_nginx.4580025_nginx_default_98d55a1c-886b-11e6-9ae6-0aea08b17949_f6b1a98f
76ddff05ecce        gcr.io/google_containers/pause:2.0   "/pause"                 58 minutes ago      Up 58 minutes                           k8s_POD.cf58006d_nginx_default_98d55a1c-886b-11e6-9ae6-0aea08b17949_c6df5a44

{% endraw %}
{% endhighlight %}


As you may have noticed despite only specifying a single container in our `yaml` configuration file we also have container `76ddff05ecce` from image `gcr.io/google_containers/pause:2.0`. The reason for this is explained well in [this](http://www.dasblinkenlichten.com/kubernetes-101-networking/) article.


>  the pause container really just holds the network endpoint for the pod.  It really doesn’t do anything else at all.


#### How to add a label to a pod?

Labels are like descriptor in the form of a `key/value` pair. To add a label we just to include that in our `nginx.yaml` definition like so:

{% highlight yaml %}

---
apiVersion: v1
kind: Pod
metadata: 
	name: nginx
	labels:
	 app: nginx
spec:
	containers: 
	- name: nginx
	  image: nginx:1.7.9
	  ports:
	  - containerPort: 80

{% endhighlight %}


Notice how we added the label `app:nginx`. This allows you to do things like:

{% highlight bash %}
{% raw %}
# Create a new pod
$ kubectl create -f nginx.yaml

# Retrieve pods that match our label

$ kubectl get pods -l app=nginx

# OR

$ kubectl describe pods -l app=nginx

{% endraw %}
{% endhighlight %}

Use the `-l` flag to specify the `key/value` pair to filter results.


#### How to execute a command in a running pod?


{% highlight bash %}
{% raw %}
 
# 'nginx' is the name of the pod defintion
# If multiple containers exist, it will run it on the first container by default
$ kubectl exec nginx date
Sun Oct 16 08:24:10 UTC 2016


# Give container id if multiple containers are running within a pod
# Get container id by running 'kubectl describe pods'
$ kubectl exec nginx -c <container-name> date


# Get a login prompt
$ kubectl exec nginx -it -- /bin/bash

{% endraw %}
{% endhighlight %}




## Deployments

Now that we've played around with pods we are ready to look at how deployments should be configured. Deployments are a declarative of way of of managing Pods and Replica Sets (yet to look at). You only need to change the desired state in your configuration and the **Deployment Controller** will take care of bringing your clusted to the desired state.

For instance if we have a `dev` environment our configuration object will have some special properties as shown below:

### Deployment Specification

{% highlight yaml %}

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
 name: nginx-deployment-dev
spec:
 replicas: 1
 template:
   metadata:
    labels:
     app: nginx-deployment-dev
   spec:
     containers:
     - name: nginx-deployment-dev
       image: nginx:1.7.9
       ports:
       - containerPort: 80

{% endhighlight %}

Let break this down.

We set the `apiVerversion` to `extensions/v1beta1` which is the required version for deployments. 

This is a 'deployment' so the `kind` is set to reflect that. 

The specification for this deployment is denoted by `spec` which has an attribute called `replicas` and a `template`. 

#### What are `replicas`?

> A Replica Set ensures that a specified number of pod “replicas” are running at any given time.
However, a Deployment is a higher-level concept that manages Replica Sets and provides declarative updates to pods along with a lot of other useful features.


#### What is a `template`?

> The .spec.template is a pod template. It has exactly the same schema as a Pod, except it is nested and does not have an apiVersion or kind.

A pod template in a *deployment* must also specify appropriate labels (i.e. don’t overlap with other controllers).


### Kicking Off a Deployment

{% highlight bash %}
{% raw %}

# Create
$ kubectl create -f nginx-deployment-dev.yaml
deployment "nginx-deployment-dev" created


# Get deployment details
$ kubectl get deployments
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment-dev   1         1         1            1           52s


$ kubectl describe deployments -l app=nginx-deployment-dev

Name:			nginx-deployment-dev
Namespace:		default
CreationTimestamp:	Sat, 08 Oct 2016 07:08:33 +0000
Labels:			app=nginx-deployment-dev
Selector:		app=nginx-deployment-dev
Replicas:		1 updated | 1 total | 1 available | 0 unavailable
StrategyType:		RollingUpdate
MinReadySeconds:	0
RollingUpdateStrategy:	1 max unavailable, 1 max surge
OldReplicaSets:		<none>
NewReplicaSet:		nginx-deployment-dev-2568522567 (1/1 replicas created)
Events:
  FirstSeen	LastSeen	Count	From				SubobjectPath	Type		Reason			Message
  ---------	--------	-----	----				-------------	--------	------			-------
  2m		2m		1	{deployment-controller }			Normal		ScalingReplicaSet	Scaled up replica set nginx-deployment-dev-2568522567 to 1

{% endraw %}
{% endhighlight %}


If you were to get the pods now you'll see the following:

{% highlight bash %}
{% raw %}
$ kubectl get pods
NAME                                    READY     STATUS    RESTARTS   AGE
nginx-deployment-dev-2568522567-fd63o   1/1       Running   0          4m
{% endraw %}
{% endhighlight %}

Notice hash id attached to the name of the pod `2568522567-fd63o `. This is because when we scale out it allows kuberentes to create multiple pods with no conflict.


### Making Changes to a Deployment

Say we want to make an update to the version of the nginx image. With a *deployment* this can be easily done by modifying the image version and applying the update as opposed to a delete/create.

Say if I change the line containing the image version to `1.8` we can easily propogate this change.

Before we make the change let's check which version is currently running:

{% highlight bash %}
{% raw %}
# Describe pod with label attached to deployment to find image
$ kubectl describe pod -l app=nginx-deployment-dev
...
Containers:
  nginx-deployment-dev:
    Container ID:	docker://3eabcfb07672340bc75725ee1971ef18913e9a236d32635853e541698920c357
    Image:		nginx:1.7.9
...
{% endraw %}
{% endhighlight %}


Once we update the `image` attribute in our yaml file `1.8` we can apply and verify as shown below:

{% highlight bash %}
{% raw %}
$ kubectl apply -f nginx-deployment-dev.yaml

# Check image version
$ kubectl describe pod -l app=nginx-deployment-dev
...
Containers:
  nginx-deployment-dev:
    Container ID:	docker://af17a5776d02282f848d7a278673c45a045797f11fdef7223cd4cb9f2cdae167
    Image:		nginx:1.8
...
{% endraw %}
{% endhighlight %}


### Deployment via the `run` command

If you wish to test a deployment, you can do this without having to write a yaml configuration file. Let's see how we can do that:


{% highlight bash %}
{% raw %}
# Use the run command to deploy via command-line
$ kubectl run sample --image=nginx:1.8

{% endraw %}
{% endhighlight %}

What does the deployment look like:

{% highlight bash %}
{% raw %}
[root@kaizencoder1 builds]# kubectl describe deployment sample
Name:			sample
Namespace:		default
CreationTimestamp:	Sun, 16 Oct 2016 07:54:16 +0000
Labels:			run=sample
Selector:		run=sample
Replicas:		1 updated | 1 total | 1 available | 0 unavailable
StrategyType:		RollingUpdate
MinReadySeconds:	0
RollingUpdateStrategy:	1 max unavailable, 1 max surge
OldReplicaSets:		<none>
NewReplicaSet:		sample-2445631297 (1/1 replicas created)
Events:
  FirstSeen	LastSeen	Count	From				SubobjectPath	Type		Reason			Message
  ---------	--------	-----	----				-------------	--------	------			-------
  1m		1m		1	{deployment-controller }			Normal		ScalingReplicaSet	Scaled up replica set sample-2445631297 to 1

{% endraw %}
{% endhighlight %}


You will notice kubernetes automatically sets a label `run=sample`. With this information we can find the specific pods details:

{% highlight bash %}
{% raw %}
$ kubectl describe pods -l run=sample
{% endraw %}
{% endhighlight %}

The above command will show the container details like which node it is running on, the container IP, environment variables etc.


#### Specify `replicas` and `labels`

{% highlight bash %}
{% raw %}
# Deployment
$ kubectl run sample --image=nginx:1.8 --replicas=2 --labels=app=nginx,release=1

# Get details
$ kubectl describe deployments -l app=nginx
Name:			sample
Namespace:		default
CreationTimestamp:	Sun, 16 Oct 2016 08:09:43 +0000
Labels:			app=nginx,release=1
Selector:		app=nginx,release=1
Replicas:		2 updated | 2 total | 2 available | 0 unavailable
...
{% endraw %}
{% endhighlight %}


## Services

Containers on a node are not aware of containers on other nodes. Services make this possible by providing a named load balancer to access one or more containers.

We can create a *service* defintion for our *deployment* `ngix-deployment-dev` (created earlier) as shown below:

{% highlight yaml %}
{% raw %}
---
apiVersion: v1
kind: Service
metadata:
 name: nginx-service
spec:
 ports:
 - port: 8000
   targetPort: 80
   protocol: TCP
 selector: 
   app: nginx-deployment-dev 
{% endraw %}
{% endhighlight %}

The `selector` used in the *service* config matches the `label` we used in the `nginx-deployment-dev` *deployment* config. Services find the pods to load balance based on the pod's labels. The kubernetes [best practices guide](http://kubernetes.io/docs/user-guide/config-best-practices/) recommends creating the *service* before *deployment*. So make sure to delete the existing deployment as shown below:

{% highlight bash %}
{% raw %}
$ kubectl delete deployment nginx-deployment-dev
{% endraw %}
{% endhighlight %}


We can create the *service* now.

{% highlight bash %}
{% raw %}
$ kubectl create -f nginx/nginx-service.yaml 
service "nginx-service" created

# Show service
$ kubectl get service
NAME            CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes      10.254.0.1      <none>        443/TCP    6d
nginx-service   10.254.56.190   <none>        8000/TCP   28s
{% endraw %}
{% endhighlight %}

The `nginx-service` IP is based on the IP range set in `/etc/kubernetes/apiserver` config:

{% highlight bash %}
{% raw %}
# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
{% endraw %}
{% endhighlight %}

Once the service is running, create the deployment as we did before. 


### Characteristic of a Service

{% highlight bash %}
{% raw %}
# Get service details
$ kubectl describe service nginx-service
Name:			nginx-service
Namespace:		default
Labels:			<none>
Selector:		app=nginx-deployment-dev
Type:			ClusterIP
IP:			10.254.56.190
Port:			<unset>	8000/TCP
Endpoints:		172.17.0.2:80,172.17.0.2:80,172.17.0.3:80
Session Affinity:	None
No events.

{% endraw %}
{% endhighlight %}

All containers on all nodes will see `nginx` running on `10.254.56.190` and port `8000`. The service will map incomming traffic to the defined `targetPort` on a pod. To be clear, the `port` is what pods use to access the *service*. The `targetPort` is the port the container accepts traffic on. 

The `Endpoints` are the pods running on each node.












