# Enable Istio ingress controller in k8s cluster

> This demo contains Istio version `1.1.7`, below all the commands needs be executed in the folder `istio-1.1.7`. Important thing need to be repeated 3 times: `istio-1.1.7`, `istio-1.1.7`, `istio-1.1.7`!

## 1. Create aks in azure
You can do this in azure portal

## 2. Connect aks cluster with kubectl

```
az login
az aks get-credentials --resource-group group_name --name cluster_name
kubectl version
```

## 3. Deploy Istio

Run below the command to install 40+ custom resources
```
$ for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
```

Then run below the command to install huge amount of config maps, secrets, namespaces `istio-system`, service accounts and other resources. It is kind of scary comparing with the small number of dependencies Ambassador needs.

```
$ kubectl apply -f install/kubernetes/istio-demo.yaml
```

Keep running below the command to make sure the istio gateway load balancer has external IP assigned
```
$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)                                                                                                                                      AGE
istio-ingressgateway   LoadBalancer   10.0.80.157   104.210.114.64   15020:32280/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:30221/TCP,15030:31705/TCP,15031:31534/TCP,15032:31105/TCP,15443:31994/TCP   173m
```

Also make sure all pods are running or complete
```
$ kubectl get pods -n istio-system
NAME                                      READY   STATUS      RESTARTS   AGE
grafana-7f4d444dd5-627gj                  1/1     Running     0          12m
istio-citadel-7f447d4d4b-9x98v            1/1     Running     0          12m
istio-cleanup-secrets-1.1.7-zlq9d         0/1     Completed   0          12m
istio-egressgateway-5cc54d4f95-p6r8m      1/1     Running     0          12m
istio-galley-84749d54b7-v52gm             1/1     Running     0          12m
istio-grafana-post-install-1.1.7-lhst9    0/1     Completed   0          12m
istio-ingressgateway-7db4b6c8f4-p8qsc     1/1     Running     0          12m
istio-pilot-6b6d445b9b-vr9cq              2/2     Running     0          12m
istio-policy-67b99cdbf4-vrttl             2/2     Running     6          12m
istio-security-post-install-1.1.7-2zxwt   0/1     Completed   0          12m
istio-sidecar-injector-6895997989-dkbkl   1/1     Running     0          12m
istio-telemetry-d6c78b4b-z8g2z            2/2     Running     5          12m
istio-tracing-79db5954f-v4drl             1/1     Running     0          12m
kiali-68677d47d7-b9z7h                    1/1     Running     0          12m
prometheus-5977597c75-2flxs               1/1     Running     0          12m
```

## 4. Deploy the application

Istio has a mechanism called `Istio sidecar injector`, if your target namespace is labeled with `istio-injection=enabled`, then when you deploy your application using `kubectl apply` the Istio sidecar injector will automatically inject Envoy containers into your application pods.

We are deploying our app to default namespace, so we first need to put right label on it.

```
$ kubectl label namespace default istio-injection=enabled
```

Then we are ready to deploy the app it self, here we are using the sample Bookinfo app, run below the command to deploys 4 services.
```
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
$ kubectl get services
NAME                       CLUSTER-IP   EXTERNAL-IP   PORT(S)              AGE
details                    10.0.0.31    <none>        9080/TCP             6m
kubernetes                 10.0.0.1     <none>        443/TCP              7d
productpage                10.0.0.120   <none>        9080/TCP             6m
ratings                    10.0.0.15    <none>        9080/TCP             6m
reviews                    10.0.0.170   <none>        9080/TCP             6m
```

Quickly check the product page container is running

```
kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
```

## 5. Create routing by using `Istio Gateways`

```
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

The `bookinfo-gateway.yaml` defines below the routings all pointing to service (destination) `productpage:9080`

```
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

## 6. Confirm the app is accessible from outside the cluster

We can use the external IP created at step 3 to call our service

```
curl -s 104.210.114.64/productpage | grep -o "<title>.*</title>"
```

## 7. Content based routing 

> Will do when have time