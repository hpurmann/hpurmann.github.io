---
layout: post
title: Managing secrets the GitOps way
category: "dev"
comments: true
---

<i>This post is a follow-up on the [Kubernetes and GitOps post]({% post_url 2019-06-02-gitops %}) from previous week.</i>

Secrets are an integral part of any application. They need credentials to access a database or authenticate against other external applications.
To store secrets, it is advisable to use a secret storage system. Nowadays, there are [several options out there](https://gist.github.com/maxvt/bb49a6c7243163b8120625fc8ae3f3cd),
many of which are open source.

One of those options is [Vault by HashiCorp](https://www.vaultproject.io/). It is probably the most popular one.
Vault stores secrets in a tree-like structure, allowing fine-grained policy definitions.
It supports authentication over GitHub and there are [plugins for Jenkins](https://github.com/jenkinsci/hashicorp-vault-plugin) available so that Jobs can retrieve secrets.

## kubernetes-vault

When [we started using Kubernetes]({% post_url 2019-06-02-gitops %}), we searched for a solution to populate our secrets from Vault.
First, we tried [Boostport's kubernetes-vault controller](https://github.com/Boostport/kubernetes-vault).
It watches for new pods with a certain annotation and expects them to have a special init container defined.
As we rolled this out to more deployments, we realized how cumbersome this approach actually is:
* Each deployment needs an additional init container, a volumes and volume mount. There is a lot of copy-pasting involved.
* It creates a runtime dependency on Vault. Because our Vault deployment was not highly available it created a single point of failure.

## Sealed secrets

At that time we read about GitOps and [Weaveworks' recommendation](https://www.weave.works/blog/storing-secure-sealed-secrets-using-gitops) to use [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets).
The idea is pretty simple: You locally encrypt your secrets with a public key and only the cluster has the private key to decrypt them.
A custom resource definition `SealedSecret` is introduced. The sealed-secrets controller lives in its own namespace and watches for these definitions.
It transparently decrypts the `SealedSecret` and creates a `Secret` with the same name and namespace.

![Sealed secrets usage](/assets/sealed-secrets/sealed-secrets-usage.png "Sealed secrets usage")

This approach allows you to store encrypted secrets in version control. It is similar to how [Travis CI solves adding secret environment variables](https://docs.travis-ci.com/user/encryption-keys).
Both aforementioned problems are solved by it:
* Each deployment can expect secret resources to be there and reference them by name.
* Deployments are independent of the availability of Vault.

But how do we transform vault secret values into sealed secrets? Let's look at that next.

## Sealed secrets workflow

Inspired from [Helm](https://helm.sh/) and its usage of Go templates, my colleague [Thomas Rucker](https://github.com/thrucker) wrote a [Go template function to fetch secrets from Vault, `vault-template`](https://github.com/actano/vault-template).
Using this, we are able to write secret template files such as this:

{% raw %}
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
  namespace: production
type: Opaque
data:
  config.yml: |
    database:
      admin_username: {{ vault "secret/database" "username" }}
      admin_password: {{ vault "secret/database" "password" }}
```
{% endraw %}

The `vault` template function will fetch the secret in `secret/database` and get its fields `username` and `password` respectively.
Sealing this secret is just a matter of invoking `kubeseal`, the CLI for sealed secrets.


To make this process easier for the dev teams we wrote a small wrapper which also preserves existing folder structures.
The first draft was [a bash script](https://gist.github.com/hpurmann/fa5bcfc5c3ac392cf8ef6a4d9a7f48b6) which we later transformed [into a helm plugin](https://github.com/actano/helm-sealed-secrets).

![Sealed secrets rendering](/assets/sealed-secrets/sealed-secrets-render.png "Sealed secrets rendering")
<div class="caption">From secret template to rendered sealed secret</div>

## Learnings

Because only the cluster has the private key to unseal, changes made to the sealed representation are not reviewable.
Moreover, thanks to a [session key](https://github.com/bitnami-labs/sealed-secrets#details) used during the encryption process, the sealed result is different on each invocation.
This further complicates code reviews. In my team, we found agreement to avoid unnecessary re-seals.

[Mozilla's SOPS](https://github.com/mozilla/sops) solves this by allowing multiple key pairs where one of them is readable from the developers machine. This approach seems worth looking at.

Another issue is related to the rotation of secrets: Because there is no direct connection from the user of a secret to the secret store, automated secret rotation is not easily achievable.

## Conclusions

Sealed secrets allow us to follow through on the GitOps approach and keep all of our configuration in a git repository.
Even though the future of this project [is not clear right now](https://github.com/bitnami-labs/sealed-secrets/issues/165) I certainly hope it continues to grow and be maintained.

<!--
{% raw %}
```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: mysecret
  namespace: production
spec:
  encryptedData:
    config.yml: AgBbbYUFeLHVlLrIh6rXb/VUTohTX+twTOst8fcfyV6csX8P0JGSmVuNuQ/0uTM4MxuCtiHO59x3DbdIcOZ78FZmVlPsmEd8LrpW3rP72DmT0T4dQDT7jvG/n5v42knBgNo5Oi9MeoBhzQDxQrRJxvhOhzake8i04yrXxFJebkIiJwpA0DTXlJI5QeyB2iNVnr+XmsPFuYxuJ2i7t4YJVlEm6QO6k0W584CKreWcWgt8fgW ...
```
{% endraw %}
-->

<!--
Additional:
* We forked [Vault on GKE](https://github.com/sethvargo/vault-on-gke) to set up Vault on Kubernetes

* Store secrets in git
* Secret source (template, vault) to secret representation (sealed), to secret deployment (unseal on server)
    * => Diagram
* Example repository
* [Mozillas sops](https://github.com/mozilla/sops) follows a similar
* vault-template travis build

## Findings

* Rolling secrets is a manual process. While making deployments easier we make
-->

