# TOGG Mobile API Gateway

[[_TOC_]]

## Description

This project uses Apache APISIX as a lightweight Mobile App Gateway, to address Backend for Frontend API Gateway
requirements with all advantages of the BFF design patterns so that the Mobile App can have one single access point to
backend features and use-cases in a mobile optimized way to enable the best features for customers.

Apache APISIX is a dynamic, real-time, high-performance API gateway. APISIX provides rich traffic management features
such as load balancing, dynamic upstream, canary release, circuit breaking, authentication, observability, and more.

We will use Apache APISIX to handle traditional north-south traffic to expose TOGG gRPC and REST microservices and
address BFF requirements.

This document describes the mobile API gateway setup, installation and customizations for TOGG
platform. [Apache APISIX helm charts](https://github.com/apache/apisix-helm-chart) used to provide the complete
installation.

## Environment Model

This setup includes an API Gateway, an API Gateway Admin Service, an etcd cluster and an API Dashboard.

![](docs/environment-model.png)

## Authentication and Authorization

API Gateway Admin Service and API Dashboard user authentication is performed through APISIX builtin mechanism at now,
soon It will be revisited to integrate with TOGG TIAM.

## Supported Services and Features

This installation provides only virtualizing http and gRPC microservices at the moment.

## Configurations

The following tables lists the configurable parameters of the apisix chart and their default values per section/component:

### Global parameters

| Parameter                 | Description                                     | Default                                                 |
|---------------------------|-------------------------------------------------|---------------------------------------------------------|
| `global.imagePullSecrets` | Global Docker registry secret names as an array | `[]` (does not add image pull secrets to deployed pods) |

### apisix parameters

| Parameter                                         | Description                                                                                                                                                                                                                                                                                                                  | Default                                                |
|---------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------|
| `apisix.enabled`                                  | Enable or disable Apache APISIX itself                                                                                                                                                                                                                                                                                       | `true`                                                 |
| `apisix.enableIPv6`                               | Enable nginx IPv6 resolver                                                                                                                                                                                                                                                                                                   | `true`                                                 |
| `apisix.enableCustomizedConfig`                   | Enable full customized `config.yaml`                                                                                                                                                                                                                                                                                         | `false`                                                 |
| `apisix.customizedConfig`                         | If `apisix.enableCustomizedConfig` is true, full customized `config.yaml`. Please note that other settings about APISIX config will be ignored                                                                                                                                                                               | `{}`                                                 |
| `apisix.image.repository`                         | Apache APISIX image repository                                                                                                                                                                                                                                                                                               | `apache/apisix`                                        |
| `apisix.image.tag`                                | Apache APISIX image tag                                                                                                                                                                                                                                                                                                      | `{TAG_NAME}` (the latest Apache APISIX image tag)      |
| `apisix.image.pullPolicy`                         | Apache APISIX image pull policy                                                                                                                                                                                                                                                                                              | `IfNotPresent`                                         |
| `apisix.kind`                                     | Apache APISIX kind use a `DaemonSet` or `Deployment`                                                                                                                                                                                                                                                                         | `Deployment`                                                     |
| `apisix.replicaCount`                             | Apache APISIX deploy replica count,kind is DaemonSet,replicaCount not become effective                                                                                                                                                                                                                                       | `1`                                                    |
| `apisix.podAnnotations`                           | Annotations to add to each pod                                                                                                                                                                                                                                                                                               | `{}`                                                   |
| `apisix.podSecurityContext`                       | Set the securityContext for Apache APISIX pods                                                                                                                                                                                                                                                                               | `{}`                                                   |
| `apisix.securityContext`                          | Set the securityContext for Apache APISIX container                                                                                                                                                                                                                                                                          | `{}`                                                   |
| `apisix.podDisruptionBudget.enabled`              | Enable or disable podDisruptionBudget                                                                                                                                                                                                                                                                                        | `false`                                                |
| `apisix.podDisruptionBudget.minAvailable`         | Set the `minAvailable` of podDisruptionBudget. You can specify only one of `maxUnavailable` and `minAvailable` in a single PodDisruptionBudget. See [Specifying a Disruption Budget for your Application](https://kubernetes.io/docs/tasks/run-application/configure-pdb/#specifying-a-poddisruptionbudget) for more details | `Not set` |
| `apisix.podDisruptionBudget.maxUnavailable`       | Set the maxUnavailable of podDisruptionBudget                                                                                                                                                                                                                                                                                | `1`                                                    |
| `apisix.resources`                                | Set pod resource requests & limits                                                                                                                                                                                                                                                                                           | `{}`                                                   |
| `apisix.nodeSelector`                             | Node labels for Apache APISIX pod assignment                                                                                                                                                                                                                                                                                 | `{}`                                                   |
| `apisix.tolerations`                              | List of node taints to tolerate                                                                                                                                                                                                                                                                                              | `{}`                                                   |
| `apisix.affinity`                                 | Set affinity for Apache APISIX deploy                                                                                                                                                                                                                                                                                        | `{}`                                                   |
| `apisix.podAntiAffinity.enabled`                  | Enable or disable podAntiAffinity                                                                                                                                                                                                                                                                                            | `false`                                                |
| `apisix.setIDFromPodUID`                          | Whether to use the Pod UID as the APISIX instance id, see [apache/apisix#5417](https://github.com/apache/apisix/issues/5417) to decide whether you should enable this setting)                                                                                                                                               | `false` |
| `apisix.customLuaSharedDicts`                     | Add custom [lua_shared_dict](https://github.com/openresty/lua-nginx-module#toc88) settings, click [here](https://github.com/apache/apisix-helm-chart/blob/master/charts/apisix/values.yaml#L27-L30) to learn the format of a shared dict                                                                                     | `[]` |
| `apisix.pluginAttrs`                              | Set APISIX plugin attributes, see [config-default.yaml](https://github.com/apache/apisix/blob/master/conf/config-default.yaml#L376) for more details                                                                                                                                                                         | `{}` |
| `apisix.luaModuleHook.enabled`                    | Whether to add a custom lua module                                                                                                                                                                                                                                                                                           | `false` |
| `apisix.luaModuleHook.luaPath`                    | Configure `LUA_PATH` so that your own lua module codes can be located                                                                                                                                                                                                                                                        | `""` |
| `apisix.luaModuleHook.hookPoint`                  | The entrypoint of your lua module, use Lua require syntax, like `"module.say_hello"`                                                                                                                                                                                                                                         | `""` |
| `apisix.luaModuleHook.configMapRef.name`          | Name of the ConfigMap where the lua module codes store                                                                                                                                                                                                                                                                       | "" |
| `apisix.luaModuleHook.configMapRef.mounts[].key`  | Name of the ConfigMap key, for setting the mapping relationship between ConfigMap key and the lua module code path.                                                                                                                                                                                                          | `""` |
| `apisix.luaModuleHook.configMapRef.mounts[].path` | Filepath of the plugin code, for setting the mapping relationship between ConfigMap key and the lua module code path.                                                                                                                                                                                                        | `""` |
| `extraVolumes`                                    | Additional `volume`, See [Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/) for the detail.                                                                                                                                                                                                          | `[]` |
| `extraVolumeMounts`                               | Additional `volumeMounts`, See [Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/) for the detail.                                                                                                                                                                                                    | `[]` |

### gateway parameters

Apache APISIX service parameters, this determines how users can access itself.

| Parameter                       | Description                                                                                                                                                                         | Default    |
|---------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| `gateway.type`                  | Apache APISIX service type for user access itself                                                                                                                                   | `NodePort` |
| `gateway.externalTrafficPolicy` | Setting how the Service route external traffic                                                                                                                                      | `Cluster`  |
| `gateway.http`                  | Apache APISIX service settings for http                                                                                                                                             |            |
| `gateway.tls`                   | Apache APISIX service settings for tls                                                                                                                                              |            |
| `gateway.tls.existingCASecret`  | Specifies the name of Secret contains trusted CA certificates in the PEM format used to verify the certificate when APISIX needs to do SSL/TLS handshaking with external services (e.g. etcd) | `""`       |
| `gateway.tls.certCAFilename`    | filename be used in the `gateway.tls.existingCASecret`                                                                                                                                          | `""`       |
| `gateway.stream`                | Apache APISIX service settings for stream                                                                                                                                           |            |
| `gateway.ingress`               | Using ingress access Apache APISIX service                                                                                                                                          |            |

### admin parameters

| Parameter                  | Description                                                                      | Default                                                 |
|----------------------------|----------------------------------------------------------------------------------|---------------------------------------------------------|
| `admin.enabled`            | Enable or disable Apache APISIX admin API                                        | `true`                                                  |
| `admin.port`               | which port to use for Apache APISIX admin API                                    | `9180`                                                  |
| `admin.servicePort`        | Service port to use for Apache APISIX admin API                                  | `9180`                                                  |
| `admin.type`               | Apache APISIX admin API service type                                             | `ClusterIP`                                             |
| `admin.externalIPs`        | IPs for which nodes in the cluster will also accept traffic for the servic       | `[]`                                                    |
| `admin.cors`               | Apache APISIX admin API support CORS response headers                            | `true`                                                  |
| `admin.credentials.admin`  | Apache APISIX admin API admin role credentials                                   | `edd1c9f034335f136f87ad84b625c8f1`                      |
| `admin.credentials.viewer` | Apache APISIX admin API viewer role credentials                                  | `4054f7cf07e344346cd3f287985e76a2`                      |
| `admin.allow.ipList`       | the IP range allowed to Apache APISIX admin API                                  | `true`                                                  |

### custom configuration snippet parameters

| Parameter                        | Description                                                                                        | Default |
|----------------------------------|----------------------------------------------------------------------------------------------------|---------|
| `configurationSnippet.main`      | Add custom Nginx configuration (main block) to nginx.conf                                          | `{}`    |
| `configurationSnippet.httpStart` | Add custom Nginx configuration (http block) to nginx.conf                                          | `{}`    |
| `configurationSnippet.httpEnd`   | Add custom Nginx configuration (http block) to nginx.conf, will be put at the bottom of http block | `{}`    |
| `configurationSnippet.httpSrv`   | Add custom Nginx configuration (server block) to nginx.conf                                        | `{}`    |
| `configurationSnippet.httpAdmin` | Add custom Nginx configuration (Admin API server block) to nginx.conf                              | `{}`    |
| `configurationSnippet.stream`    | Add custom Nginx configuration (stream block) to nginx.conf                                        | `{}`    |

### etcd parameters

| Parameter                       | Description                                                                                                                                                      | Default                     |
|---------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|
| `etcd.enabled`                  | use built-in etcd                                                                                                                                                | `true`                      |
| `etcd.host`                     | if `etcd.enabled` is false, use external etcd, support multiple address, if your etcd cluster enables TLS, please use https scheme, e.g. https://127.0.0.1:2379. | `["http://etcd.host:2379"]` |
| `etcd.prefix`                   | apisix configurations prefix                                                                                                                                     | `/apisix`                   |
| `etcd.timeout`                  | Set the timeout value in seconds for subsequent socket operations from apisix to etcd cluster                                                                    | `30`                        |
| `etcd.auth.rbac.create`        | Switch to enable RBAC authentication                                                                                                                                             | `false`                     |
| `etcd.auth.rbac.user`           | root username for etcd                                                                                                                                           | `""`                        |
| `etcd.auth.rbac.password`       | root password for etcd                                                                                                                                           | `""`                        |
| `etcd.auth.tls.enabled`         | enable etcd client certificate                                                                                                                                   | `false`                     |
| `etcd.auth.tls.existingSecret`  | name of the secret contains etcd client cert                                                                                                                     | `""`                        |
| `etcd.auth.tls.certFilename`    | etcd client cert filename using in `etcd.auth.tls.existingSecret`                                                                                                | `""`                        |
| `etcd.auth.tls.certKeyFilename` | etcd client cert key filename using in `etcd.auth.tls.existingSecret`                                                                                            | `""`                        |
| `etcd.auth.tls.verify`          | whether to verify the etcd endpoint certificate when setup a TLS connection to etcd                                                                              | `true`                      |
| `etcd.auth.tls.sni`            | specify the TLS [Server Name Indication](https://en.wikipedia.org/wiki/Server_Name_Indication) extension,  the ETCD endpoint hostname will be used when this setting is unset. | `""` |

If etcd.enabled is true, set more values of bitnami/etcd helm chart use etcd as prefix.

### plugins and stream_plugins parameters

Default enabled plugins. See [configmap template](https://github.com/apache/apisix-helm-chart/blob/master/charts/apisix/templates/configmap.yaml) for details.


### external plugin parameters

| Parameter                       | Description                                                                                                                                                      | Default                     |
|---------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|
| `extPlugin.enabled` | Enable External Plugins. See [external plugin](https://apisix.apache.org/docs/apisix/next/external-plugin/) | `false` |
| `extPlugin.cmd` | the command and its arguements to run as a subprocess | `{}` |

### custom plugin parameters

| Parameter                       | Description                                                                                                                                                      | Default                     |
|---------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|
| `apisix.customPlugins.enabled` | Whether to configure some custom plugins | `false` |
| `apisix.customPlugins.luaPath` | Configure `LUA_PATH` so that custom plugin codes can be located | `""` |
| `apisix.customPlugins.plugins[].name` | Custom plugin name | `""` |
| `apisix.customPlugins.plugins[].attrs` | Custom plugin attributes | `{}` |
| `apisix.customPlugins.plugins[].configMap.name` | Name of the ConfigMap where the plugin codes store | `""` |
| `apisix.customPlugins.plugins[].configMap.mounts[].key` | Name of the ConfigMap key, for setting the mapping relationship between ConfigMap key and the plugin code path. | `""` |
| `apisix.customPlugins.plugins[].configMap.mounts[].path` | Filepath of the plugin code, for setting the mapping relationship between ConfigMap key and the plugin code path. | `""` |

### discovery parameters

| Parameter                       | Description                                                                                                                                                                         | Default    |
|---------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| `discovery.enabled`                  | Enable or disable Apache APISIX integration service discovery | `false` |
| `discovery.registry`                  | Registry is the same to the one in APISIX [config-default.yaml](https://github.com/apache/apisix/blob/master/conf/config-default.yaml#L281), and refer to such file for more setting details. also refer to [this documentation for integration service discovery](https://apisix.apache.org/docs/apisix/discovery) | nil |

If you have enabled this feature,  here is an example:

```yaml
discovery:
  enabled: true
  registry:
    eureka:
      host:
        - "http://${username}:${password}@${eureka_host1}:${eureka_port1}"
        - "http://${username}:${password}@${eureka_host2}:${eureka_port2}"
      prefix: "/eureka/"
      fetch_interval: 30
      weight: 100
      timeout:
        connect: 2000
        send: 2000
        read: 5000
```

### logs parameters

| Parameter                       | Description                                                                                                                                                                         | Default    |
|---------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| `logs.enableAccessLog`                  | Enable access log or not, default true | `true` |
| `logs.accessLog`                  | Access log path | `/dev/stdout` |
| `logs.accessLogFormat`                  | Access log format | `$remote_addr - $remote_user [$time_local] $http_host \"$request\" $status $body_bytes_sent $request_time \"$http_referer\" \"$http_user_agent\" $upstream_addr $upstream_status $upstream_response_time \"$upstream_scheme://$upstream_host$upstream_uri\"` |
| `logs.accessLogFormatEscape`                  | Allows setting json or default characters escaping in variables | `default` |
| `logs.errorLog`                  | Error log path | `/dev/stderr` |
| `logs.errorLogLevel`                  | Error log level | `warn` |

### dashboard parameters

Configurations for apisix-dashboard sub chart.

### ingress-controller parameters

Configurations for Apache APISIX ingress-controller sub chart.

### serviceMonitor parameters

| Parameter                      | Description                                                  | Default                      |
| ------------------------------ | ------------------------------------------------------------ | ---------------------------- |
| `serviceMonitor.enabled`       | Enable or disable Apache APISIX serviceMonitor               | `false`                      |
| `serviceMonitor.namespace`     | Namespace where the serviceMonitor is deployed               | `""`                         |
| `serviceMonitor.interval`      | Interval at which metrics should be scraped                  | `15s`                        |
| `serviceMonitor.path`          | Path of the Prometheus metrics endpoint                      | `/apisix/prometheus/metrics` |
| `serviceMonitor.containerPort` | Container port which Prometheus metrics are exposed          | `9091`                       |
| `serviceMonitor.labels`        | ServiceMonitor extra labels                                  | `{}`                         |
| `serviceMonitor.annotations`   | ServiceMonitor annotations                                   | `{}`                         |

### initContainers parameters

| Parameter                      | Description          | Default   |
|--------------------------------|----------------------|-----------|
| `initContainer.image`          | Init container image | `busybox` |
| `initContainer.tag`            | Init container tag   | `1.28`    |

### TOGG specific configurations

- Istio service mesh settings at apisix values.yaml file.
- Istio Gateway and Istio VirtualService created for the APISIX gateway.
  See [Istio gateway yaml file](charts/apisix/templates/gateway.yaml)
  and [Istio virtual service yaml file](charts/apisix/templates/virtualservice.yaml)

## Setup

### 1. Prerequisites

- You need a Kubernetes cluster or minikube installation.
- You need an Istio installation
- You have to set Admin API credentials in the apisix values.yaml
- You have to set Authentication users and credentials in the apisix-dashboard values.yaml

### 2. Deploy Apache APISIX

Create Kubernetes namespace;

```shell
kubectl create ns togg-mobile-api-gateway
```

Helm install the application;

```shell
helm install togg-mobile-api-gateway -n togg-mobile-api-gateway ./charts/apisix/
helm install togg-mobile-api-dashboard -n togg-mobile-api-gateway ./charts/apisix-dashboard/
```

> **_NOTE:_** We are not using APISIX Ingress Controller, so we will not install it

### 3. Create Upstream and Routes in the Apache APISIX

You may use curl or APISIX Dashboard to create upstreams and routes to virtualize a microservices.

Here, we are providing sample curl scripts to demonstrate how to create an upstream and route for the TIAM Profile
microservice. In this scenario, Apache APISIX will act as an gRPC proxy server, and a gRPC client will be able to
consume gRPC virtual service which will be virtualized by Apache APISIX gateway.

```sh
# Create an upstream for TIAM ProfileService microservice
curl --location --request PUT 'http://{api-admin-service-host}:{admin-api-port}/apisix/admin/upstreams/1' \
--header 'X-API-KEY: put-your-admin-api-key-here' \
--header 'Content-Type: application/json' \
--data-raw '{
  "scheme": "grpc",
  "type": "roundrobin",
  "nodes": {
    "tiam-ms-profile-tiam-ms-profile.togg-id-services.svc.cluster.local:80": 1    
  }
}'

# List upstreams to check if any upstream is created for the TIAM Profile microservice
curl --location --request GET 'http://{api-admin-service-host}:{admin-api-port}/apisix/admin/upstreams/' \
--header 'X-API-KEY: put-your-admin-api-key-here'

# Create route for TIAM ProfileService microservice
curl --location --request PUT 'http://{api-admin-service-host}:{admin-api-port}/apisix/admin/routes/1' \
--header 'X-API-KEY: put-your-admin-api-key-here' \
--header 'Content-Type: application/json' \
--data-raw '{
  "methods": ["POST", "GET"],
  "uri": "/tiam.microservices.ProfileService/*",
  "upstream_id": "1"
}'

# List routes
curl --location --request GET 'http://{api-admin-service-host}:{admin-api-port}/apisix/admin/routes/' \
--header 'X-API-KEY: put-your-admin-api-key-here'
```

## Test API Gateway

Here, we demonstrate how to test TIAM Profile microservice using [grpcurl](https://github.com/fullstorydev/grpcurl);

```sh
./grpcurl -v -import-path /tiam-ms-profile/src/main/proto -proto tiam_microservices.proto --plaintext -d '{"request_context": {"tenantId": "togg", "userId": "put-here-user-id"}}' {api-gateway-host}:{api-gateway-port} tiam.microservices.ProfileService/GetUserProfile
```

In the TOGG Azure environment, we already configured all, and you may test TIAM Profile virtual backend service as the
following;

```sh
./grpcurl -v -import-path /tiam-ms-profile/src/main/proto -proto tiam_microservices.proto --plaintext -d '{"request_context": {"tenantId": "togg-id-dev", "userId": "ad859322-b6ff-4b88-96f0-58f53c5d42e2"}}' mobile-api-gateway.development.togg.cloud:80 tiam.microservices.ProfileService/GetUserProfile
```

## Creating a Virtual API for the UEP Backend API

You may check [here](docs/setup-uep-backend-api.md) to learn how to virtualize the UEP backend in Mobile API Gateway.

## Next Steps

- Security hardening for the Apache APISIX Admin API and dashboard user credentials

## Additional Resources

- [Apache APISIX Documentation](https://apisix.apache.org/docs/apisix/getting-started)
- [Apache APISIX Admin API Documentation](https://apisix.apache.org/docs/apisix/admin-api)
- [Apache APISIX gRPC Proxy Documentation](https://apisix.apache.org/docs/apisix/grpc-proxy)
