# Overview

We're running all MVP Studio projects in Kubernetes on [GKE](https://cloud.google.com/kubernetes-engine/). This
repository contains non-project specific configuration scripts, Kubernetes configuration files, etc. that allow you to
provision a cluster to work with the rest of the MVP Studio system.

# Directories

The subdirectories in here are:

* bootstrap: these are configurations and scripts that need to be run exactly once when a new cluster is created.
* running: anything that is running in our production cluster belongs here. That is, if you haven't changed anything
  then running `kubectl apply -f .` from this directory shouldn't change anything.

**Never** modify cluster state or resources manually: that makes it very hard to recreate the cluster state for a new
cluster and leaves no audit trail. Instead, add, modify, or delete files here. If done correctly we should be able to
exactly re-create the state of our cluster (in another region for example in case of an outage) but `kubectl apply -f .`
in the `boostrap` directory followed by `kubectl apply -f .` in the `running` directory.
