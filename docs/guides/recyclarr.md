# Introduction

This is a guide on how to use native k8s `CronJob`s to run Recyclarr against your
existing Sonarr and Radarr installations in two easy steps.

## Creating a `ConfigMap`

Step one is to create a `configmap` based on the [documentation](https://github.com/recyclarr/recyclarr/wiki/Configuration-Reference).

!!! Note
    This example is based on the default configuration. You will want to make changes
    based on the documentation above.

If you are referencing the example below, replace the API keys with the keys for your installation. You can find them under `Settings > General`.

You must also replace the base_url entries with urls that are reachable from the pod that the cronjob will create. This is actually pretty easy. If you are running Sonarr or Radarr inside the same cluster as Recyclarr, you should be able
to use the addresses in the examples by replacing `default` with the correct namespace. If you aren't sure of the exact naming, `kubectl get services -A` will list all of the services in the cluster. And don't forget the port!

```yaml
# configmap.yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: recyclarr-config
  namespace: default
  labels:
    app.kubernetes.io/name: recyclarr
    app.kubernetes.io/instance: recyclarr
data:
  recyclarr.yml: |
    sonarr:
      - base_url: http://sonarr.default.svc:8989
        api_key: f7e74ba6c80046e39e076a27af5a8444

        # Quality Definition Settings
        quality_definition: hybrid

        # Release Profile Settings
        release_profiles:
          - trash_ids:
              - d428eda85af1df8904b4bbe4fc2f537c # Anime - First release profile
              - 6cd9e10bb5bb4c63d2d7cd3279924c7b # Anime - Second release profile
            strict_negative_scores: true
            tags: [anime]
          - trash_ids:
              - EBC725268D687D588A20CBC5F97E538B # Low Quality Groups
              - 1B018E0C53EC825085DD911102E2CA36 # Release Sources (Streaming Service)
              - 71899E6C303A07AF0E4746EFF9873532 # P2P Groups + Repack/Proper
            strict_negative_scores: false
            tags: [tv]
          - trash_ids: [76e060895c5b8a765c310933da0a5357] # Optionals
            filter:
              include:
                - 436f5a7d08fbf02ba25cb5e5dfe98e55 # Ignore Dolby Vision without HDR10 fallback
                - f3f0f3691c6a1988d4a02963e69d11f2 # Ignore The Group -SCENE
            tags: [tv]
    radarr:
      - base_url: http://radarr.default.svc:7878
        api_key: bf99da49d0b0488ea34e4464aa63a0e5

        # Quality Definition Settings
        quality_definition:
          type: movie
          preferred_ratio: 0.5

        # Custom Format Settings
        delete_old_custom_formats: false
        custom_formats:
          - names:
              - BR-DISK
              - EVO (no WEB-DL)
              - LQ
              - x265 (720/1080p)
              - 3D
            quality_profiles:
              - name: HD-1080p
              - name: HD-720p2
                score: -1000
          - names:
              - TrueHD ATMOS
              - DTS X
            quality_profiles:
              - name: SD
```

## Creating the `CronJob`

The second and last step is to create a `CronJob`.

This example will run every 8 hours.

!!! Note
    This is intentionally set to run in `preview` mode. Remove the `,"-p"` from the args to apply changes to Radarr.

```yaml
# radarr-cronjob.yaml
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: recyclarr-radarr
  namespace: default
  labels:
    app.kubernetes.io/name: recyclarr
    app.kubernetes.io/instance: recyclarr-radarr
    app.kubernetes.io/version: "2.2"
spec:
  schedule: "0 */3 * * *" # every 8 hours
  concurrencyPolicy: Replace
  successfulJobsHistoryLimit: 21 # 1 week
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          volumes:
            - name: recyclarr-config
              configMap:
                name: recyclarr-config
          containers:
            - name: recyclarr
              image: ghcr.io/recyclarr/recyclarr:2.2
              command: ["recyclarr"]
              args: ["radarr", "-p"]
              volumeMounts:
                - mountPath: "/config/recyclarr.yml"
                  name: recyclarr-config
                  subPath: recyclarr.yml
```
