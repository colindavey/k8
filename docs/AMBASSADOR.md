# Ambassador

All inbound traffic to the MVP Studio cluster is handled by [Ambassador](https://www.getambassador.io/).
It operates basically as an 
[Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
, but we use the custom resources defined by Ambassador instead of the Ingress resource.

## Routing

All routing inbound to the cluster is done by Ambassador via the 
[Mapping](https://www.getambassador.io/docs/latest/topics/using/intro-mappings/) 
resource, and a good example is 
[the mapping for the mvpstudio website](../running/ambassador-edge/routing/mvpstudio-org/prod.yml). 
In this case the mapping does a simple hostname match and routes all traffic for that hostname to the correct service.

## SSL

Ambassador also handles TLS offloading, so all traffic inbound to the cluster must be encrypted (all http requests 
are redirected to https), but after Ambassador it is unencrypted. To do this we use the 
[Host](https://www.getambassador.io/docs/latest/topics/running/host-crd/) resource, which tells ambassador to setup 
a [Lets Encrypt](https://letsencrypt.org/) cert for that host using an 
[http-01 challenge](https://letsencrypt.org/docs/challenge-types/#http-01-challenge). 
Again a good example is 
[the host resource for the mvpstudio website]((../running/ambassador-edge/routing/mvpstudio-org/prod.yml)).


## Debugging

There is a console you can use with the ambassador edge cli.
Installation instructions for the cli can be found 
[here](https://www.getambassador.io/docs/latest/topics/using/edgectl/edge-control/#installing-edge-control).


Then run the following to naviate to the console:
```
edgectl login --namespace=ambassador-edge 172.19.250.20:30037 
```

There is also an admin page built in to ambassador you can view.

First find a running ambassador pod like:
```
$ kubectl get --namespace=ambassador-edge pods
NAME                         READY   STATUS    RESTARTS   AGE
ambassador-8db4cb58d-gsmjj   1/1     Running   0          34d
ambassador-8db4cb58d-t9q2b   1/1     Running   0          34d
ambassador-8db4cb58d-tfqnn   1/1     Running   0          34d
```

Obviously the actual pod names will likely be different when you run this. 
Once you have the list of pods select one and set up port forwarding:

```
kubectl port-forward deployment/ambassador -n ambassador-edge 8877
```

and you can then point your browser to `http://localhost:8877/ambassador/v0/diag/` to see diagnostics.

## Redis

If you look at the deployments in the `ambassador-edge` namespace you will notice there is a redis deployment.
This is used for a handful of features and does not require any persistent storage.
Per the [docs](https://www.getambassador.io/docs/1.3/topics/using/filters/oauth2/#redis):
> The Ambassador Edge Stack relies on Redis to store short-lived authentication credentials and rate limiting 
information. If the Redis data store is lost, users will need to log back in and all existing rate-limits would be 
reset.
