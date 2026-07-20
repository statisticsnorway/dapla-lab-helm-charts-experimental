# prodcom

![Version: 0.2.2](https://img.shields.io/badge/Version-0.2.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

Prodcom - SSBs saksbehandlingsløsning for industriell vareproduksjon (Plotly Dash-app).

**Homepage:** <https://github.com/statisticsnorway/stat-prodcom>

## Source Code

* <https://github.com/statisticsnorway/stat-prodcom>
* <https://github.com/statisticsnorway/dapla-lab-helm-charts-experimental>

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://statisticsnorway.github.io/dapla-lab-helm-charts-library | library-chart | 4.6.1 |

## Hvordan tjenesten fungerer

Dette er en **app-image-tjeneste** (samme mønster som `datadoc`): charten kjører et
ferdigbygget Docker-image direkte - den kloner *ikke* koden og installerer *ikke*
avhengigheter ved oppstart.

```
Bruker ─▶ Istio VirtualService ("/") ─▶ Service:4180 ─▶ oauth2-proxy (sidecar)
                                                              │  autentisering (Keycloak)
                                                              ▼
                                                   Dash-appen på 127.0.0.1:8061  (servert på "/")
```

- oauth2-proxy-sidecaren lytter på `4180` og videresender til appen på
  `networking.service.port` (8061).
- Appen serveres på rot-stien `/` (ikke bak jupyter-server-proxy). Dette styres av
  miljøvariabelen `PRODCOM_STANDALONE=true`, som charten setter.
- Datatilgang til Google-bøtter gis via Dapla ssbucketeer (annotasjoner på StatefulSet-en)
  ved å velge team/tilgangsgruppe under *Data*.
- **Prod-data leses kun i produksjonsmiljøet**: appen laster data fra produksjonsbøtta
  bare når `DAPLA_ENVIRONMENT=PROD` (satt av charten fra `deployEnvironment`). I
  test-clusteret (`TEST`) starter appen uten data - ingen prod-data i test.
- Tjenesten slettes automatisk hver kveld (deleteJob, kl. 20:00 UTC = kl. 21/22 norsk
  tid), som andre brukertjenester i Dapla Lab.

## Forutsetninger (må være på plass før tjenesten fungerer)

1. **Appbildet må bygges.** Workflowen *Build Prodcom service image* i `stat-prodcom`
   (`.github/workflows/build-service-image.yaml`) bygger og publiserer imaget til
   `europe-north1-docker.pkg.dev/artifact-registry-5n/strukt-naering-docker/prodcom`
   via Workload Identity Federation - eierteamets eget registry, samme modell som
   Datadoc-editor. Den pusher en uforanderlig `<dato>-<sha>`-tag og en flytende
   `latest`. *Engangsforutsetning*: `stat-prodcom` må stå i `artifact_registry.repos`
   i `teams/strukt-naering/team.yaml` (statisticsnorway/terraform-ssb-dapla-teams) -
   samme mekanisme som datadoc-repoene i `teams/dapla-metadata`.

2. **Pull-tilgang fra Dapla Lab (engangs plattform-oppgave).** Clusterets node-SA må
   ha `artifactregistry.reader` på `strukt-naering-docker`: legg repoet til i
   `gcp_gar_repos` i `dapla-lab-iac` (prod + dev) - nøyaktig slik
   `dapla-metadata-docker` er lagt inn for datadoc. Kyverno-policyen
   `restrict-image-registries` tillater `europe-*-docker.pkg.dev/artifact-registry-5n/*`,
   så charten trenger ingen `service-catalog`-annotasjon eller policy-unntak.

3. **`dash/app.py`-endringen er på plass** (merget i `stat-prodcom` PR #186): appen
   serveres på rot-stien og kjører headless når `PRODCOM_STANDALONE=true`, og prod-data
   lastes kun når `DAPLA_ENVIRONMENT=PROD`. Eksisterende bruk i JupyterLab er uendret.

4. **Datatilgang (kun produksjon).** I produksjonsmiljøet må team/tilgangsgruppen valgt
   under *Data* ha lese/skrive-tilgang til `gs://ssb-strukt-naering-data-produkt-prod`
   (appen leser bl.a. `vti/temp/omsetning_422/...` og `vti/inndata/nspek/...` ved
   oppstart). I test-clusteret lastes ingen data, og tjenesten starter uavhengig av
   bøttetilgang.

## Image-kontrakt

Imaget som `tjeneste.image.repository:tjeneste.image.version` peker på må:

- Serve Prodcom Dash-appen over HTTP på `$PORT` (default `8061`), på rot-stien `/`.
- Respektere `PRODCOM_STANDALONE=true` (serve på `/`, kjør headless på `0.0.0.0:$PORT`).
- Inneholde Python-avhengighetene appen faktisk bruker, inkludert `rpy2` med **R** og
  den SSB-interne R-pakken **Kostra** (HB-metoden kjører `importr("Kostra")`).
  Merk: appen importerer *ikke* sentence-transformers/catboost/scikit-learn/cx-oracle -
  disse brukes kun av pipelines utenfor `dash/` og er utelatt fra imaget.
- Kjøre som uid `1000` med primærgruppe gid `100` (`users`): charten setter
  `fsGroup: 100`, så volumet montert på `/home/onyxia/work` er gruppeeid av gid 100.
- Ha en skrivbar `HOME` (`/home/onyxia`); charten peker `XDG_CACHE_HOME` til
  `/home/onyxia/work/.cache` på det monterte volumet.

## Lokal validering

```bash
helm repo add dapla-lab-library https://statisticsnorway.github.io/dapla-lab-helm-charts-library
helm dependency build charts/prodcom
helm lint charts/prodcom
helm template prodcom charts/prodcom \
  --set istio.enabled=true \
  --set istio.hostname=prodcom.example.no \
  --set dapla.group=<ditt-team>-developers
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| avansert.data.mountStandard | bool | `true` |  |
| dapla.group | string | `""` |  |
| dapla.sharedBuckets | list | `[]` |  |
| dapla.sourceData.reason | string | `""` |  |
| dapla.sourceData.requestedDuration | string | `"4h"` |  |
| daplaUser | string | `""` |  |
| deleteJob.clusterRoleName | string | `"onyxia-delete-job"` |  |
| deleteJob.cronHourAtDay | string | `"20"` |  |
| deleteJob.cronMinuteAtDay | string | `"0"` |  |
| deleteJob.enabled | bool | `true` |  |
| deleteJob.imageVersion | string | `"v1.1.0"` |  |
| deleteJob.serviceAccount.annotations | object | `{}` |  |
| deployEnvironment | string | `"TEST"` |  |
| diskplass.accessMode | string | `"ReadWriteOnce"` |  |
| diskplass.enabled | bool | `false` |  |
| diskplass.size | string | `"10Gi"` |  |
| environment.group | string | `"users"` |  |
| environment.user | string | `"onyxia"` |  |
| fullnameOverride | string | `""` |  |
| global.suspend | bool | `false` |  |
| imagePullSecrets | list | `[]` |  |
| istio.enabled | bool | `false` |  |
| istio.gateways[0] | string | `"istio-namespace/example-gateway"` |  |
| istio.hostname | string | `"chart-example.local"` |  |
| kubernetes.enabled | bool | `false` |  |
| kubernetes.role | string | `"view"` |  |
| nameOverride | string | `""` |  |
| networking.clusterIP | string | `"None"` |  |
| networking.service.port | int | `8061` |  |
| networking.type | string | `"ClusterIP"` |  |
| nodeSelector | object | `{}` |  |
| oidc.enabled | bool | `true` |  |
| oidc.secretName | string | `""` |  |
| oidc.tokenExchangeUrl | string | `""` |  |
| podAnnotations | object | `{}` |  |
| podDisruptionBudget.enabled | bool | `true` |  |
| podLabels."onyxia.app" | string | `"prodcom"` |  |
| podLabels.maintained-by-team | string | `"strukt-naering"` |  |
| podSecurityContext.fsGroup | int | `100` |  |
| replicaCount | int | `1` |  |
| resources.cpu | string | `"500m"` | Garantert CPU (requests; 1000m = one core) |
| resources.memory | string | `"4Gi"` | Garantert minne (requests; also used as the memory limit) |
| security.networkPolicy.enabled | bool | `false` |  |
| security.networkPolicy.from | list | `[]` |  |
| security.oauth2.authenticatedEmails | string | `""` |  |
| security.oauth2.clientId | string | `"onyxia-services"` |  |
| security.oauth2.oidcIssuerUrl | string | `"overwritten-by-onyxia"` |  |
| security.oauth2.provider | string | `"keycloak-oidc"` |  |
| security.serviceEntry.enabled | bool | `true` |  |
| security.serviceEntry.hosts[0] | string | `"www.googleapis.com"` |  |
| security.serviceEntry.hosts[1] | string | `"oauth2.googleapis.com"` |  |
| security.serviceEntry.hosts[2] | string | `"storage.googleapis.com"` |  |
| security.serviceEntry.hosts[3] | string | `"sts.googleapis.com"` |  |
| security.serviceEntry.hosts[4] | string | `"auth.test.ssb.no"` |  |
| security.serviceEntry.hosts[5] | string | `"data.ssb.no"` |  |
| security.serviceEntry.hosts[6] | string | `"www.ssb.no"` |  |
| securityContext | object | `{}` |  |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` |  |
| serviceAccount.name | string | `""` |  |
| startupProbe.failureThreshold | int | `60` |  |
| startupProbe.initialDelaySeconds | int | `10` |  |
| startupProbe.periodSeconds | int | `10` |  |
| startupProbe.successThreshold | int | `1` |  |
| startupProbe.timeoutSeconds | int | `30` |  |
| tjeneste.image.pullPolicy | string | `"Always"` | Image pull policy |
| tjeneste.image.repository | string | `"europe-north1-docker.pkg.dev/artifact-registry-5n/strukt-naering-docker/prodcom"` | Container repository (without tag) for the prebuilt Prodcom app image. Built and published to strukt-naering's own registry (the team owning the Prodcom data product) by stat-prodcom's "Build Prodcom service image" workflow - WIF push, same model as Datadoc-editor. See README.md -> "Image-kontrakt". |
| tjeneste.image.version | string | `"latest"` | Image tag to deploy: "latest" (moving alias) or an immutable <date>-<sha> tag from the build workflow's Step Summary. |
| tolerations | list | `[]` |  |
| userAttributes.environmentVariableName | string | `"OIDC_TOKEN"` |  |
| userAttributes.userAttribute | string | `"access_token"` |  |
| userAttributes.value | string | `""` |  |
