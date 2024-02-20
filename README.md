# Kubernetes is Insecure by Default (And What You Can Do About it)

I want to be clear and say that I am very grateful for all the work that the community has done to build Kubernetes. I hesitate to even think about how many person-hours have gone into the project. The fact that Kubernetes is "insecure by default" is not a value judgement, and I'm not even complaining, I just think it's a fact. Various design decisions have been made to make it easier to get started with Kubernetes, and these decisions have been very successful in terms of adoption. So where vanilla, standard Kubernetes begins, in terms of default configuration, is simply a starting point that most will work to modify and adapt to their organisation's needs.

This repo is an attempt to build a basic 'secure by default' stance for Kubernetes, which is of course based on my opinion, and not everyone will agree.

## What Will We Do?

Kubernetes has made design decisions which, depending on your opinion, might lead to a "insecure by default" stance when it is first deployed.

I suggest there are two main reasons for an initial, default, insecure stance:

1. Allowing containers to run as root, for example when we do something basic like `kubectl run nginx --image=nginx` that container will run as root.
2. In general, by default, any pod can talk to any other pod.

To build an opinionated more "secure by default" stance, we're going to automate the following:

1. Configure a network policy that only allows pods in the same namespace to talk to each other and to the Internet.
1. When a new namespace is created, this network policy is automatically applied to it.
1. When a new namespace is created, the PSA restricted label is automatically applied to it.
1. For fun, when a plain nginx container is created, the image is swapped out for nginx-unprivileged. (This of course wouldn't be done in the real world, but it's an important note because not all container images can run as non-root.)

## Install Policies

This assumes Kyverno has been installled.

Install:

```
cd policies
kubectl create -f add-default-security-context-to-pod.yaml
kubectl create -f add-network-security-policy-to-namespaces.yaml
kubectl create -f add-restricted-psa-to-namespaces.yaml
kubectl create -f namespaces.yaml
kubectl create -f swap-nginx-image-for-nginx-unprivileged.yaml
```

Update:

```
cd policies
kubectl delete clusterpolicy add-network-security-policy-to-namespaces
kubectl apply -f add-default-security-context-to-pod.yaml
kubectl apply -f add-network-security-policy-to-namespaces.yaml
kubectl apply -f add-restricted-psa-to-namespaces.yaml
kubectl apply -f namespaces.yaml
kubectl apply -f swap-nginx-image-for-nginx-unprivileged.yaml
```

