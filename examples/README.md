# Example

Below is a basic example that you can deploy anything w/ this chart. In this case we don't utilize the `bootstrapSecret` as its not necessary for this demo. Post deploy/delete alerts will be sent to the https://bitsofinfo.slack.com `#bitsofinfo-dev` channel ([self signup to channel](https://join.slack.com/t/bitsofinfo/shared_invite/enQtNjY1ODIzNTkyMDMyLTEzZGUwNzExOWYyMmZmMTQyYWZiYzJjYTJkNGI3MWMzNzQ3MTE2NzVhM2Q1ZjE4OGViYjA1NGY4MzdiZDg3ZWI))

This should be enough to get your feet wet.

### Setup

Add to your local `/etc/hosts` (unless you have DNS setup)
```
[bitsofinfo-traefik-controller-lb-ip] bitsofinfo-traefik.test.local myapp-stage-context1-latest-80.local
```

**IMPORTANT!**: Before we continue we need to setup an `IngressController` [lets use Traefik, click here for setup instructions](TRAEFIK_SETUP.md)

Next, lets ensure the Helm repo exists on your machine:
```
helm repo add bitsofinfo-appdeploy https://raw.githubusercontent.com/bitsofinfo/appdeploy/master/repo
helm repo update
```

Lets deploy a dummy app using the chart!
```
helm install \
  --debug \
  --namespace bitsofinfo-apps \
  --name myapp-1.0 \
  bitsofinfo/appdeploy --version 1.1.0 \
  --set image.repository="nginx" \
  --set image.tag="latest" \
  --set app.name="myapp" \
  --set app.environment="stage" \
  --set app.context="stage-context1" \
  --set aliases[0]="myapp-alias1" \
  --set aliases[1]="myappalias2" \
  --set aliases[2]="otheralias3" \
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
  --set hooks.default.postInstallUpgrade.validator.checksConfig[0].path=/ \
  --set hooks.default.postInstallUpgrade.validator.checksConfig[0].name="/" \
  --set hooks.default.postInstallUpgrade.validator.checksConfig[0].method="GET" \
  --set hooks.default.postInstallUpgrade.validator.checksConfig[0].timeout=5 \
  --set hooks.default.postInstallUpgrade.validator.checksConfig[0].retries=15 \
  --set-string hooks.default.postInstallUpgrade.validator.checksConfig[0].tags[0]=80 \
  --set hooks.default.postInstallUpgrade.validator.useIngressHost=false \
  --set hooks.default.postDelete.validator.useIngressHost=false \
  --set ingress.metadata.annotations[0].name="my-external-dns" \
  --set ingress.metadata.annotations[0].value="enabled" \
  --set ingress.metadata.annotations[1].name="kubernetes.io/ingress.class" \
  --set ingress.metadata.annotations[1].value="traefik" \
  --set ingress.dns.fqdnSuffix=".local" \
  --set ingress.metadata.labels[0].name="bitsofinfo-ingress" \
  --set ingress.metadata.labels[0].value="yes"

```

Wait for the install to complete:
```
helm list

kubectl get all -n bitsofinfo-apps

kubectl get ingress -n bitsofinfo-apps

kubectl get jobs -n bitsofinfo-apps
```

You should see a notification in the https://bitsofinfo.slack.com `#bitsofinfo-dev` channel ([self signup to channel](https://join.slack.com/t/bitsofinfo/shared_invite/enQtNjY1ODIzNTkyMDMyLTEzZGUwNzExOWYyMmZmMTQyYWZiYzJjYTJkNGI3MWMzNzQ3MTE2NzVhM2Q1ZjE4OGViYjA1NGY4MzdiZDg3ZWI))

Check traefik dashboard: https://bitsofinfo-traefik.test.local/dashboard/ to see the deployed backend

Hit the actual deployed artifact at: (assuming you have DNS setup) You should get a NGINX welcome page
* http://myapp-stage-context1-latest-80.local
* http://myapp-alias1-stage-context1-latest-80.local
* http://myappalias2-stage-context1-latest-80.local
* http://otheralias3-stage-context1-latest-80.local


Cleanup:
```
helm delete --purge myapp-1.0

helm delete --purge bitsofinfo-traefik
```

You should see a app delete notification in the https://bitsofinfo.slack.com `#bitsofinfo-dev` channel ([self signup to channel](https://join.slack.com/t/bitsofinfo/shared_invite/enQtNjY1ODIzNTkyMDMyLTEzZGUwNzExOWYyMmZmMTQyYWZiYzJjYTJkNGI3MWMzNzQ3MTE2NzVhM2Q1ZjE4OGViYjA1NGY4MzdiZDg3ZWI))
