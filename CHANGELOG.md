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
