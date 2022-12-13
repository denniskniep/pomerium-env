# Developing
https://www.pomerium.com/docs/deploying/from-source

## Create local certs
Go to pomerium repository and create certs

```
# Install mkcert.
go install filippo.io/mkcert@latest
# Bootstrap mkcert's root certificate into your operating system's trust store.
mkcert -install
# Create your wildcard domain.
mkcert "*.pomerium.localhost"
```

## Create config
Go to pomerium repository and create `.config.yaml`

## Start environment
```
sudo docker-compose up
```

## Debug in VisualStudio


# URL Mapping:

* https://pomerium.localhost:8443/ping (`pomerium/internal/controlplane/http.go`)
* https://pomerium.localhost:8443/healthz (`pomerium/internal/controlplane/http.go`)
* https://pomerium.localhost:8443/robots.txt 
* https://pomerium.localhost:8443/.well-known/pomerium (`pomerium/internal/controlplane/http.go`)
* https://pomerium.localhost:8443/.well-known/pomerium/jwks.json (`pomerium/internal/controlplane/http.go`)


* https://app.pomerium.localhost/ping/ (`pomerium/internal/controlplane/http.go`)
* https://app.pomerium.localhost/healthz (`pomerium/internal/controlplane/http.go`)
* https://app.pomerium.localhost/robots.txt 
* https://app.pomerium.localhost/.pomerium/ (`pomerium/proxy/handlers.go`)
* https://app.pomerium.localhost/.pomerium/jwt (`pomerium/proxy/handlers.go`)


# Questions
* What is the flow of httprequest to grafana?

# Codemap

## Startup
Class which containes the main startup logic:
`pomerium/pkg/cmd/pomerium/pomerium.go`

## Envoy

Envoy Server
`pomerium/pkg/envoy/envoy.go`

Copies the envoy config into a tmp folder e.g. `/tmp/pomerium-envoy3952078631/envoy`. This config file is generated via BootstrapConfigBuilder `pomerium/config/envoyconfig/bootstrap.go`. 

There is a static resource in the envoy config which is defining a  pomerium-control-plane-grpc cluster. 
This cluster has an endpoint which is an [Aggregated Discovery Service](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol#aggregated-discovery-service)

Threse is also a dynamic resource part in the envoy config, which points to the pomerium-control-plane-grpc. 

The ControlPlane builds the dynamic envoy config via `pomerium/internal/controlplane/xds.go` and stores it as resources in `pomerium/internal/controlplane/xdsmgr/xdsmgr.go` for envoys discovery via [xds protocol](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol).

Envoy Config Builder:
`pomerium/config/envoyconfig/builder.go`

Inspect current config with curl and [envoy admin endpoint](https://www.envoyproxy.io/docs/envoy/latest/operations/admin)

```
sudo curl --no-buffer -XGET --unix-socket /tmp/pomerium-envoy-admin.sock http:/socket/config_dump
```

The started Envoy Server is the first touchpoint with the http request of the user

## Authroize
on every request the grpc endpoint `Check` from 
`pomerium/authorize/grpc.go` is called by
Envoy 'External Authorization' Http Filter
https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/ext_authz/v3/ext_authz.proto


## Proxy
`pomerium/proxy/proxy.go`
proxy is a pomerium service that provides reverse proxying of
internal routes

## Reproxy
reproxy contains a handler for re-proxying traffic through the http controlplane.
`pomerium/internal/httputil/reproxy/reproxy.go`

## Controlplane
`pomerium/internal/controlplane/server.go`

## DataBroker


## Rego/OPA
`pomerium/pkg/policy/rules/rules.go` contains useful pre-defined rego AST rules.


`pomerium/pkg/policy/parser/parser.go` is a parser for Pomerium Policy Language.


## Config

`pomerium/config/policy.go`
struct for `routes` element in config

# Local IDP
https://www.pomerium.com/docs/guides/local-oidc