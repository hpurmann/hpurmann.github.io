---
layout: post
title: Declarative operations with Kubernetes and GitOps
category: "dev"
comments: true
---

<!--TODO: Split sentence-->
For the longest time, deploying our monolithic application was nothing of concern for the common software developer at my current employer, Actano.
After a successful build, the application instances would update without downtime. Magically! More accurately, it was custom shell scripts which implemented a [blue green deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html).

This approach had a few disadvantages:
* *Confidence:* Due do its imparative and non-idempotent nature, the development team was afraid to change anything. The risk of breaking something with no easy way back was too high.
* *Security:* The CI Job would SSH into the production servers and execute commands remotely.
* *Ownership:* It was originally authored by the CTO and another developer. The team wouldn't know how to handle its failure. Most developers didn't even bother looking at it.

Even with those problems, it worked out in the monolithic environment. However this drastically changed when the company decided to split up the monolith into domain specific services.

## Kubernetes

As I was interested in the topic, I got the opportunity to be one of the developers driving the change to Kubernetes – whilst learning about it myself.

Kubernetes is a container-orchestration platform. It handles application deployments and their scaling.
Resources are defined as objects in the YAML format. This declarative nature is a big win compared to the usual scripting.
Instead of saying "Add one more instance of my application" (imparative) you say "I want 3 instances to be running" (declarative).
Kubernetes then translates those declarations into executable commands internally.

Because everything is declarative, it is idempotent by nature. Applying the same declaration twice is always the same as doing it once.
If some change in the declaration doesn't provide the desired outcome, we can always go back by applying the former declaration.

<!--
* Configuration as Code => Kubernetes
* TODO: Explanation Kubernetes

## How to setup continuous delivery?

* GitOps: Declare cluster state in git repository, automated diff and sync of definition vs actual state
* Secure: Nothing connects to the cluster from the outside (link to weaveworks CI/CD security article)

* Terraform

* Learnings:
    * Environments are static

-->
