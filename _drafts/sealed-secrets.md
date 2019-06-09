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
It allows authentication over GitHub and there are [plugins for Jenkins](https://github.com/jenkinsci/hashicorp-vault-plugin) availabe to fetch secrets in Jobs.
<!--TODO: Un-comment?-->
<!--We mostly used these to inject secrets into container environments in deployments.-->

When [we started using Kubernetes]({% post_url 2019-06-02-gitops %}),

<!--
## Sealed secrets

Story:
* Move to Kubernetes (link gitops blog post): Secrets are stored in base64: encoded, not encrypted
* Runtime dependency on Vault, cumbersome init container needed: https://github.com/actano/kubernetes-vault-init
* I usually like to solve the problem at hand with a simple solution, only considering more complex ones if needed

## Sealed secrets
* When reading about GitOps we saw their recommendation of Sealed secrets (link blog article)
* Interface is the secret name
* [Mozillas sops](https://github.com/mozilla/sops) follows a similar
* Compare to how Travis CI allows encrypted secrets in their configuration

* We forked [Vault on GKE](https://github.com/sethvargo/vault-on-gke) to set up Vault on Kubernetes

* Helm template idea
* Vault-on-gke

* Sealed secrets provide static secrets, no vault connection on runtime needed.
* "Renders" secret templates and seal

* Secrets should not be committed into git
* Secrets are base64 plain-text
* Draft for encryption at rest (link and explain concept in new kubernetes versions)
* Store secrets in git
* Secret source (template, vault) to secret representation (sealed), to secret deployment (unseal on server)
    * => Diagram
* Example repository

## Findings
* Drawback of sealed secrets: Diffs in git are not reviewable. Since only the cluster has the private key, only it can know the secret's content.
Moreover, thanks to a session key (link) the sealed reprensentation.
This means that sealing the same secret again will yield a different result. While this is more secure (WHY?), it also makes reviewing changes harder.
Right now we agreed to avoid unnecessary re-seals.
-->

