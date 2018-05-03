---
layout: post
title:  "Kubernetes Notebook"
date:   2018-05-02
author: "Haani Niyaz"
tags: 
 - docker
 - k8s
 - tutorial
---

Helpful notes on Kubernetes concepts.

## Pods

### Basic Pod Template

{% highlight yaml %}
apiVersion: v1
kind: Pod
metadata:                   
  name: my-nginx
spec:
  containers: # Container runtime definition
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
{% endhighlight %}

#### Notes

1. Pods control containers. It is an abstraction layer for kubernetes to manage containers.
2. Pods are ephemeral in nature therefore they are not meant to be accessed directly.
3. Pods have a unique IP address.
4. Containers within a Pod can communicate with each other because they share the networking namespace.
5. Pods even on different nodes can communicate with other directly via IP (no NAT).
6. External access is via 'Services' (covered later).



## ReplicationControllers

### Basic ReplicationController Template

{% highlight yaml %}
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 3 # Desired number of pods	
  selector:
    app: webserver # Pods that should be in the rc's scope
  template: # Used when creating new pod replicas
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
          - containerPort: 80
{% endhighlight %}     


#### Notes

1. ReplicationControllers don't care about the actual content of the Pod template (e.g: image, environment vars etc.) **after it has been created**. 
2. Changes to the `spec.selector` label have no effects on existing Pods. Changing this would make the Pod fall out of scope of the ReplicationController.
3. The `rc.spec.template` only affects new Pods that are created and not existing Pods. You will need to delete the old Pods and allow the Replication Controller replace it with the changes in the modified template for changes to take effect.
4. The pod labels `spec.template.labels` must match `spec.selector` of the Replication Controller.
5. The relationship between a Pod and the Replication Controller is via the labels. This allows you to add/remove Pods to a ReplicationController.
6. It is important to note that if you change the label of a running Pod, the ReplicationController will spin up a new Pod to replace it to meet the desired `spec.replicas` count.
7. Cannot match pods with the same label 'key'. For example, a single ReplicationController can't match pods with the label `app=webserver` and `app=frontend`. This is only possible via ReplicaSets (covered below).
8. Pods can be created independently and linked to a ReplicationController via the `spec.selector`.


## ReplicaSets

### Basic ReplicaSet Template

{% highlight yaml %}
apiVersion: v1/beta2
kind: ReplicaSet
metadata:
  name: my-nginx
spec:
  replicas: 3 # desired number of pods
  selector:
   matchLabels:
    app: webserver # pods that should be in the rc's scope
  template: # used when creating new pod replicas
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
          - containerPort: 80
{% endhighlight %}     


#### Notes

1. New generation ReplicaControllers with more expressive Pod selectors.

Example:

{% highlight yaml %}
...
selector:
 matchExpressions:
  - key: app # The selector requires the pod to contain label with 'app' key.
    operator: in
    values: # The label values can be 'webserver' or 'frontend'
     - webserver
     - frontend
...
{% endhighlight %}     


## Services

### Basic ReplicaSet Template

{% highlight yaml %}
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    external: web-service
spec:
  ports:
    - port: 80 # The port exposed by this service
      protocol: TCP # Default is 'TCP' but let's be explicit
      targetPort: 80 # Port number to access on the pod
  selector:
    app: webserver
{% endhighlight %}                                                                                                                     



#### Notes

1. Entrypoint to one or more Pods which has an IP address and port that never changes while the service exists.
2. Clients can connect to the Service IP (a.k.a ClusterIP) and port which is routed to one of the Pods backing the service.
3. Services don't operate at Layer 7 (HTTP level) so things like cookie based session affinity or setting HTTP headers are not available. Services deal with the TCP and UDP packets and don't care about the payload they carry (HTTP requests).
4. Services within a Pod is discoverable via environment vars or DNS (run by Kubernetes). 
5. Services don't link to Pods directly. There is a resource called EndPoints inbetween them that contain the list of Pod IPs and the ports they can be accessed on for the respective service. Although `spec.selector` is defined in the Service to select the Pods, it is only used to build a list of IPs and ports. The Service proxy selects on of those IP and port pairs and redirects the incoming request to the server listening at that location.
6. The EndPoints IPs are not Node IPs; they are Pod IPs.


#### External Access

**NodePort**: 

- Reserves a port on all worker nodes and forward incoming connections to the pods that part of that service. This is handled by the kube proxy which intercepts traffic (via iptables) and retrieves the EndPoints via the Service.

**LoadBalancer**: 

