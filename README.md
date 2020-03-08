# Overview

We're running all MVP Studio projects in Kubernetes on [Rancher](https://rancher.com/docs/rancher/v2.x/en/overview/).
This repository contains non-project specific configuration scripts, Kubernetes configuration files, etc. that allow you
to provision a cluster to work with the rest of the MVP Studio system.

## Accessing Rancher and Setting up Kubectl

In order to access the cluster you must first VPN into the Continu cluster. See the
[VPN_README.md](./docs/VPN_README.md) in this directory for details. Assuming you have a running VPN connection then you
can connect to the master node.

The master node is also running the main rancher container:
```
https://172.19.250.22:8443/
```

The credentials to access Rancher can be found in the standard MVP Studio secrets bucket (`passwords.mvpstudio.org`
bucket on GCE). Once in Rancher you can click the `Kubeconfig file` link to get a `~/.kube/config` file. You can also
find the ssh credentials for the nodes in that bucket in the `continu-k8.txt` file.

## Directories

The subdirectories in here are:

* bootstrap: these are configurations and scripts that need to be run exactly once when a new cluster is created.
* running: anything that is running in our production cluster belongs here. That is, if you haven't changed anything
  then running `kubectl apply -f .` from this directory shouldn't change anything.

**Never** modify cluster state or resources manually: that makes it very hard to recreate the cluster state for a new
cluster and leaves no audit trail. Instead, add, modify, or delete files here. If done correctly we should be able to
exactly re-create the state of our cluster (in another region for example in case of an outage) but `kubectl apply -f .`
in the `boostrap` directory followed by `kubectl apply -f .` in the `running` directory.

# Cluster Components

The cluster contains many components. These are documented is the following sub-documents:

* [Ambassador](./docs/AMBASSADOR.md) is how we route all traffic in the cluster.
* [Hardware](./docs/HARDWARE.md) documents the hardware provided to us by Continu and how we've configured it.
* [Storage](./docs/STORAGE.md) documentation on how we've configured the SAN array and created a
  [Ceph](https://docs.ceph.com/docs/master/start/intro/) cluster from it that can be used for persistent volumes.

