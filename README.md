
## Create myecho demo

```bash
kubectl apply -f 1.namespace.yaml -f 2b.service.yaml  -f 2a.pods_echoheaders_debug.yaml
kubectl exec -ti $(kubectl get po -l app=opsutils -o name  -n stubborn-ambient )  -n stubborn-ambient -- curl  http://debug-svc-myecho
```

Myecho service is up

```json
{
  "path": "/",
  "headers": {
    "host": "debug-svc-myecho",
    "user-agent": "curl/8.12.1",
    "accept": "*/*",
    "x-forwarded-proto": "http",
    "content-length": "0"
  },
  "method": "GET",
  (...)
}
```




## Enable RBAC (KO)

The aim is to prevent HTTP GET on the `debug-svc-myecho` service.

1. Add AuthorizationPolicy
2. Modify the service to link it to the Waypoint via the label `istio.io/use-waypoint: mywaypoint`.
3.  Waypoint created with label `istio.io/waypoint-for: service`.

The service remains accessible even though it should be blocked by RBAC.


```bash
kubectl apply -f 4.gtw-waypoint.yaml -f 5b.AuthorizationPolicy-post.yaml
kubectl exec -ti $(kubectl get po -l app=opsutils -o name  -n stubborn-ambient )  -n stubborn-ambient -- curl http://debug-svc-myecho
```



## working solution

By moving the service label `debug-svc-myecho` to the namespace, and changing the Waypoint type from 'service' to 'workload', it works.
HTTP GET access is denied.

```bash
kubectl apply -f 1.namespace.yaml -f 2b.service.yaml  -f 4.gtw-waypoint.yaml 
kubectl exec -ti $(kubectl get po -l app=opsutils -o name  -n stubborn-ambient )  -n stubborn-ambient -- curl http://debug-svc-myecho
> RBAC: access denied
```

And if you use the authorized HTTP POST method, it works as expected.

```bash
kubectl exec -ti $(kubectl get po -l app=opsutils -o name  -n stubborn-ambient )  -n stubborn-ambient -- curl -X POST http://debug-svc-myecho
```

```json
{
  "path": "/",
  "headers": {
    "host": "debug-svc-myecho",
    "user-agent": "curl/8.12.1",
    "accept": "*/*",
    "x-forwarded-proto": "http",
    "content-length": "0"
  },
  "method": "POST",
  (...)
}
```



## Delete demo

```bash
kubectl delete namespace stubborn-ambient
```