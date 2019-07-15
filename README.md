# appdeploy

This is a fairly generic Helm chart that can deploy any Docker image to Kubernetes in an opinionated fashion.

The chart deploys a single app in an opinionated way that adheres to the conventions described below. Whereby an "app" is defined
as a single Docker `image:tag/version` to be deployed to a Kubernetes cluster where the app container resides in a `Pod` by itself as part of a `Deployment` and *optionally* may require a single `bootstrapSecret` to bootstrap itself. The `Pods` are accessible via a `Service` as well as a dedicated `Ingress` bound to an auto-generated "version specific" hostname that is unique to the app's `image:tag`. The chart also optionally supports creating one or more Helm Hook `Jobs`.

* [Conventions](#convention)
* [What it does](#does)
* [What its not intended for](#doesnot)
* [Alerts](#alerts)
* [Options](#options)
* [tpl-variables](#metavar)
* [Diagram](#diag)
* [Examples](examples/)
* [Custom Helm Hooks](#hooks)
* [Using as a dependency](#dependency)
* [Packaging/Using](#pack)
* [Related projects](#related)

# <a id="convention"></a>Conventions

In addition to whatever target k8s `namespace` the chart deploys into, all apps (single Docker `image:tag` artifacts) are labeled and categorized as follows with the following logical values. These values (*and any other chart value*) are made available to you to cross reference in other `values` via Helm's `tpl` evaluation function for your `Deployment`. *(see `tpl variable substitution` below)*

### appname
The logical *name* for the application being deployed.

### version
The the version of the app, i.e. its `image.tag`

### environment
As one would expect, an *environment* is something like `dev`, `qa`, `prod` or `stage`.

### context
Within an *environment*, one or more *contexts* can exist as a way to logically sub-divide applications within an *environment*. The loose rule is that all applications sharing the same *context* talk to one another or similar configuration sources/databases etc. *Contexts* permit a level of classification that applications can use to alter how they bootup, configs they load etc. within an *environment*. The name of a *context* generally includes the *environment* implicit in its name (i.e. `qa-silo1`)

### classifier
Sometimes an application artifact may operate in different *modes*, these different modes may enable or disable certain aspects of standard functionality provided by the artifact; such as the availability or lack thereof certain exposed APIs or ports. In these cases the *classifier* can simply act as a naming decoration to help identify this.

---

Again all of the above is simple a convention of this chart but does not physically *impose anything* on the artifacts you choose to deploy with this chart. These attributes (and anything else in `values.yaml`) are however made available to you as as variables in the chart itself that you can leverage to route into a Pod's `spec.template` commands, args and env variables; which can then subsequently be consumed to drive an app to make decisions on how to bootstrap itself, what configs to load etc. *(see `tpl variable substitution` below)*

In short, the above concepts may or may not work for you, but have proved to be a useful set of conventions used in the real world and has worked pretty well.

## Important!

In an of itself, this chart deploy's nothing as its default [values.yaml](values.yaml) is sparse on
actual configuration that would actually instantiate anything. For example its `image.repository`, `image.tag`, `env` and `command` are empty and up to you to provide the values for those items.

## <a id="does"></a>What it does

This chart produces the following "version specific" Kubernetes YAML objects that adhere to a general convention described above.

*"Version specific"*, meaning all artifacts produced with this chart will be appropriately named according with the following convention: `[appname]-[context]-[image.tag][-[classifier]]` naming conventions.

Depending on your `values` customizations, this Chart can produce the following version specific artifacts:

* **RBAC**: an optional RBAC `ServiceAccount`, `[Cluster]Role`, `[Cluster]RoleBinding`
* **Secret**: for the app's `bootstrapSecret`
* **Deployment**: for the app's `Pod` specification, optionally mounting the `bootstrapSecret` and presenting as the K8s generated `ServiceAccount` above within the cluster.
* **Service**: to access all the app's `containerPorts`
* **Ingress**: one or more, depending on the apps `containerPorts` configuration with hostname naming convention `[appname]-[context]-[image.tag][-[classifier]][-[port]]` at the configured `ingress.dns.fqdnSuffix`
* **Helm Hooks**: Post deploy/delete health checks (`Jobs`) and alerts to Slack as well as additional/optional arbitrary `Jobs` for doing things like migrations etc (i.e `hooks.custom.[hookname]` in `values.yaml`)

## <a id="doesnot"></a>What its not intended for

This chart is NOT intended to wire your app up to one or more custom "live user facing" hostnames (i.e. www.myProdUserFacingWebsite.com) or do canary releases etc; if you are interested in that you should look at the [appconduits Helm chart](https://github.com/bitsofinfo/appconduits). The primary intent of the `appdeploy` chart is simple to deploy a new version of an app; validate it, notify the team, and provide a **version specific** `Ingress` `host` binding to access it via an Ingress Controller. If you want to route traffic to your app via other `hosts:` or sophisticated routing rules, you should be creating separate/custom `Service` and `Ingress` combinations; again see [appconduits](https://github.com/bitsofinfo/appconduits).

---

When using all available options, each invocation of `appdeploy` can produce a single point of management Helm `release` that is comprised all of the components described above. No matter how many different applications your team manages, if you follow some simple conventions as well externalizing the bulk of application specific configuration, secured via a unique (and limited life) `bootstrapSecret`; you can really begin to see the economies of scale for a unified DevOps deployment approach no matter what the artifact.

<a id="diag"></a>![Diagram of appdeploy](/docs/diag.png "Diagram1")


## <a id="alerts"></a>Post install/upgrade/delete checks and alerts

The checking and alerting engine leverages: https://github.com/bitsofinfo/kubernetes-helm-healthcheck-hook

## <a id="options"></a>Configurable options

All configurable options are documented in [values.yaml](values.yaml)

See the [helm docs for setting values](https://github.com/helm/helm/blob/master/docs/chart_best_practices/values.md)
on how to customize/change these values when doing a `helm ...` invocation.

## <a id="metavar"></a>tpl variable substitution

Certain chart `values` can themselves be mini `templates` i.e. (`chartValue: "{{ .Values.someOtherValueRef }}`) (parsed via [Helm's tpl function](https://github.com/helm/helm/blob/master/docs/charts_tips_and_tricks.md) function). This can be useful when specifying `env` vars or `command.args` values and most labels etc. See [values.yaml](values.yaml) for full details on what values are parsed by `tpl`.

Currently the following are parsed:

* Any `.Values.env.[name].value`
* Any `.Values.command.args`
* Any `service.labels.[name].value`
* Any `pod.labels.[name].value`
* Any `ingress.metadata.labels.[name].value`
* Any `ingress.metadata.annotations.[name].value`
* Any `hooks.custom.[hookname].pod.labels.[name].value`
* Any `hooks.custom.[hookname].env.[name].value`
* Any `hooks.custom.[hookname].command.args`

You can reference any valid path relative from `.` (i.e. `$`). Some custom generated
variables that are composed and not literally in `values.yaml`:
* `.fullAppIdentifier_full` = `[app.shortName]-[app.context]-[image.tag][-[app.classifier]]`
* `.fullAppIdentifier_short` = a shorter version of `.fullAppIdentifier_full` if > 63 chars
* `.fullAppIdentifier_hash` = hash of `.fullAppIdentifier_full`
* `.fullAppIdentifier` = one of the latter values, whichever is < 63 chars in decending order
* `.bootstrapSecretFilePath` = `.Values.bootstrapSecret.mount.path/.Values.bootstrapSecret.mount.fileName`
* `.serviceAccountName` = `.Values.serviceAccount.name` (which is a `tpl` parsed value)

Basic example of what you can declare in your `values`:
```
...
env:
  MY_VAR_NAME:
    value: "{{ .Values.app.name | replace 'dog' 'cat' }}"
```

## Examples

For examples see the [examples/ folder](examples/)

## <a id="hooks"></a>Custom Helm Hooks

The chart also permits declaring a variable number of custom [Helm Hooks](https://github.com/helm/helm/blob/master/docs/charts_hooks.md) within your `values` by declaring them under `hooks.custom.[yourhookname]`. This functionality
is intended to be used for doing custom things w/ your deployment such as running migrations. All created
objects are labeled and named like any of the other objects created by this chart.

For further information see `hooks` in [values.yaml](values.yaml)

There is also a working example in the [examples/ folder](examples/)

## <a id="dependency"></a>As a dependency in another chart

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

# Specify value customizations for the 'appdeploy' dependency
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

## <a id="pack"></a>Helm package/update

```
helm package . -d repo/charts/
helm repo index repo/
```

## Using!

```
helm repo add bitsofinfo-appdeploy https://raw.githubusercontent.com/bitsofinfo/appdeploy/master/repo
helm repo update
```

```
# requirements.yaml
dependencies:
- name: appdeploy
  version: "1.1.5"
  repository: "https://raw.githubusercontent.com/bitsofinfo/appdeploy/master/repo"
```

# <a id="related"></a>Related Projects

* [appconduits](https://github.com/bitsofinfo/appconduits)
* [helmfile-deploy](https://github.com/bitsofinfo/helmfile-deploy)
