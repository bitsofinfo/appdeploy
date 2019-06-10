# appdeploy

This is a Helm chart that can deploy an single application
Docker image in a standard convention based way.

# IN PROGRESS

## Important!

In an of itself, this chart deploy's nothing its default [values.yaml](values.yaml) is sparse on
actual app specific configuration, i.e. its `image.repository`, `image.tag`, `env` and `command` are empty.

## What it does

This chart produces the following "version specific" Kubernetes YAML objects that adhere
to a general convention described above (TBD)

*"Version specific"*, meaning all artifacts produced with this chart will be
appropriately named according to the `[appname]-[context]-[version][-[classifier]]`
naming conventions.

Produced version specific artifacts:

* **RBAC**: optional RBAC `ServiceAccount`, `[Cluster]Role`, `[Cluster]RoleBinding`
* **Secret**: for the app's bootstrap secret
* **Deployment**: for the app's Pod specification
* **Service**: to access all the app's ports
* **Ingress**: one or more, depending on the apps `containerPorts` configuration
* **Helm Hooks**: Post deploy/delete checks and alerts to slack

## Post install/upgrade/delete checks and alerts

The checking and alerting engine leverages: https://github.com/bitsofinfo/kubernetes-helm-healthcheck-hook

## Configurable options

All configurable options are documented in [values.yaml](values.yaml)

See the [helm docs for setting values](https://github.com/helm/helm/blob/master/docs/chart_best_practices/values.md)
on how to customize/change these values when doing a `helm ...` invocation.

## meta-variable subsitution

Certain `values` can contain "meta-variables" that will be parsed into real values which
can be useful when specifying `env` vars or `command.args` values.

* `[[#app.name]]`: replaced with value of `.Values.app.name`
* `[[#app.context]]`: replaced with value of `.Values.app.context`
* `[[#app.environment]]`: replaced with value of `.Values.app.environment`
* `[[#app.classifier]]`: replaced with value of `.Values.app.classifier`
* `[[#image.tag]]`: replaced with value of `.Values.image.tag`
* `[[#namespace]]`: replaced with value of `.Release.Namespace`
* `[[#fullAppIdentifier]]`: replaced with value of `[app.name]-[app.context]-[image.tag | replace "." "-"][-[app.classifier]]`
* `[[#bootstrapSecretFilePath]]`: replaced with value of `.Values.bootstrapSecret.mount.path/.Values.bootstrapSecret.mount.fileName`


## Simple example

Here is a simple standalone example to simply show what would be generated:

(take out `--dry-run` to actually deploy it...)

This is not realistic, but simply a demo that the chart can function on its own
as well.

```
helm install \
  --dry-run \
  --debug \
  --namespace my-apps \
  --name myapp-1.0 \
  basicapp \
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
  --set ingress.metadata.annotations[0].name="my-external-dns" \
  --set ingress.metadata.annotations[0].value="enabled" \
  --set ingress.metadata.annotations[1].name="kubernetes.io/ingress.class" \
  --set ingress.metadata.annotations[1].value="traefik" \
  --set ingress.dns.fqdnSuffix=".local" \
  --set ingress.metadata.labels[0].name="my-ingress" \
  --set ingress.metadata.labels[0].value="internal" \
  --set hooks.postDelete.hookDeletePolicy="before-hook-creation" \
  --set hooks.postInstallUpgrade.hookDeletePolicy="before-hook-creation"

helm list

# assumes you have external-dns running for this domain OR a hosts file entry
curl -k -v http://myapp-stage-context1-latest-80.local

helm delete --purge myapp-1.0
```

## As a dependency in another chart

Build it: `helm package appdeploy`

Then copy the resulting `tgz` into your dependant chart's `charts/` folder

From there you can customize your chart's `values.yaml` to further refine the
defaults provided by `appdeploy/values.yaml`

For example, lets say you create a deriviative chart based on `appdeploy`.
Your derivative chart's `values.yaml` could further provide defaults on top
of `appdeploy` as follows:

**my-custom-chart/values.yaml**
```
my-own-chart-property1: "some value"

# Specify value customizations for the basicapp dependency
# To override the values of "sub-charts" you have to scope them
# as follows:
# https://github.com/helm/helm/blob/master/docs/chart_template_guide/subcharts_and_globals.md

appdeploy:
  replicaCount: 20

  containerPorts:
    - port: "3333"
      name: "my-https"
      ingress: true
      service: true
      tls: true

  healthcheck:
    liveness:
      containerPort: 3333
    readiness:
      containerPort: 3333
```


# Using

```
helm plugin install https://github.com/aslafy-z/helm-git.git
helm repo add appdeploy-master git+https://github.com/bitsofinfo/appdeploy@?ref=master
```
