# Set up azure k8s service
## 1. Create aks in azure
You can do this in azure portal

## 2. Connection aks cluster with kubectl
```
az login
az aks get-credentials --resource-group group_name --name cluster_name
```

## 3. Deploy 2 services by using the attached yaml files
```
$ kubectl apply -f ingress-default-backend.yaml 
kubectl adeployment.extensions/ingress-default-backend created
ppservice/ingress-default-backend created

$ kubectl apply -f http-svc.yaml 
deployment.extensions/http-svc created
service/http-svc created

$ kubectl apply -f http-svc-ingress.yaml
ingress.extensions/app created

```

<aside class="notice">
...

Unfortunatelly after multple rounds of trying I failed test HAProxy ingress controller on a newly started azure k8s cluster. So I give up.

But at least we know the HAProxy ingress controller is very similar to Nginx ingress controller, that we need create a ingress resource which defines the host and match.
</aside>




# Reference
>[HAProxy Ingress Controller for Kubernetes](https://www.haproxy.com/blog/haproxy_ingress_controller_for_kubernetes/)  
[Deploying HAProxy Ingress Controller](https://github.com/jcmoraisjr/haproxy-ingress/tree/master/examples/deployment)