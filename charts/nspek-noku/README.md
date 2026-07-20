# nspek-noku

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

NSPEK/NØKU - SSBs editeringsløsning for strukturstatistikk for næringene (Plotly Dash-app).

**Homepage:** <https://github.com/statisticsnorway/stat-naringer-dash>

## Source Code

* <https://github.com/statisticsnorway/stat-naringer-dash>
* <https://github.com/statisticsnorway/dapla-lab-helm-charts-experimental>

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://statisticsnorway.github.io/dapla-lab-helm-charts-library | library-chart | 4.6.1 |

## Hvordan tjenesten fungerer

Dette er en **app-image-tjeneste** (samme mønster som `prodcom` og `datadoc`): charten
kjører et ferdigbygget Docker-image direkte - den kloner *ikke* koden og installerer
*ikke* avhengigheter ved oppstart. Appen er NSPEK/NØKU-editeringsappen fra
`stat-naringer-dash` (`src/nspek_noku/app_nspek_noku.py`).

```
Bruker ─▶ Istio VirtualService ("/") ─▶ Service:4180 ─▶ oauth2-proxy (sidecar)
                                                              │  autentisering (Keycloak)
                                                              ▼
                                                   Dash-appen på 127.0.0.1:8008  (servert på "/")
                                                              │
                                        Cloud SQL Auth Proxy (sidecar, localhost:5432)
                                                              ▼
                                              Cloud SQL Postgres (strukt-naering)
```

- oauth2-proxy-sidecaren lytter på `4180` og videresender til appen på
  `networking.service.port` (8008).
- Appen serveres på rot-stien `/` (ikke bak jupyter-server-proxy). Dette styres av
  miljøvariabelen `NSPEK_NOKU_STANDALONE=true`, som charten setter.
- **Appen krever Postgres ved oppstart**: tilkoblingspoolen åpnes ved import, mot
  `localhost:5432` via en Cloud SQL Auth Proxy-sidecar (`--auto-iam-authn`,
  IAM-databasebruker velges av appen ut fra `DAPLA_ENVIRONMENT`). Uten databasen
  starter ikke tjenesten - `database.cloudSqlProxy.instance` må settes.
- Datatilgang til Google-bøtter gis via Dapla ssbucketeer (annotasjoner på StatefulSet-en)
  ved å velge team/tilgangsgruppe under *Data*. Appen leser og skriver
  `/buckets/produkt/naringer/...` via de standardmonterte bøttene, og bruker
  `/buckets/shared/vof` (delt bøtte) til VoF-fanen når den er montert.
- Egress: charten har en egen ServiceEntry for rå TCP mot Cloud SQL-privat-IP-området
  (port 3307, samme blokk som vscode-python-cloudsql) - mesh-en kjører
  `outboundTrafficPolicy: REGISTRY_ONLY`, så uten den når ikke cloud-sql-proxy
  databasen. HTTPS-egress (GCS, KLASS, auth) styres av den delte
  `dapla/egress-datadoc.json`-skjemaet per miljø. Stilarkene fra cdn.jsdelivr.net
  hentes av brukerens nettleser, ikke av poden, og trenger ingen egress-åpning.
- Tjenesten slettes automatisk hver kveld (deleteJob, kl. 20:00 UTC = kl. 21/22 norsk
  tid), som andre brukertjenester i Dapla Lab.

## Forutsetninger (må være på plass før tjenesten fungerer)

1. **Appbildet må bygges.** Workflowen *Build nspek-noku service image* i
   `stat-naringer-dash` (`.github/workflows/build-service-image.yaml`) bygger og
   publiserer imaget til
   `europe-north1-docker.pkg.dev/artifact-registry-5n/strukt-naering-docker/nspek-noku`
   via Workload Identity Federation - eierteamets eget registry, samme modell som
   Prodcom-tjenesten. Den pusher en uforanderlig `<dato>-<sha>`-tag og en flytende
   `latest`. *Engangsforutsetning*: `stat-naringer-dash` må stå i
   `artifact_registry.repos` i `teams/strukt-naering/team.yaml`
   (statisticsnorway/terraform-ssb-dapla-teams) - samme mekanisme som `stat-prodcom`.

2. **Pull-tilgang fra Dapla Lab.** Clusterets node-SA må ha `artifactregistry.reader`
   på `strukt-naering-docker` i `gcp_gar_repos` i `dapla-lab-iac`. Denne er allerede
   på plass (lagt inn for Prodcom-tjenesten) og gjelder hele registryet, så nye
   image-navn under `strukt-naering-docker/` krever ingen ny endring. Kyverno-policyen
   `restrict-image-registries` tillater `europe-*-docker.pkg.dev/artifact-registry-5n/*`,
   så charten trenger ingen policy-unntak.