- Automatically provision a LB from within the cloud infrastructure. This is still an extension of the NodePort service so behind the LB, it has the Nodes as instances with the port it needs to connect to.

**Ingress**: 

- The LoadBalancer service will create a LB with a public IP for every service. Ingress only requires 1 LB which can be backed by many services.
- Ingress operates at the HTTP layer which means you can leverage things like URL based routing.
- For the Ingress resource to work, an Ingress Controller needs to be running in the cluster. An Ingress Controller is a software LB like NGINX or HAProxy.


#### Internal Access

**ClusterIP**

- Only accessible within the cluster e.g: Bitbucket uses Elasticsearch as a search index which will be configured as an internal ClusterIP service.
- The default Service type is `ClusterIP`.

**None**

- Used for stateful apps to provide FQDN to Pods.
- Known as a 'headless service'.


## Ingress


### Basic Ingress Template

{% highlight yaml %}
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myweb
spec:
  rules:
  - host: rsvp.myweb.com
    http:
      paths:
      - backend: 
          serviceName: rsvp
          servicePort: 80
  - host: home.myweb.com
    http:
      paths:
      - backend: home
          serviceName: 
          servicePort: 80
{% endhighlight %}


#### Notes

- When a request lands on the Ingress Controller, it looks up the ingress rules and uses the Service to find the EndPoints (discussed earlier) to find the Pods.


## Deployment

### Basic Deployment Template

{% highlight yaml %}
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      labels: 
        app: webserver
    spec:
      containers:
        - name: webserver
          image: nginx:alpine
          ports:
          - containerPort: 80
{% endhighlight %}

#### Notes

- When a Deployment is created, a ReplicaSet resource is created for each deployment. The ReplicaSets also manage the underlying Pods.
- Default deployment strategy is to perform a rolling updated (`RollingUpdate`). The alternative is to recreate (`Recreate`) which deletes all the old Pods and then create new ones.
- A new ReplicaSet is created when you need to make changes to the Pod templates. The ReplicaSet is then managed by the Deployment.
- Modifying a ConfigMap will not trigger an updated deployment in anyway.
- ReplicaSets will always create identitical Pods (apart from names and IPs). If the Pod template includes requirements for a persistent volume (PV), all Pods will be bound to that same storage volume. This means each Pod cannot have its own seperate persistent storage (This is solved by `StatefulSets`).


## StatefulSets

### Basic StatefulSet Template

#### Notes

- Used for managing stateful apps. You could think of them as 'pets'.
- Unlike ReplicaSet (part of Deployment) which launches completely new Pods because the data is stateless, StatefulSets ensures that the new Pod is launced with the same name, DNS name and the state of the application.
- Each Pod has its own persistent volume (storage) which is different from its peers e.g: Bitbucket requires each cluster node to have a seperate `home` directory.
- Each Pod gets a stable name and can be predicted when you scale up and down i.e: For 2 Bitbucket replicas you will get Pod names `bitbucket-0` & `bitbucket-1`. The name is derived from the StateSet name and is assigned zero-based ordinal index.
- Pods are always started in order, and the next Pod is only started when the previous Pod has started and is ready to accept traffic.
- To address a Pod by its hostname for stable network identity, the StatefulSet requires you to create headless Service that is used to provide a FQDN like `bitbucket-0.bitbucket-headless.default.svc.cluster.local` where the headless Service is called `bitbucket-headless` and we use the `default` Namespace.
- When scaling  StatefulSets, the next ordinal index will be used so it is predictable. Similarly when scaling down, it starts removing the Pod with the highest ordinal index.
- When scaling down it is done at 1 Pod at a time.
- In terms of persistent storage, you define a Pod template and a PVC (Persistent Volume Claim) template. This will result in each Pod getting its own stateful storage (Persistent Volume). If a Pod is deleted, the PV (Persistent Volume) will not be deleted due to importance of stateful data. When the Pod is brought back up, it will automatically bind to the same PV.
- Must be bound to a headless Service.
- If a Pod needs to be recheduled because it has been evicted or deleted, there are no gurantees that it will be created on the same Node. However the Pods identity is still left intact.
- Peer discovery is via DNS SRV record lookups

Example:

In the following we do a SRV lookup where `bitbucket-internal` is the Service and `stateful-test` is the Namepsace.

{% highlight bash %}
$ kubectl run -it srvlookup --image=tutum/dnsutils --rm --restart=Never -- dig SRV bitbucket-internal.stateful-test.svc.cluster.local
{% endhighlight %}

