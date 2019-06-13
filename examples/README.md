# Example

Below is a basic example that you can deploy anything w/ this chart. In this case we don't utilize the `bootstrapSecret` as its not necessary for this demo. Post deploy/delete alerts will be sent to the https://bitsofinfo.slack.com `#bitsofinfo-dev` channel

This should be enough to get your feet wet.

### Setup

You have an [Traefik Ingress Controller Deployed](https://github.com/helm/charts/tree/master/stable/traefik).

*Note Traefik is not a requirement to use `appdeploy` but just the one picked for this example...*
```
helm install stable/traefik --name appdeploy-traefik \
  --namespace kube-system \
  --set dashboard.enabled=true \
  --set ssl.enabled=true \
  --set ssl.insecureSkipVerify=true \
  --set accessLogs.enabled="true" \
  --set rbac.enabled="true" \
  --set kubernetes.labelSelector="appdeploy-ingress=yes" \
  --set kubernetes.ingressEndpoint.useDefaultPublishedService=true \
  --set dashboard.domain=appdeploy.test.local
```

Label the Traefik dashboard Ingress w/ `appdeploy-ingress=yes` so it will be recognized by the controller
```
kubectl label ingress -n kube-system appdeploy-traefik-dashboard appdeploy-ingress=yes
```

You have a host entry (or legit DNS) setup for the `LoadBalancer` for the installed Traefik Ingress Controller `Service` above
```
kubectl get services -n kube-system | grep appdeploy-traefik
```

Add to your local `/etc/hosts` (unless you have DNS setup)
```
[appdeploy-traefik-controller-lb-ip] appdeploy.test.local myapp-stage-context1-latest-80.local
```

Now lets hit the Traefik dashboard to see all auto-configured frontends/backends via `Ingress`: https://appdeploy.test.local

Create a new namespace for our app:
```
kubectl create namespace my-apps
```

Lets deploy an dummy app using the chart!
```
helm install \
  --debug \
  --namespace my-apps \
  --name myapp-1.0 \
  bitsofinfo-appdeploy/appdeploy --version 1.0.1 \
  --set image.repository="nginx" \
  --set image.tag="latest" \
  --set app.name="myapp" \
  --set app.environment="stage" \
  --set app.context="stage-context1" \
  --set containerPorts[0].service=true \
  --set containerPorts[0].port=80 \
  --set containerPorts[0].port=80 \
  --set containerPorts[0].name=nginx80 \
  --set containerPorts[0].ingress=true \
  --set containerPorts[0].tls=false \
  --set healthcheck.liveness.containerPort=80 \
  --set healthcheck.liveness.path=/ \
  --set healthcheck.liveness.scheme=HTTP \
  --set healthcheck.readiness.containerPort=80 \
  --set healthcheck.readiness.path=/ \
  --set healthcheck.readiness.scheme=HTTP \
  --set hooks.postInstallUpgrade.checksConfig[0].path=/ \
  --set hooks.postInstallUpgrade.checksConfig[0].name="/" \
  --set hooks.postInstallUpgrade.checksConfig[0].method="GET" \
  --set hooks.postInstallUpgrade.checksConfig[0].timeout=5 \
  --set hooks.postInstallUpgrade.checksConfig[0].retries=15 \
  --set-string hooks.postInstallUpgrade.checksConfig[0].tags[0]=80 \
  --set hooks.postInstallUpgrade.useIngressHost=false \
  --set hooks.postDelete.useIngressHost=false \
  --set ingress.metadata.annotations[0].name="my-external-dns" \
  --set ingress.metadata.annotations[0].value="enabled" \
  --set ingress.metadata.annotations[1].name="kubernetes.io/ingress.class" \
  --set ingress.metadata.annotations[1].value="traefik" \
  --set ingress.dns.fqdnSuffix=".local" \
  --set ingress.metadata.labels[0].name="appdeploy-ingress" \
  --set ingress.metadata.labels[0].value="yes"
```

Wait for the install to complete:
```
helm list

kubectl get all -n my-apps

kubectl get ingress -n my-apps

kubectl get jobs -n my-apps
```

You should see a notification in the https://bitsofinfo.slack.com `#bitsofinfo-dev` channel

Check traefik dashboard: https://appdeploy.test.local/dashboard/ to see the deployed backend

Hit the actual deployed artifact: http://myapp-stage-context1-latest-80.local (assuming you have DNS setup) You should get a NGINX welcome page


Cleanup:
```
helm delete --purge myapp-1.0

helm delete --purge appdeploy-traefik
```

You should see a app delete notification in the https://bitsofinfo.slack.com `#bitsofinfo-dev` channel
