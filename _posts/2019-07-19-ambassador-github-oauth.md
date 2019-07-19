---
layout: post
title: Exposing internal services behind oAuth
category: "dev"
comments: true
---

Operating web applications can be complicated. Luckily, a number of open source tools are available today to make our lives easier.
They give insights into deployed systems by providing solutions for logging, monitoring, distributed tracing and visualization.

In our [migration to Kubernetes]({% post_url 2019-06-02-gitops %}) we (re-)deployed those solutions.
In the past, we have secured access to such internal services with basic authentication configured in HAproxy.
This approach was suboptimal because it needed tedious manual work to update the list of users.

Instead, it would be nice to use an existing identity provider.
We are using GitHub to host our source code. Therefore it seemed natural to use oAuth2 with GitHub as the resource server.

For the first service, we tried [Bitly's oauth2_proxy](https://github.com/bitly/oauth2_proxy). Unfortunately, the project is no longer maintained and only allows a single upstream. Of course it is possible to expose multiple services behind [an nginx reverse proxy](https://carlosbecker.com/posts/prometheus-authentication-with-oauth2_proxy/) but this setup seemed quite cumbersome to use.

We wanted to achieve two things:
1. Single sign on (SSO) for all internal services
1. Expose each service at its own (sub)domain

We found a solution by chance. When evaluating different API gateway solutions, we ended up choosing [Ambassador](https://www.getambassador.io/). It is built on [the envoy proxy](https://www.envoyproxy.io/), which we already use in our [service mesh istio](https://istio.io/). Ambassador's [configuration is decentralized](https://www.getambassador.io/about/microservices-api-gateways/#self-service-publishing). This allows service teams to autonomously expose their services, without the help of an operations team. Adding an annotation to a service is enough to expose it.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  annotations:
    getambassador.io/config: |
      ---
      apiVersion: ambassador/v1
      kind: Mapping
      ambassador_id: production-gateway
      name: my-service-mapping
      prefix: /my-service
      service: my-service.prod:80
spec:
  ...
```

This configuration will expose `my-service` on `https://api.example.com/my-service`, given that the ambassador instance `production-gateway` is exposed on that particular domain.

## Authentication and authorization service

Ambassador provides a [flexible mechanism to provide custom authorization](https://www.getambassador.io/reference/services/auth-service/) for your API. All requests must pass through this service before they reach their intended destination. A `200 OK` response from the `AuthService` indicates successful authentication and makes Ambassador forward it to the actual service.


```yaml
annotations:
  getambassador.io/config: |
    ---
    apiVersion: ambassador/v1
    kind: AuthService
    name: authentication
    ambassador_id: internal-gateway
    auth_service: "ambassador-github-oauth.internal-gateway:3000"
    proto: http
    ---
    apiVersion: ambassador/v1
    kind: Mapping
    name: login_mapping
    ambassador_id: internal-gateway
    prefix: /auth/login
    rewrite: /auth/login
    service: ambassador-github-oauth.internal-gateway:3000
    bypass_auth: true
```

Notice the `bypass_auth` directive to disable authentication for requests to the `/auth/login` route.

Because nothing similar existed, we wrote a small service which provides [GitHub OAuth2 authentication for Ambassador](https://github.com/actano/ambassador-github-oauth).
We deployed a separate Ambassador instance named `internal-gateway` and exposed it on a `internal.yourdomain.com` domain.
A GitHub OAuth App is registered to point to this domain.

[Certmanager](https://github.com/jetstack/cert-manager) manages automatic certificate issuance and renewals:
```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: internal-gateway-tls
  namespace: internal-gateway
spec:
  secretName: internal-gateway-tls
  dnsNames:
    - internal.yourdomain.com
  issuerRef:
    name: letsencrypt
    kind: ClusterIssuer
  acme:
    config:
      - dns01:
          provider: <dns-provider-name>
        domains:
          - internal.yourdomain.com
```

## Exposing internal services

With the above in place, all development teams could expose their internal services with minimal configuration:
```yaml
annotations:
  getambassador.io/config: |
    ---
    apiVersion: ambassador/v1
    kind: TLSContext
    name: grafana_context
    ambassador_id: internal-gateway
    hosts:
    - grafana.yourdomain.com
    secret: grafana-tls.monitoring
    ---
    apiVersion: ambassador/v1
    kind: Mapping
    name: grafana_mapping
    ambassador_id: internal-gateway
    host: grafana.yourdomain.com
    prefix: /
    service: monitoring-grafana.monitoring:80
```

<!--
TODO: Make umbrella chart public, link it here

## Findings
* Only one authentication mechanism supported. Not possible to deploy application internally by using oauth for internal access protection + application auth
-->
