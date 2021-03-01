# Traefik Ingress Controller setup for examples

Lets get a [Traefik Ingress Controller Deployed](https://github.com/helm/charts/tree/master/stable/traefik).

*Note Traefik is not a requirement to use `appdeploy` but just the one picked for this example...*
```
helm install bitsofinfo-traefik stable/traefik \
  --namespace kube-system \
  --set dashboard.enabled=true \
  --set ssl.enabled=true \
  --set ssl.insecureSkipVerify=true \
  --set accessLogs.enabled="true" \
  --set rbac.enabled="true" \
  --set kubernetes.labelSelector="bitsofinfo-ingress=yes" \
  --set kubernetes.ingressEndpoint.useDefaultPublishedService=true \
  --set dashboard.domain=bitsofinfo-traefik.test.local
```

Label the Traefik dashboard Ingress w/ `bitsofinfo-ingress=yes` so it will be recognized by the controller
```
kubectl label ingress -n kube-system bitsofinfo-traefik-dashboard bitsofinfo-ingress=yes
```

You have a host entry (or legit DNS) setup for the `LoadBalancer` for the installed Traefik Ingress Controller `Service` above.

**IMPORTANT**:
*NOTE! since the Traefik chart deploys a `Service` of type `LoadBalancer` for the Ingress Controller, you need to be deploying to a Kubernetes cluster that supports provisioning some sort of `LoadBalancer`. If your target k8s cluster does NOT support `LoadBalancer` types, such as Minikube, then you can install something like [akrobateo](https://github.com/kontena/akrobateo) which will give you a psuedo load balancer capability on platforms like Minikube: https://github.com/kontena/akrobateo.*
*If [akrobateo](https://github.com/kontena/akrobateo) is giving you problems try using [minikube-lb-patch](https://github.com/elsonrodriguez/minikube-lb-patch). You will basically have to run the following two lines:*
```
sudo route -n add -net $(cat ~/.minikube/profiles/minikube/config.json | jq -r ".KubernetesConfig.ServiceCIDR") $(minikube ip)
kubectl run minikube-lb-patch --replicas=1 --image=elsonrodriguez/minikube-lb-patch:0.1 --namespace=kube-system
```

Alternatively you can also just do `minikube service bitsofinfo-traefik -n kube-system` 

List the Traefik services
```
kubectl get services -n kube-system | grep bitsofinfo-traefik
```

Now lets hit the Traefik dashboard to see all auto-configured frontends/backends via `Ingress`: https://bitsofinfo-traefik.test.local

Create a new namespace for our apps:
```
kubectl create namespace bitsofinfo-apps
```
