# Traefik IngressController setup for examples

Lets get a [Traefik Ingress Controller Deployed](https://github.com/helm/charts/tree/master/stable/traefik).

*Note Traefik is not a requirement to use `appdeploy` but just the one picked for this example...*
```
helm install stable/traefik --name bitsofinfo-traefik \
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

You have a host entry (or legit DNS) setup for the `LoadBalancer` for the installed Traefik Ingress Controller `Service` above
```
kubectl get services -n kube-system | grep bitsofinfo-traefik
```

Now lets hit the Traefik dashboard to see all auto-configured frontends/backends via `Ingress`: https://bitsofinfo-traefik.test.local

Create a new namespace for our apps:
```
kubectl create namespace bitsofinfo-apps
```
