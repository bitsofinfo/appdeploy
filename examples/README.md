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

Let's deploy a dummy app using the chart!
```
helm install \
  --debug \
  --namespace bitsofinfo-apps \
  --name myapp-1.0 \
  bitsofinfo-appdeploy/appdeploy --version 1.1.5 \
  -f example.yaml
```

Wait for the install to complete:
```
helm list

kubectl get all -n bitsofinfo-apps

kubectl get ingress -n bitsofinfo-apps

kubectl get jobs -n bitsofinfo-apps

kubectl get secrets -n bitsofinfo-apps
```

You should see the `hooks.custom.prehook`, `hooks.custom.posthook` and `hooks.default.postInstallUpgrade` notifications in the https://bitsofinfo.slack.com `#bitsofinfo-dev` channel ([self signup to channel](https://join.slack.com/t/bitsofinfo/shared_invite/enQtNjY1ODIzNTkyMDMyLTEzZGUwNzExOWYyMmZmMTQyYWZiYzJjYTJkNGI3MWMzNzQ3MTE2NzVhM2Q1ZjE4OGViYjA1NGY4MzdiZDg3ZWI))

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

You should see a app delete notification (`hooks.default.postDelete`) in the https://bitsofinfo.slack.com `#bitsofinfo-dev` channel ([self signup to channel](https://join.slack.com/t/bitsofinfo/shared_invite/enQtNjY1ODIzNTkyMDMyLTEzZGUwNzExOWYyMmZmMTQyYWZiYzJjYTJkNGI3MWMzNzQ3MTE2NzVhM2Q1ZjE4OGViYjA1NGY4MzdiZDg3ZWI))
