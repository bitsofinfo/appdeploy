# 1.1.13
* Added `tpl` support for `image.repository` and `image.tag`
  
# 1.1.12
* Added `extraService.annotations` support w/ `tpl` support

# 1.1.11
* Added `pod.hostname` and `pod.subdomain` with `tpl` substitution support
* Added `extraService.*` section to permit configuration of an secondary `Service` with completely custom `selectors, name, labels, type` etc

# 1.1.10
* Upgraded to `kubernetes-helm-healthcheck:0.1.19`
* + `hooks.default.[postInstallUpgrade|postDelete].validator.debugRequests` that corresponds
to `kubernetes-helm-healthcheck:0.1.19` new `--verbose-debug-requests` argument
* Removed all default `resources.[limits|requests]` settings in `values.yaml`

# 1.1.9
* **BREAKING CHANGE:** Changed `ingress.metadata.annotations` and `ingress.metadata.labels` from a list to key-value pairs

# 1.1.8
* Fix `postDelete` and `postInstallUpgrade` http(s) schemes if `useIngressHost: true`

# 1.1.7
* Added `bootstrapSecret.mount.defaultMode`
* Added `securityContext.fsGroup`

# 1.1.6
* Added `ingress.dns.fqdnSuffixPrefix`

# 1.1.5
* Remove webhook from default values, put in example only

# 1.1.4
* Fix #8

# 1.1.3
* Fix #7

# 1.1.2
* Fix #6

# 1.1.1
* Support for `hooks.custom.[hookname]` Helm hook Jobs
* Refactoring of secrets and Pod template to a golang template

# 1.1.0
* Breaking change release
* `meta-variables` (i.e. `[[#varname]]`) are gone and just replaced with Helm `tpl` function calls where `values` (on certain keys) are just parsed as real templates, the entire `[[#varname]]` syntax is gone
* Breaking changes in `values.yaml`
  * `hooks.postInstallUpgrade` is now `hooks.default.postInstallUpgrade.validator`
  * `hooks.postDelete` is now `hooks.default.postDelete.validator`
* Added `app.shortName` for `app.name` that are too long. (defaults to `{{.Values.app.name}}`)
* Added a hashId of fullAppIdentifier to all generated objects as label
* General cleanup and refactoring

# 1.0.2
* Generate `Ingress` for each alias in `aliases`

# 1.0.1
* + `useIngressHost` and `enabled` on `hooks`

# 1.0.0

Initial release
