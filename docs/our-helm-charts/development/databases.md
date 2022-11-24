# Databases

Databases from other repositories can be added to charts as dependencies. The
databases are only installed if the `<db name>.enabled` key is set to true.

See [home-assistant][home-assistant] for reference.

## Chart.yaml

Add the following entries to `Chart.yaml` under the `dependencies` section.

```yaml
...
dependencies:
- name: postgresql
  version: <chart version>
  repository: https://charts.bitnami.com/bitnami
  condition: postgresql.enabled
- name: mariadb
  version: <chart version>
  repository: https://charts.bitnami.com/bitnami
  condition: mariadb.enabled
- name: influxdb
  version: <chart version>
  repository: https://charts.bitnami.com/bitnami
  condition: influxdb.enabled
```

## values.yaml

Update the `values.yaml` with the following. Refer the respective database
chart `values.yaml` for additional values.

### [MariaDB](https://github.com/bitnami/charts/tree/master/bitnami/mariadb)

```yaml
...
mariadb:
  enabled: false
  architecture: standalone
  auth:
    database: <chart name>
    username: <chart name>
    password: <chart password>
    rootPassword: home-assistantrootpass
  primary:
    persistence:
      enabled: false
```

### [Postgres](https://github.com/bitnami/charts/tree/master/bitnami/postgresql)

```yaml
...
postgresql:
  enabled: false
  postgresqlUsername: <chart name>
  postgresqlPassword: <chart password>
  postgresqlDatabase: <chart name>
  persistence:
    enabled: false
```

### [InfluxDB](https://github.com/bitnami/charts/tree/master/bitnami/influxdb)

```yaml
...
influxdb:
  enabled: false
  architecture: standalone
  database: <chart name>
  authEnabled: false
  persistence:
    enabled: false
```

[home-assistant]: https://github.com/k8s-at-spiti/charts/tree/master/charts/stable/home-assistant