3. **Standalone-endringen i appen er på plass**: appen serveres på rot-stien og kjører
   headless når `NSPEK_NOKU_STANDALONE=true` (se stat-naringer-dash-patchen).
   Eksisterende bruk i JupyterLab er uendret.

4. **Databasetilgang.** `database.cloudSqlProxy.instance` må settes til
   instance connection name for `strukt-naering`-databasen
   (`<prosjekt>:<region>:<instans>`), og teamet/tilgangsgruppen må ha IAM-tilgang til
   Cloud SQL-instansen. Appen velger IAM-databasebruker ut fra `DAPLA_ENVIRONMENT`
   (PROD -> prod-gruppebrukeren, ellers test-gruppebrukeren). Merk: appen krever også
   at `skjemamottak`-tabellen har rader for gjeldende årgang - en tom database gir
   oppstartsfeil.

5. **Datatilgang.** Team/tilgangsgruppen valgt under *Data* må ha lese/skrive-tilgang
   til produktbøtta til strukt-naering (appen bruker `/buckets/produkt/naringer/...`).
   VoF-fanen krever i tillegg at den delte VoF-bøtta legges til under delte bøtter
   (montert på `/buckets/shared/vof`); uten den starter appen, men fanen skjules.

## Image-kontrakt

Imaget som `tjeneste.image.repository:tjeneste.image.version` peker på må:

- Serve NSPEK/NØKU Dash-appen over HTTP på `$PORT` (default `8008`), på rot-stien `/`.
- Respektere `NSPEK_NOKU_STANDALONE=true` (serve på `/`, kjør headless på `0.0.0.0:$PORT`).
- Inneholde Python-avhengighetene appen faktisk bruker, inkludert `rpy2` med **R**
  (importeres av `ssb-dash-framework` ved oppstart - Kostra-pakken trengs *ikke*).
  Merk: Altinn/SUV/fagfunksjoner-pakkene brukes kun av pipelines utenfor appen og er
  utelatt fra imaget.
- Kjøre som uid `1000` med primærgruppe gid `100` (`users`): charten setter
  `fsGroup: 100`, så volumet montert på `/home/onyxia/work` er gruppeeid av gid 100.
  `ssb-dash-framework` skriver appens logg til `/home/onyxia/work/app.log`.
- Ha en skrivbar `HOME` (`/home/onyxia`); charten peker `XDG_CACHE_HOME` til
  `/home/onyxia/work/.cache` på det monterte volumet.

## Lokal validering

```bash
helm repo add dapla-lab-library https://statisticsnorway.github.io/dapla-lab-helm-charts-library
helm dependency build charts/nspek-noku
helm lint charts/nspek-noku
helm template nspek-noku charts/nspek-noku \
  --set istio.enabled=true \
  --set istio.hostname=nspek-noku.example.no \
  --set dapla.group=<ditt-team>-developers \
  --set database.cloudSqlProxy.instance=my-gcp-project:europe-north1:my-db-instance
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
| database.cloudSqlProxy.enabled | bool | `true` | Run a Cloud SQL Auth Proxy sidecar on localhost:5432. The app opens its Postgres connection pool at startup and cannot start without it. |
| database.cloudSqlProxy.instance | string | `""` | Cloud SQL instance connection name, <project>:<region>:<instance>, for the strukt-naering database. Must be set for the service to start. |
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
| networking.service.port | int | `8008` |  |
| networking.type | string | `"ClusterIP"` |  |
| nodeSelector | object | `{}` |  |
| oidc.enabled | bool | `true` |  |
| oidc.secretName | string | `""` |  |
| oidc.tokenExchangeUrl | string | `""` |  |
| podAnnotations | object | `{}` |  |
| podDisruptionBudget.enabled | bool | `true` |  |
| podLabels."onyxia.app" | string | `"nspek-noku"` |  |
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
| security.serviceEntry.hosts[5] | string | `"auth.ssb.no"` |  |
| security.serviceEntry.hosts[6] | string | `"data.ssb.no"` |  |
| security.serviceEntry.hosts[7] | string | `"www.ssb.no"` |  |
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
| tjeneste.image.repository | string | `"europe-north1-docker.pkg.dev/artifact-registry-5n/strukt-naering-docker/nspek-noku"` | Container repository (without tag) for the prebuilt NSPEK/NØKU app image. Built and published to strukt-naering's own registry by stat-naringer-dash's "Build nspek-noku service image" workflow - WIF push, same model as the Prodcom service. See README.md -> "Image-kontrakt". |
| tjeneste.image.version | string | `"latest"` | Image tag to deploy: "latest" (moving alias) or an immutable <date>-<sha> tag from the build workflow's Step Summary. |
| tolerations | list | `[]` |  |
| userAttributes.environmentVariableName | string | `"OIDC_TOKEN"` |  |
| userAttributes.userAttribute | string | `"access_token"` |  |
| userAttributes.value | string | `""` |  |
