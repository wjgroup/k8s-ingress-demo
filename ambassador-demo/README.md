# Enabled Ambassador gateway in k8s cluster
## 1. Create aks in azure
You can do this in azure portal

## 2. Connection aks cluster with kubectl
```
az login
az aks get-credentials --resource-group group_name --name cluster_name
```

## 3. check if cluster has RBAC enabled
```
kubeclt api-versions
```

If you see something like `rbac.authorization.k8s.io/v1` in it then RBAC is enabled

## 4. Deploy Ambassador
```
$ kubectl apply -f https://getambassador.io/yaml/ambassador/ambassador-rbac.yaml
service/ambassador-admin created
clusterrole.rbac.authorization.k8s.io/ambassador created
serviceaccount/ambassador created
clusterrolebinding.rbac.authorization.k8s.io/ambassador created
customresourcedefinition.apiextensions.k8s.io/authservices.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/mappings.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/modules.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/ratelimitservices.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/tcpmappings.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/tlscontexts.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/tracingservices.getambassador.io created
```

From above the output you see list of resources have been created after the installation.

## 3. Deploy 2 services by using the attached yaml files
```
$ kubectl apply -f ingress-default-backend.yaml 
kubectl adeployment.extensions/ingress-default-backend created
ppservice/ingress-default-backend created

$ kubectl apply -f http-svc.yaml 
deployment.extensions/http-svc created
service/http-svc created
```
