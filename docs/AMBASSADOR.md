# Ambassador

All of our cluster routing is handled by [Ambassador](https://www.getambassador.io/). If you like to debug the Amassador
routing you must first find a running ambassador pod like:

```
$ kubectl get --namespace=ambassador pods
NAME                         READY   STATUS    RESTARTS   AGE
ambassador-8db4cb58d-gsmjj   1/1     Running   0          34d
ambassador-8db4cb58d-t9q2b   1/1     Running   0          34d
ambassador-8db4cb58d-tfqnn   1/1     Running   0          34d
```

Obviously the actual pod names will likely be different when you run this. Once you have the list of pods select one and
set up port forwarding:

```
$ kubectl --namespace=ambassador port-forward ambassador-8db4cb58d-gsmjj 8877 8877
```

and you can then point your browser to `http://localhost:8877/ambassador/v0/diag/` to see diagnostics.
