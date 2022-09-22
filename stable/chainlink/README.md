# Chainlink

## Introduction
This is a helm chart for running a Chainlink node on Kubernetes. It includes a Postgres database for storing its data. The node is deployed as a statefulset, with the P2P OCR port exposed on a unique public load balancer. Meanwhile, the HTTP web UI is served via an ingress, so that it can benefit from automatic TLS and DNS management using [cert-manager](https://cert-manager.io/) and [external-dns](https://github.com/kubernetes-sigs/external-dns). Additionally, the chart defaults to securing the web UI endpoint using basic auth over HTTPS, using a dynamically generated credential generated via [kubernetes-secret-generator](https://github.com/mittwald/kubernetes-secret-generator).

# Example usage

```
nodeConfig:
  network: rinkeby
  ethUrl: wss://rinkeby-ws.naas.linkpool.io
  secureCookies: false
  enableGasUpdater: true
  apiUser: "example@linkpool.io"
  apiPassword: "<passed securely using a CI tool>"
  walletPassword: "<passed securely using a CI tool>""
  featureOffChainReporting: true
resources:
  requests:
    cpu: 1000m
    memory: 2048Mi
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` |  |
| fullnameOverride | string | `""` |  |
| global.postgresql.database | string | `"chainlink"` | the database for postgres-ha |
| global.postgresql.password | string | `"chainlink_everything777"` | the password for postgres-ha |
| global.postgresql.postgresqlDatabase | string | `"chainlink"` | the database for standalone postgres |
| global.postgresql.postgresqlPassword | string | `"chainlink_everything777"` | the password for standalone postgres |
| global.postgresql.postgresqlUsername | string | `"postgres"` | the username for standalone postgres |
| global.postgresql.username | string | `"postgres"` | the username for postgres-ha |
| httpService.port | int | `80` | the port to expose the ClusterIP on. |
| httpService.type | string | `"ClusterIP"` | the type of service for the http web ui. Defaults to ClusterIP for use with an ingress. |
| image.pullPolicy | string | `"Always"` |  |
| image.repository | string | `"smartcontract/chainlink"` |  |
| image.tag | string | `"1.1.0"` |  |
| ingress.annotations | object | `{"nginx.ingress.kubernetes.io/limit-connections":"5","nginx.ingress.kubernetes.io/limit-rpm":"500","nginx.ingress.kubernetes.io/limit-rps":"10","nginx.ingress.kubernetes.io/ssl-redirect":"true"}` | Annotations for the http web ui ingress. |
| ingress.auth.authFileContents | string | `"example@linkpool.io:$apr1$KeZZLiRh$H/3lxu/wcfAY9t4Qg5bcq0"` |  |
| ingress.auth.enabled | bool | `true` | Whether or not to protect the web UI ingress with basic auth. |
| ingress.auth.password | string | `"exampletobereplaced"` | the password for the ingress basic auth. |
| ingress.auth.secret.annotations | object | `{}` |  |
| ingress.auth.username | string | `"example@linkpool.io"` | the username for the ingress basic auth.  |
| ingress.class | string | `"ingress-nginx-external"` | The class to use for the ingress. |
| ingress.enabled | bool | `true` | Whether or not to enable the http web UI ingress. |
| ingress.path | string | `"/"` | The path to set on the http web UI ingress. Should be left at "/" |
| ingress.pathType | string | `"Prefix"` | The pathType to set on the http web UI ingress. Should be left at Prefix for most controllers. |
| ingress.staticDomains | list | `["node-example.company.tld"]` | The DNS names to expose the http web UI at. |
| ingress.tls.enabled | bool | `true` |  |
| ingress.tls.secretName | string | `"chainlink-ingress-tls"` |  |
| nameOverride | string | `""` |  |
| networkPolicy.httpIngressPolicy.enabled | bool | `false` | whether or not to include a network policy for allowing inbound traffic to the chainlink node from an ingress controller. Defaults to false. |
| networkPolicy.httpIngressPolicy.ingressControllerNamespace | string | `"ingress-nginx"` | The namespace of the ingress controller to allow access from. Defaults to `ingress-nginx`. |
| networkPolicy.namespacePolicy.enabled | bool | `false` | whether or not to include a network policy for restricting/isolating inbound traffic to the chainlink namespace. Defaults to false. |
| networkPolicy.p2pServicePolicy.enabled | bool | `false` | whether or not to include a network policy for allowing inbound traffic from the internet for P2P OCR via a LoadBalancer service. Defaults to false. |
| networkPolicy.prometheusPolicy.enabled | bool | `false` | whether or not to include a network policy for allowing inbound traffic to the chainlink node from a namespace running prometheus. Defaults to false. |
| networkPolicy.prometheusPolicy.prometheusNamespace | string | `"kube-prometheus-stack"` | The namespace of the prometheus service to allow access from. Defaults to `kube-prometheus-stack`. |
| nodeConfig.apiPassword | string | `"linkt0gETHER"` | the password for the node's admin credentials. Recommended to be passed through securely via your CI tool. |
| nodeConfig.apiUser | string | `"admin@linkpool.io"` | the username for the node's admin credentials. |
| nodeConfig.ethUrl | string | `"wss://"` | The Ethereum node to connect to. |
| nodeConfig.featureOffChainReporting | bool | `false` | Enables OCR. Enabled by default. |
| nodeConfig.logLevel | string | `"debug"` | The log level for the chainlink node |
| nodeConfig.network | string | `"mainnet"` | The Ethereum network to use. |
| nodeConfig.p2pBootstrapPeers | string | `nil` | Pass through specific bootstrap peers for OCR if desired. |
| nodeConfig.p2pPeerID | string | `nil` | Pass through a specific peer ID for OCR if desired. |
| nodeConfig.secureCookies | bool | `false` | Requires the use of secure cookies for authentication. |
| nodeConfig.tlsPort | int | `0` | the port used for HTTPS connections. 0 disables tls so that we can handle it in the kubernetes ingress. |
| nodeConfig.walletPassword | string | `"sd093KB+330b7S="` | the password for the node's wallet. Recommended to be passed through securely via your CI tool. |
| nodeSelector | object | `{}` |  |
| p2pService.enabled | bool | `false` | Whether or not to enable the P2P OCR service. |
| p2pService.port | int | `7138` | the port for the P2P OCR service. |
| p2pService.type | string | `"LoadBalancer"` | The type of service for the P2P endpoint. Defaults to loadBalancer to publicly expose for OCR connectivity.  |
| podAnnotations | object | `{}` |  |
| postgresOperator.backups.gcs.bucket | string | `nil` | The GCS bucket to use. |
| postgresOperator.backups.gcs.enabled | bool | `true` | whether or not to enable a gcs backup repo. Defaults to false. |
| postgresOperator.backups.gcs.retention.count | int | `14` | the number of backups to retain on gcs |
| postgresOperator.backups.gcs.retention.type | string | `"time"` | the type of backup units to retain. can be count or time. |
| postgresOperator.backups.gcs.scheduleEnabled | bool | `false` | whether or not to enable the backup schedule for the gcs backup |
| postgresOperator.backups.gcs.schedules.full | string | `"0 1 * * *"` | the cron schedule for full database backups to gcs |
| postgresOperator.backups.gcs.schedules.incremental | string | `"0 */4 * * *"` | the cron schedule for incremental database backups to gcs |
| postgresOperator.backups.gcs.secretName | string | `nil` | the secret containing the GCS config file and service account key. |
| postgresOperator.backups.volume.enabled | bool | `false` | whether or not to enable a backup volume. Defaults to true. |
| postgresOperator.backups.volume.retention.count | int | `14` | the number of backups to retain on the backup volume |
| postgresOperator.backups.volume.retention.type | string | `"time"` | the type of backup units to retain. can be count or time. |
| postgresOperator.backups.volume.scheduleEnabled | bool | `false` | whether or not to enable the backup schedule for the backup volume |
| postgresOperator.backups.volume.schedules.full | string | `"0 1 * * *"` | the cron schedule for full database backups to the backup volume |
| postgresOperator.backups.volume.schedules.incremental | string | `"0 */4 * * *"` | the cron schedule for incremental database backups to the backup volume |
| postgresOperator.backups.volume.size | string | `"1Gi"` | the size of the backup volume. |
| postgresOperator.backups.volume.storageClassName | string | `"standard"` | the storageclass to use for the backup volume. |
| postgresOperator.enabled | bool | `false` | whether or not to use a database provisioned via postgres-operator. |
| postgresOperator.instanceCPU | string | `"250m"` | sets the CPU limit for the Postgres instances. This defaults to no limit being set, but an example value is set below. Setting "instances" overrides this value. |
| postgresOperator.instanceMemory | string | `"256Mi"` | sets the memory limit for the Postgres instances. This defaults to no limit being set, but an example value is set below. Settings "instances" overrides this value. |
| postgresOperator.instanceSize | string | `"1Gi"` | sets the size of the volume that contains the data. This defaults to the value below. Settings "instances" overrides this value. |
| postgresOperator.monitoring | bool | `false` | enables the ability to monitor the Postgres cluster through a metrics exporter than can be scraped by Prometheus. This defaults to the value below. |
| postgresOperator.postgresVersion | int | `13` | sets the desired postgres version |
| postgresOperator.users | list | `[{"databases":["chainlink"],"name":"postgres","options":"SUPERUSER"}]` | a list of users to be configured on the postgres-operator managed db. Should usually be left as-is. |
| postgresql.enabled | bool | `true` | whether or not to deploy a standalone postgres database helm chart. |
| postgresqlHA.enabled | bool | `false` | whether or not to deploy a high availability postgres database helm chart. |
| rbac.create | bool | `true` | If set to true, rbac resources will be created for the service account. Required if using the chart's service account. |
| replicaCount | int | `1` |  |
| resources | object | `{}` |  |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` | If true, a service account will be created for the node. If set to false, you must set `serviceAccount.name` |
| serviceAccount.name | string | `nil` | The name of an existing service account to use for the node. Combined with `serviceAccount.create` |
| tolerations | list | `[]` |  |
