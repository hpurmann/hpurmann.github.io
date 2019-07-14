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

## Authorization service


<!--
* We tried [oauth2_proxy]()
* oauth2_proxy: Not maintained, separate installation for each exposed service (link to blog article with the nginx idea)
* Ambassador API gateway:
* [flexible authentication mechanism](https://www.getambassador.io/reference/services/auth-service/)

* Grafana, Kibana, Prometheus, Alertmanager,


## Findings
* Only one authentication mechanism supported. Not possible to deploy application internally by using oauth for internal access protection + application auth
-->
