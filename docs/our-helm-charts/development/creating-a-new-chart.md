# Creating a new chart

## Dependencies

If you would like to help create new charts using the common library, there's
a few tools you will need.

- [helm](https://helm.sh/docs/intro/install/)
- [Task](https://taskfile.dev) (optional)
- [pre-commmit](https://pre-commit.com) (optional - required with tasks)

## Prerequisites

If you wish to use the `task` commands to create a new chart from the template,
you need to make sure that [Task](https://taskfile.dev) is installed.

Please see the [project docs](https://taskfile.dev/#/installation) for
more information on how to install it.

## Creating a new chart

To create a new chart, run the following:

```sh
git clone https://github.com/k8s-at-home/charts.git
cd charts

task deps:install
task chart:create CHART_TYPE=stable CHART=chart-name
```

Second, be sure to checkout the many charts that already use the common library like
[qBittorrent](https://github.com/k8s-at-home/charts/tree/master/charts/stable/qbittorrent),
[node-red](https://github.com/k8s-at-home/charts/tree/master/charts/stable/node-red)
or the many others in this repository.

You can recognize which charts include this common chart as dependency by the following snippet:

```yaml
# Chart.yaml
...
dependencies:
- name: common
  version: 4.3.0 # make sure to use the latest common library version available
  repository: https://library-charts.k8s-at-home.com
...
```

### Chart metadata

In order to keep our chart documentation similar across all charts we rely heavily on template files.
We use [helm-docs](https://github.com/k8s-at-home/helm-docs) to render these templates into the `README.md` file.

Don't forgot to populate/update the chart information in `charts/stable/chart-name/Chart.yaml` and
`charts/stable/chart-name/values.yaml` files.

Aside from the default chart metadata (such as the `version`, `appVersion`, etc fields), we require a
`artifacthub.io/changes` chart annotation
(as documented by [ArtifactHUB](https://artifacthub.io/docs/topics/annotations/helm/))
that describes the modifications in this chart version.

```yaml
# Chart.yaml
...
annotations:
  artifacthub.io/changes: |
    - kind: added
      description: Initial version
...
```

!!! info
    Because we rely so heavily on standardized templates, any manual changes to `README.md` will get overwriten
    when the documentation is generated.

    Any chart-specific documentation should go in the `README_CONFIG.md.gotmpl` file.

## Testing

When testing locally, make sure you update the dependencies from within the chart
directory:

```bash
helm dependency update
```

If making local changes to the `common` library, the test chart may reference
the local development chart:

```yaml
# common-test/Chart.yaml
...
dependencies:
- name: common
  version: <new version>
  repository: file://.../common
...
```

Be sure to lint your chart to check for any errors.

```sh
# Linting
task chart:lint CHART_TYPE=stable CHART=chart-name
task chart:ct-lint CHART_TYPE=stable CHART=chart-name
```

## Values

Edit `values.yaml` with some basic defaults you want to present to the user.
Please ensure you anotate them with [helm-docs](https://github.com/k8s-at-home/helm-docs)
e.g.

```yaml
#
# IMPORTANT NOTE
#
# This chart inherits from our common library chart. You can check the default values/options here:
# https://github.com/k8s-at-home/library-charts/tree/master/charts/stable/common/values.yaml
#

image:
  # -- image repository
  repository: nodered/node-red
  # -- image tag
  tag: 1.2.5
  # -- image pull policy
  pullPolicy: IfNotPresent

# -- environment variables.
# @default -- See below
env:
  # -- Set the container timezone
  TZ: UTC

# -- Configures service settings for the chart.
# @default -- See values.yaml
service:
  main:
    ports:
      http:
        port: 1880

ingress:
  # -- Enable and configure ingress settings for the chart under this key.
  # @default -- See values.yaml
  main:
    enabled: false

persistence:
  data:
    enabled: false
    type: emptyDir
    mountPath: /data
```

If not using a service, set the `service.enabled` to `false`.

```yaml
...
service:
  main:
    enabled: false
...
```

## Templates

### Basic

In its most basic form a new chart can consist of two simple files in the
`templates` folder. This will automatically render everything, based only on
what is (or isn't) present in `values.yaml`.

**`templates/common.yaml`**:

```yaml
{{ include "common.all . }}
```

**`templates/NOTES.txt`**:

```yaml
{{ include "common.notes.defaultNotes" . }}
```

### Advanced

Sometimes it is not required to implement additional logic in a chart that you
do not wish to expose through settings in `values.yaml`. For example, when you
want to always mount a Secret or configMap as a volume in the Pod. In that
case it is also possible to write more advanced template files.

**`templates/common.yaml`**:

```yaml
{{/* First Make sure all variables are set and merged properly */}}
{{- include "common.values.setup" . }}

{{/* Append the configMap volume to the volumes */}}
{{- define "myapp.settingsVolume" -}}
enabled: "true"
mountPath: "/app/configuration.yaml"
subPath: "configuration.yaml"
type: "custom"
volumeSpec:
  configMap:
    name: {{ include "common.names.fullname" . }}-settings
{{- end -}}
{{- $_ := set .Values.persistence "myapp-settings" (include "myapp.settingsVolume" . | fromYaml) -}}

{{/* Render the templates */}}
{{ include "common.all" . }}
```

An actual example of this can be found in the [zigbee2mqtt][zigbee2mqtt] chart.

[zigbee2mqtt]: https://github.com/k8s-at-home/charts/tree/master/charts/stable/zigbee2mqtt
