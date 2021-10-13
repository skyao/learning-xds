---
title: "LDS API的概述"
linkTitle: "LDS API概述"
weight: 461
date: 2021-09-28
description: >
  介绍LDS API的服务定义和参数说明
---

### LDS服务定义

LDS的服务定义如下：

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/listener/v3/lds.proto

```protobuf
service ListenerDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.config.listener.v3.Listener";

  rpc DeltaListeners(stream discovery.v3.DeltaDiscoveryRequest)
      returns (stream discovery.v3.DeltaDiscoveryResponse) {
  }

  rpc StreamListeners(stream discovery.v3.DiscoveryRequest)
      returns (stream discovery.v3.DiscoveryResponse) {
  }

  rpc FetchListeners(discovery.v3.DiscoveryRequest) returns (discovery.v3.DiscoveryResponse) {
    option (google.api.http).post = "/v3/discovery:listeners";
    option (google.api.http).body = "*";
  }
}
```

### 请求参数

对于 DeltaDiscoveryRequest 和 DiscoveryRequest，请求参数中的 type_url 字段的取值是 "type.googleapis.com/envoy.config.listener.v3.Listener" (或者隐含)。

### 应答参数

对于 DiscoveryResponse 和 DeltaDiscoveryResponse，返回的资源类型被抽象为 `google.protobuf.Any` ，对于LDS返回的实际是 Listener 这个消息体，定义在 `api/envoy/config/listener/v3/listener.proto` 文件中: 

```protobuf
package envoy.config.listener.v3;

message Listener {

  string name = 1;

  core.v3.Address address = 2 [(validate.rules).message = {required: true}];


  repeated FilterChain filter_chains = 3;

  repeated ListenerFilter listener_filters = 9;

  ......
}
```

详细的字段定义请见后续的详细解说。

### 范例

来自envoy官方文档 [Life of a Request](https://www.envoyproxy.io/docs/envoy/latest/intro/life_of_a_request) 的例子: 

```yaml
  listeners:
  # There is a single listener bound to port 443.
  - name: listener_https
    address:
      socket_address:
        protocol: TCP
        address: 0.0.0.0
        port_value: 443
    # A single listener filter exists for TLS inspector.
    listener_filters:
    - name: "envoy.filters.listener.tls_inspector"
    # On the listener, there is a single filter chain that matches SNI for acme.com.
    filter_chains:
    - filter_chain_match:
        # This will match the SNI extracted by the TLS Inspector filter.
        server_names: ["acme.com"]
      # Downstream TLS configuration.
      transport_socket:
        name: envoy.transport_sockets.tls
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.DownstreamTlsContext
          common_tls_context:
            tls_certificates:
            - certificate_chain: {filename: "certs/servercert.pem"}
              private_key: {filename: "certs/serverkey.pem"}
      filters:
      # The HTTP connection manager is the only network filter.
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          use_remote_address: true
          http2_protocol_options:
            max_concurrent_streams: 100
          # File system based access logging.
          access_log:
          - name: envoy.access_loggers.file
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.access_loggers.file.v3.FileAccessLog
              path: "/var/log/envoy/access.log"
          # The route table, mapping /foo to some_service.
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["acme.com"]
              routes:
              - match:
                  path: "/foo"
                route:
                  cluster: some_service
          # CustomFilter and the HTTP router filter are the HTTP filter chain.
          http_filters:
          # - name: some.customer.filter
          - name: envoy.filters.http.router
```

在这个例子中，定义了以下内容：

1. 一个 listener ，绑定到 TCP 端口 443

    ```yaml
      # There is a single listener bound to port 443.
      - name: listener_https
        address:
          socket_address:
            protocol: TCP
            address: 0.0.0.0
            port_value: 443
    ```

    这个 listener 有一个 listener filters，用于支持 TLS

    ```yaml
        # A single listener filter exists for TLS inspector.
        listener_filters:
        - name: "envoy.filters.listener.tls_inspector"
    ```

    这个 tls inspector 会从请求中解析出 SNI，在这个例子中是 "acme.com"

2. 一个 filter chain, 设置有 filter chain match, 通过 `server_names` 来进行匹配：

    - ```yaml
            - filter_chain_match:
                # This will match the SNI extracted by the TLS Inspector filter.
                server_names: ["acme.com"]
        ```
        
        这样就会匹配到上面 SNI 为 "acme.com" 的请求。

3. filter chain 的 transport_socket，用于设置下游连接的传输套接字实现:

    ```yaml
          transport_socket:
            name: envoy.transport_sockets.tls
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.DownstreamTlsContext
              common_tls_context:
                tls_certificates:
                - certificate_chain: {filename: "certs/servercert.pem"}
                  private_key: {filename: "certs/serverkey.pem"}
    ```

    按照要求，为了启用 TLS，在typed_config中设置一个名称为 `envoy.transport_sockets.tls` 的传输套接字和 `DownstreamTlsContext`。

4. filter chain 的 network filters，范例中只设置了一个 HttpConnectionManager

    ```yaml
          filters:
          # The HTTP connection manager is the only network filter.
          - name: envoy.filters.network.http_connection_manager
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
    		......
    ```

    
