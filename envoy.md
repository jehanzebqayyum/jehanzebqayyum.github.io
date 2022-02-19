# Envoy as Proxy

Envoy can act as a proxy, as a sidecar container, and complex redirection rules can be configured.

A simple envoy config has two major parts clusters (targets) and listeners (where forwarding rules can be configured)

So let's say we need a rule that a url with /v1 patterns goes to service-1 and /v2 to service-2. 
Where service-1 is being deprecated, and has envoy proxy to forward new urls to service-2.


## Sample envoy config

```
static_resources:
  listeners:
    - address:
        socket_address:
          address: 0.0.0.0
          port_value: 8080
      filter_chains:
        - filters:
            - name: envoy.filters.network.http_connection_manager
              typed_config:
                "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
                codec_type: AUTO
                stat_prefix: ingress_http
                route_config:
                  name: local_route
                  virtual_hosts:
                    - name: backend
                      domains:
                        - "*"
                      routes:
                        - match:
                            prefix: "/v1"
                          route:
                            cluster: service-1
                        - match:
                            prefix: "/v2"
                          route:
                            cluster: service-2
                http_filters:
                  - name: envoy.filters.http.router
                    typed_config: {}
  clusters:
    - name: service-1
      connect_timeout: 0.25s
      type: STRICT_DNS
      lb_policy: ROUND_ROBIN
      load_assignment:
        cluster_name: service-1
        endpoints:
          - lb_endpoints:
              - endpoint:
                  address:
                    socket_address:
                      address: service-1
                      port_value: 80
    - name: service-2
      connect_timeout: 0.25s
      type: STRICT_DNS
      lb_policy: ROUND_ROBIN
      load_assignment:
        cluster_name: service-2
        endpoints:
          - lb_endpoints:
              - endpoint:
                  address:
                    socket_address:
                      address: 127.0.0.1
                      port_value: 80
```

more complex regex are also possible e.g.

```
                        - match:
                            safe_regex:
                              google_re2: { }
                              regex: /v1/users/[^\/]+$
                            headers:
                            - name: ":method"
                              exact_match: "GET"
                          route:
                            cluster: service-2

```
