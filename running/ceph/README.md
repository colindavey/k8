# Overview

This is the pods, CRDs, etc. for our [Ceph](https://docs.ceph.com/docs/master/start/intro/) cluster. It was set up via
[Rook](https://rook.io/docs/rook/v1.2/). Specifically we started with the code in Rook's repo:

```
git clone --single-branch --branch release-1.2 https://github.com/rook/rook.git
cd cluster/examples/kubernetes/ceph
```

And we then modified the `yml` files in there to fit our needs. The modified files have been copied to this directory.

Per [the Rook documentation](https://rook.io/docs/rook/master/ceph-cluster-crd.html), our `dataDirHostPath`, which
is set to `/var/lib/rook`, holds keys, configs, etc. so if you tear down the cluster and restart it you must
delete this directory too.

# Starting

To start the cluster you need to start things in order. First:

```
kubectl apply -f common.yml
```

to set up the custom resource definitions, roles, and role bindings. Then:

```
kubectl apply -f operator.yml
```

to deploy the operator. Note that the storage volumes for the operator are defined as:

```
      volumes:
      - name: rook-config
        emptyDir: {}
      - name: default-config-dir
        emptyDir: {}
```

it's not at all clear from the documentation but it seems that the operator does not need to maintain state anywhere for
the cluster to be robust. I tried killing the operator and letting it restart and the cluster was just fine.
Then create the cluster:

```
kubectl apply -f cluster.yml
```

Once the cluster is up you should be able to [view the cluster status](#status). Note that it takes several minutes for
the operator to create all the services and pods so don't worry if the dashboard or other components aren't available
immediately.


# Status

To view the status of the Rook cluster you can first set up port forwarding to the dashboard service:

```
kubectl -n rook-ceph port-forward service/rook-ceph-mgr-dashboard 7000
```

then point your browser at `http://localhost:7000/#/login`. To get the login credentials you can run:

```
kubectl -n rook-ceph get secret rook-ceph-dashboard-password \
    -o jsonpath="{['data']['password']}" | base64 --decode && echo
```

The username is `admin` and the above command will give you the password.
