# jupyter-pyspark

![Version: 0.0.5](https://img.shields.io/badge/Version-0.0.5-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

The JupyterLab IDE with Python and Spark, tailored for SSB's data platform (Dapla).

**Homepage:** <https://jupyter.org/>

## Source Code

* <https://github.com/statisticsnorway/dapla-lab-helm-charts-services>

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://statisticsnorway.github.io/dapla-lab-helm-charts-services | library-chart | 3.2.4 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` |  |
| autoscaling.enabled | bool | `false` |  |
| autoscaling.maxReplicas | int | `100` |  |
| autoscaling.minReplicas | int | `1` |  |
| autoscaling.targetCPUUtilizationPercentage | int | `80` |  |
| dapla.buckets.enabled | bool | `false` |  |
| dapla.buckets.mountStandard | bool | `true` |  |
| dapla.group | string | `""` |  |
| deleteJob.clusterRoleName | string | `"onyxia-delete-job"` |  |
| deleteJob.cronHourAtDay | string | `"20"` |  |
| deleteJob.cronMinuteAtDay | string | `"0"` |  |
| deleteJob.enabled | bool | `true` |  |
| deleteJob.imageVersion | string | `"v1.1.0"` |  |
| deleteJob.serviceAccount.annotations | object | `{}` |  |
| deployEnvironment | string | `"DEV"` |  |
| discovery.mlflow | bool | `true` |  |
| environment.group | string | `"users"` |  |
| environment.user | string | `"onyxia"` |  |
| fullnameOverride | string | `""` |  |
| git.branch | string | `""` |  |
| git.cache | string | `""` |  |
| git.configMapName | string | `""` |  |
| git.email | string | `""` |  |
| git.enabled | bool | `false` |  |
| git.name | string | `""` |  |
| git.repository | string | `""` |  |
| git.token | string | `""` |  |
| global.suspend | bool | `false` |  |
| imagePullSecrets | list | `[]` |  |
| init.personalInit | string | `""` |  |
| init.personalInitArgs | string | `""` |  |
| init.regionInit | string | `""` |  |
| init.standardInitPath | string | `"/opt/onyxia-init.sh"` |  |
| istio.enabled | bool | `false` |  |
| istio.gateways[0] | string | `"istio-namespace/example-gateway"` |  |
| istio.hostname | string | `"chart-example.local"` |  |
| kubernetes.enabled | bool | `false` |  |
| kubernetes.role | string | `"view"` |  |
| mlflow.configMapName | string | `""` |  |
| nameOverride | string | `""` |  |
| networking.clusterIP | string | `"None"` |  |
| networking.service.port | int | `8888` |  |
| networking.sparkui.port | int | `4040` |  |
| networking.type | string | `"ClusterIP"` |  |
| nodeSelector | object | `{}` |  |
| oidc.configMapName | string | `""` |  |
| oidc.enabled | bool | `true` |  |
| oidc.tokenExchangeUrl | string | `""` |  |
| persistence.accessMode | string | `"ReadWriteOnce"` |  |
| persistence.enabled | bool | `true` |  |
| persistence.size | string | `"10Gi"` |  |
| podAnnotations | object | `{}` |  |
| podLabels."onyxia.app" | string | `"jupyterlab"` |  |
| podSecurityContext.fsGroup | int | `100` |  |
| replicaCount | int | `1` |  |
| repository.condaRepository | string | `""` |  |
| repository.configMapName | string | `""` |  |
| repository.pipRepository | string | `""` |  |
| resources.requests.cpu | string | `""` |  |
| resources.requests.memory | string | `""` |  |
| security.allowlist.enabled | bool | `false` |  |
| security.allowlist.ip | string | `"0.0.0.0/0"` |  |
| security.networkPolicy.enabled | bool | `false` |  |
| security.networkPolicy.from | list | `[]` |  |
| security.oauth2.authenticatedEmails | string | `""` |  |
| security.oauth2.clientId | string | `"my-client"` |  |
| security.oauth2.oidcIssuerUrl | string | `""` |  |
| security.oauth2.provider | string | `"keycloak-oidc"` |  |
| security.password | string | `"changeme"` |  |
| security.serviceEntry.enabled | bool | `true` |  |
| security.serviceEntry.hosts[0] | string | `"storage.googleapis.com"` |  |
| securityContext | object | `{}` |  |
| service.image.pullPolicy | string | `"IfNotPresent"` |  |
| service.image.version | string | `"europe-north1-docker.pkg.dev/artifact-registry-5n/dapla-lab-docker/onyxia/jupyter-pyspark:py311-spark3.5.3"` |  |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` |  |
| serviceAccount.name | string | `""` |  |
| startupProbe.failureThreshold | int | `60` |  |
| startupProbe.initialDelaySeconds | int | `10` |  |
| startupProbe.periodSeconds | int | `10` |  |
| startupProbe.successThreshold | int | `1` |  |
| startupProbe.timeoutSeconds | int | `5` |  |
| tolerations | list | `[]` |  |
| userAttributes.environmentVariableName | string | `"OIDC_TOKEN"` |  |
| userAttributes.userAttribute | string | `"access_token"` |  |
| userAttributes.value | string | `""` |  |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.11.0](https://github.com/norwoodj/helm-docs/releases/v1.11.0)
