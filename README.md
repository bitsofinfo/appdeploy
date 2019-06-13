# appdeploy

This is a fairly generic Helm chart that can deploy any Docker image to Kubernetes in an opinionated fashion.

The chart deploys a single app in an opinionated way that adheres to the conventions described below. Whereby an "app" is defined
as a single Docker `image:tag/version` to be deployed to a Kubernetes cluster where the app container resides in a `Pod` by itself as part of a `Deployment` and *optionally* may require a single `bootstrapSecret` to bootstrap itself. The `Pods` are accessible via a `Service` as well as a dedicated `Ingress` bound to an auto-generated "version specific" hostname that is unique to the app's `image:tag`.

# Conventions

In addition to whatever target k8s `namespace` the chart deploys into, all apps (single Docker `image:tag` artifacts) are labeled and categorized as follows with the following logical values. These values are made available to you in your `values` as variables you can feed into your `Deployment`. *(see `meta-variable substitution` below)*

### appname
The logical *name* for the application being deployed.

### version
The the version of the app, i.e. its `image.tag`

### environment
As one would expect, an *environment* is something like `dev`, `qa`, `prod` or `stage`.

### context
Within an *environment*, one or more *contexts* can exist as a way to logically sub-divide applications within an *environment*. The loose rule is that all applications sharing the same *context* talk to one another or similar configuration sources/databases etc. *Contexts* permit a level of classification that applications can use to alter how they bootup, configs they load etc. within an *environment*. The name of a *context* generally includes the *environment* implicit in its name (i.e. `qa-silo1`)

### classifier
Sometimes an application artifact may operate in different *modes*, these different modes may enable or disable certain aspects of standard functionality provided by the artifact; such as the availability or lack thereof certain exposed APIs or ports.

---

Again all of the above is simple a convention of this chart but does not physically *impose anything* on the artifacts you choose to deploy with this chart. These attributes are however made available to you as as variables in the chart itself that you can leverage to route into a Pod's `spec.template` commands, args and env variables; which can then subsequently be consumed to drive an app to make decisions on how to bootstrap itself, what configs to load etc. *(see `meta-variable substitution` below)*

In short, the above concepts may or may not work for you, but have proved to be a useful set of conventions used in the real world and has worked pretty well.

## Important!

In an of itself, this chart deploy's nothing as its default [values.yaml](values.yaml) is sparse on
actual configuration that would actually instantiate anything. For example its `image.repository`, `image.tag`, `env` and `command` are empty and up to you to provide the values for those items.

## What it does

This chart produces the following "version specific" Kubernetes YAML objects that adhere to a general convention described above.

*"Version specific"*, meaning all artifacts produced with this chart will be appropriately named according with the following convention: `[appname]-[context]-[image.tag][-[classifier]]` naming conventions.

Depending on your `values` customizations, this Chart can produce the following version specific artifacts:

* **RBAC**: an optional RBAC `ServiceAccount`, `[Cluster]Role`, `[Cluster]RoleBinding`
* **Secret**: for the app's `bootstrapSecret`
* **Deployment**: for the app's `Pod` specification, optionally mounting the `bootstrapSecret` and presenting as the K8s generated `ServiceAccount` above within the cluster.
* **Service**: to access all the app's `containerPorts`
* **Ingress**: one or more, depending on the apps `containerPorts` configuration with hostname naming convention `[appname]-[context]-[image.tag][-[classifier]][-[port]]` at the configured `ingress.dns.fqdnSuffix`
* **Helm Hooks**: Post deploy/delete health checks (`Jobs`) and alerts to Slack as well as additional/optional arbitrary `Jobs` for doing things like migrations etc.

## What its not intended for

This chart is NOT intended to wire your app up to one or more custom "live user facing" hostnames (i.e. www.myProdUserFacingWebsite.com) or do canary releases etc; if you are interested in that you should look at the [appconduits Helm chart](https://github.com/bitsofinfo/appconduits). The primary intent of the `appdeploy` chart is simple to deploy a new version of an app; validate it, notify the team, and provide a **version specific** `Ingress` `host` binding to access it via an Ingress Controller. If you want to route traffic to your app via other `hosts:` or sophisticated routing rules, you should be creating separate/custom `Service` and `Ingress` combinations; again see [appconduits](https://github.com/bitsofinfo/appconduits).

---

When using all available options, each invocation of `appdeploy` can produce a single point of management Helm `release` that is comprised all of the components described above. No matter how many different applications your team manages, if you follow some simple conventions as well externalizing the bulk of application specific configuration, secured via a unique (and limited life) `bootstrapSecret`; you can really begin to see the economies of scale for a unified DevOps deployment approach no matter what the artifact.

![Diagram of appdeploy](/docs/diag.png "Diagram1")


## Post install/upgrade/delete checks and alerts

The checking and alerting engine leverages: https://github.com/bitsofinfo/kubernetes-helm-healthcheck-hook

## Configurable options

All configurable options are documented in [values.yaml](values.yaml)

See the [helm docs for setting values](https://github.com/helm/helm/blob/master/docs/chart_best_practices/values.md)
on how to customize/change these values when doing a `helm ...` invocation.

## meta-variable substitution

Certain `values` can contain "meta-variables" that will be parsed into real values which
can be useful when specifying `env` vars or `command.args` values. You can use these meta-variables
directly in your `values` and they will be replaced with the real value as noted below.

* `[[#app.name]]`: replaced with value of `.Values.app.name`
* `[[#app.context]]`: replaced with value of `.Values.app.context`
* `[[#app.environment]]`: replaced with value of `.Values.app.environment`
* `[[#app.classifier]]`: replaced with value of `.Values.app.classifier`
* `[[#image.tag]]`: replaced with value of `.Values.image.tag`
* `[[#namespace]]`: replaced with value of `.Release.Namespace`
* `[[#fullAppIdentifier]]`: replaced with value of `[app.name]-[app.context]-[image.tag | replace "." "-"][-[app.classifier]]`
* `[[#bootstrapSecretFilePath]]`: replaced with value of `.Values.bootstrapSecret.mount.path/.Values.bootstrapSecret.mount.fileName`

## Examples

For examples see the [examples/ folder](examples/)


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

## Helm package/update

```
helm package . -d repo/charts/
helm repo index repo/
```

# Using

```
helm repo add bitsofinfo-appdeploy https://raw.githubusercontent.com/bitsofinfo/appdeploy/master/repo
```

```
# requirements.yaml
dependencies:
- name: appdeploy
  version: "1.0.0"
  repository: "https://raw.githubusercontent.com/bitsofinfo/appdeploy/master/repo"
```
