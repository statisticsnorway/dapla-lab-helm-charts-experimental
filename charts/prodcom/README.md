# prodcom

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

Prodcom - SSBs saksbehandlingsløsning for industriell vareproduksjon. Kjører Prodcom-appen
(Plotly Dash) som en frittstående tjeneste i Dapla Lab / Onyxia.

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

1. **Appbildet må bygges og publiseres.** Prodcom-teamet må bygge et image fra
   `stat-prodcom` og publisere det til registeret angitt i `tjeneste.image.repository`
   (default: `europe-north1-docker.pkg.dev/artifact-registry-5n/dapla-lab-docker/onyxia/prodcom`).
   Se *Image-kontrakt* under. En ferdig `Dockerfile` er foreslått i `stat-prodcom`
   (se medfølgende patch).

2. **`dash/app.py` må støtte frittstående kjøring.** Appen er opprinnelig skrevet for
   jupyter-server-proxy. En liten, bakoverkompatibel endring gjør at den kan serveres på
   rot-stien og kjøre headless, og at prod-data kun lastes når `DAPLA_ENVIRONMENT=PROD`
   (se medfølgende patch til `stat-prodcom`). Endringen påvirker ikke eksisterende bruk
   inne i JupyterLab.

3. **Datatilgang (kun produksjon).** I produksjonsmiljøet må team/tilgangsgruppen valgt
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
  disse brukes kun av pipelines utenfor `dash/` og kan utelates fra imaget.
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
| global.suspend | bool | `false` | Suspender (skaler til 0) tjenesten |
| tjeneste.image.repository | string | `"europe-north1-docker.pkg.dev/artifact-registry-5n/dapla-lab-docker/onyxia/prodcom"` | Container-repo for det ferdigbygde Prodcom-appbildet |
| tjeneste.image.version | string | `"latest"` | Image-tag som skal deployes |
| tjeneste.image.pullPolicy | string | `"Always"` | Image pull policy |
| networking.type | string | `"ClusterIP"` | Service-type |
| networking.clusterIP | string | `"None"` | Headless service |
| networking.service.port | int | `8061` | Porten Dash-appen lytter på i containeren |
| ressurser.requests.cpu | string | `"500m"` | Garantert CPU |
| ressurser.requests.memory | string | `"4Gi"` | Garantert minne (også brukt som memory limit) |
| dapla.group | string | `""` | Team/tilgangsgruppe hvis datatilganger aktiveres (må ha tilgang til Prodcom-bøtta) |
| dapla.sourceData.reason | string | `""` | Begrunnelse (kun for data-admins/kildedata) |
| dapla.sourceData.requestedDuration | string | `"4h"` | Varighet på tilgang |
| dapla.sharedBuckets | list | `[]` | Delte bøtter som skal gjøres tilgjengelige |
| avansert.data.mountStandard | bool | `true` | Monter standardbøtter under `/buckets` |
| security.serviceEntry.enabled | bool | `true` | Aktiver egress til eksterne tjenester |
| security.serviceEntry.hosts | list | GCS + SSB-auth | Hoster appen får nå (egress) |
| security.networkPolicy.enabled | bool | `false` | Aktiver network policy (ingress-policyen mot Istio rendres kun når `from` er satt) |
| security.oauth2.provider | string | `"keycloak-oidc"` | OAuth2-provider |
| security.oauth2.oidcIssuerUrl | string | `"overwritten-by-onyxia"` | OIDC issuer URL |
| oidc.enabled | bool | `true` | Opprett OIDC-secret |
| oidc.tokenExchangeUrl | string | `""` | Token exchange URL |
| istio.enabled | bool | `false` | Aktiver Istio-ingress |
| istio.hostname | string | `"chart-example.local"` | Ingress-hostnavn |
| diskplass.enabled | bool | `false` | Opprett et varig lokalt filsystem (PVC). Av som standard |
| diskplass.size | string | `"10Gi"` | Størrelse på PVC hvis aktivert |
| deleteJob.enabled | bool | `true` | Slett tjenesten automatisk hver kveld (kl. `cronHourAtDay` UTC) |
| deleteJob.cronHourAtDay | string | `"20"` | Time på døgnet (UTC) tjenesten slettes |
| podDisruptionBudget.enabled | bool | `true` | Opprett PodDisruptionBudget |
| replicaCount | int | `1` | Antall replicas |
| serviceAccount.create | bool | `true` | Opprett service account |
| kubernetes.enabled | bool | `false` | Gi tjenesten `view`-tilgang til eget namespace |
| deployEnvironment | string | `"TEST"` | Miljø tjenesten deployes i |
| environment.user | string | `"onyxia"` | Bruker i containeren |
| environment.group | string | `"users"` | Gruppe i containeren |
