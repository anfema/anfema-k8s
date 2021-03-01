# Creating a Helm Chart

A Chart has the following directory structure:

```
my-chart/
  Chart.yaml          # Chart metadata
  LICENSE             # optional
  README.md           # optional
  values.yaml         # The default configuration values for this chart
  values.schema.json  # optional: A JSON Schema for the values.yaml file
  charts/             # dependencies
  crds/               # Custom Resource Definitions
  templates/          # Custom Templates to be used
  templates/NOTES.txt # optional: Template documentation
```

To create a new chart run `helm create <chart-name>`, this will create the
directory structure with all the needed files.

You can use `helm lint <chart-name>` to run a linter over the chart to tell you
any errors and finally `helm package <chart-name>` to create a chart archive to
distribute.

## Chart.yaml

Required fields:

```yaml
apiVersion: "v2"
name: MyChart
version: "1.0.0"
```

Keep `apiVersion` at `v2` if using helm 3 (you should!). The `version` is a
semver field which defines the version of the chart, not the contained software.
If you want to version the software contained in the chart use the optional
field `appVersion` (which does not have to be a SemVer version even).
If you want to re-deploy/update a chart you have to change the chart's version
field, else helm will not pick up the updates.

Optional fields:

```yaml
kubeVersion: ">=1.15.0"
description: This is my example chart
type: application
appVersion: "1.0.0"
deprecated: false
keywords:
  - keyword
  - another
home: https://example.com/myproject
sources:
  - https://example.com/myproject.git
dependencies:
  - name: nginx
    version: "1.2.3"
    repository: https://example.com/charts
    condition: nginx.enabled
    tags:
      - nginx
    import-values:
      - data
    alias: nginx-1-2-3
maintainers:
  - name: Someone
    email: someone@example.com
icon: http://example/com/myproject/icon.png
annotations:
  example: A list of annotations keyed by name (optional).
```

Description of the non-obvious fields:
- `type` can be `application` or `library`, use library charts to externalize
  commonly used templates
- `deprecated` set to `true` and update the chart version to enable chart
  deprecation
- `home` just an URL to the project homepage
- `sources` URLs to the sourcecode of the project
- `dependencies.condition`, only include this dependency if the condition
  variable is `true` in `values.yaml` (or external overrides of that file)
- `dependencies.repository` can be an URL or a `@alias` name created by adding
  the repository on the command line
- `dependencies.import-values`, import a path from the dependencies'
  `values.yaml` into this chart's values.
- `dependencies.alias` if you want to include a dependency multiple times (e.g.
  with a condition), you can use an alias to reference a specific dependency.

You can use the `dependencies` field or put dependent charts into the `charts`
sub-directory. Both work the same.

## values.yaml

In `values.yaml` you will find all default settings for the chart and
overridden settings for all dependency charts.

```yaml
a-variable-in-this-chart: 1

<dependency>:
    an-overridden-variable: 2
```

replace `<dependency>` with the name of the dependency (or the `alias` if you
want to specify a specific instance)

## What goes into the template folder

See [Required Templates in a Chart](Required_Templates_in_a_Chart.md) for
examples.