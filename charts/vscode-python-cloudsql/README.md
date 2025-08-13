# vscode-python-cloudsql

![Version: 0.0.2](https://img.shields.io/badge/Version-0.0.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

Minimal VS Code med Python og mulighet til å koble til Cloud SQL. Brukeren kan selv installere pakker etter behov.

**Homepage:** <https://manual.dapla.ssb.no/statistikkere/vscode-python.html>

## Source Code

* <https://github.com/statisticsnorway/dapla-lab-images>
* <https://github.com/statisticsnorway/dapla-lab-helm-charts-library>

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://statisticsnorway.github.io/dapla-lab-helm-charts-library | library-chart | 4.4.8 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` |  |
| autoscaling.enabled | bool | `false` |  |
| autoscaling.maxReplicas | int | `100` |  |
| autoscaling.minReplicas | int | `1` |  |
| autoscaling.targetCPUUtilizationPercentage | int | `80` |  |
| avansert.data.mountStandard | bool | `true` |  |
| avansert.startupScript.scriptArgs | string | `""` |  |
| avansert.startupScript.scriptPath | string | `""` |  |
| dapla.group | string | `"dapla-felles-developers"` |  |
| dapla.sharedBuckets | list | `[]` |  |
| dapla.sourceData.reason | string | `""` |  |
| dapla.sourceData.requestedDuration | string | `"4h"` |  |
| daplaUser | string | `""` |  |
| database.cloudSqlProxy.enabled | bool | `false` |  |
| database.cloudSqlProxy.instance | string | `""` |  |
| deleteJob.clusterRoleName | string | `"onyxia-delete-job"` |  |
| deleteJob.cronHourAtDay | string | `"20"` |  |
| deleteJob.cronMinuteAtDay | string | `"0"` |  |
| deleteJob.enabled | bool | `true` |  |
| deleteJob.imageVersion | string | `"v1.1.0"` |  |
| deleteJob.serviceAccount.annotations | object | `{}` |  |
| deployEnvironment | string | `"DEV"` |  |
| diskplass.accessMode | string | `"ReadWriteOnce"` |  |
| diskplass.enabled | bool | `true` |  |
| diskplass.size | string | `"10Gi"` |  |
| environment.group | string | `"users"` |  |
| environment.user | string | `"onyxia"` |  |
| fullnameOverride | string | `""` |  |
| gitConfig.git.cache | string | `""` |  |
| gitConfig.git.configMapName | string | `""` |  |
| gitConfig.git.email | string | `""` |  |
| gitConfig.git.enabled | bool | `false` |  |
| gitConfig.git.name | string | `""` |  |
| gitConfig.github.branch | string | `""` |  |
| gitConfig.github.build | bool | `false` |  |
| gitConfig.github.repository | string | `""` |  |
| gitConfig.github.token | string | `""` |  |
| global.suspend | bool | `false` |  |
| imagePullSecrets | list | `[]` |  |
| init.regionInit | string | `""` |  |
| init.regionInitCheckSum | string | `""` |  |
| init.standardInitPath | string | `"/opt/onyxia-init.sh"` |  |
| istio.enabled | bool | `false` |  |
| istio.gateways[0] | string | `"istio-namespace/example-gateway"` |  |
| istio.hostname | string | `"chart-example.local"` |  |
| kubernetes.enabled | bool | `true` |  |
| maskinportenGuardianUrl | string | `""` |  |
| nameOverride | string | `""` |  |
| networking.clusterIP | string | `"None"` |  |
| networking.service.port | int | `8080` |  |
| networking.type | string | `"ClusterIP"` |  |
| nodeSelector | object | `{}` |  |
| oidc.enabled | bool | `true` |  |
| oidc.secretName | string | `""` |  |
| oidc.tokenExchangeUrl | string | `""` |  |
| podAnnotations | object | `{}` |  |
| podDisruptionBudget.enabled | bool | `true` |  |
| podSecurityContext.fsGroup | int | `100` |  |
| pseudoServiceUrl | string | `""` |  |
| replicaCount | int | `1` |  |
| repository.condaRepository | string | `""` |  |
| repository.configMapName | string | `""` |  |
| repository.pipRepository | string | `""` |  |
| resources.cpu | string | `""` |  |
| resources.memory | string | `""` |  |
| security.allowlist.enabled | bool | `false` |  |
| security.allowlist.ip | string | `"0.0.0.0/0"` |  |
| security.networkPolicy.enabled | bool | `false` |  |
| security.networkPolicy.from | list | `[]` |  |
| security.oauth2.authenticatedEmails | string | `""` |  |
| security.oauth2.clientId | string | `"my-client"` |  |
| security.oauth2.oidcIssuerUrl | string | `"overwritten-by-onyxia"` |  |
| security.oauth2.provider | string | `"keycloak-oidc"` |  |
| security.serviceEntry.enabled | bool | `true` |  |
| security.serviceEntry.hosts[0] | string | `"storage.googleapis.com"` |  |
| securityContext | object | `{}` |  |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` |  |
| serviceAccount.name | string | `""` |  |
| statbankBaseUrl | string | `""` |  |
| statbankEncryptUrl | string | `""` |  |
| statbankTestBaseUrl | string | `""` |  |
| statbankTestEncryptUrl | string | `""` |  |
| suvDaplaApiUrl | string | `""` |  |
| tjeneste.image.pullPolicy | string | `"IfNotPresent"` |  |
| tjeneste.version | string | `"py311-2025.06.15T17_10Z"` |  |
| tolerations | list | `[]` |  |
| userAttributes.environmentVariableName | string | `"OIDC_TOKEN"` |  |
| userAttributes.userAttribute | string | `"access_token"` |  |
| userAttributes.value | string | `""` |  |
| userPreferences | object | `{}` |  |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.11.0](https://github.com/norwoodj/helm-docs/releases/v1.11.0)
