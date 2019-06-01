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
Instead of saying "Add one more instance of my application" (imparative style) you say "I want three instances to be running" (declarative style).
Kubernetes then translates those declarations into executable commands internally.

Because everything is declarative, it is idempotent by nature. Applying the same declaration twice is always the same as doing it once.
If some change in the declaration doesn't provide the desired outcome, we can always go back by applying the former declaration.
To do so easily, it is advised to store those in version control.

After having setup Kubernetes and a repository to store the application's Kubernetes objects, we are still left with two problems:
* How do we setup continuous deployment? For security reasons we generally try to avoid setups where the build server has credentials to acces the production cluster (see point 2 above).
* How do we ensure that the repositories state is on par with the cluster's state? Some kind of automation would be nice.

## GitOps

Enter GitOps. The term was coined initially by [Weaveworks](https://www.weave.works/technologies/gitops/) and promises to solve the above problems.
Its main component is a small application which follows the [Kubernetes Operator pattern](https://coreos.com/blog/introducing-operators.html).
The operator is deployed in the production cluster and watches for changes in a configured git repository.
The config repository serves as the source of truth for all Kubernetes resources.
If the desired cluster state diverges from the actual, the operator takes necessary actions to converge against it again.

To be able to do continuous deployment, the operator also watches your Docker image registries.
It detects new images and auto-upgrades your configuration files to reflect new image tags.

![Deployment pipeline with GitOps](/assets/gitops/deployment-pipeline.png "Deployment pipeline with GitOps")
<div class="caption">Image taken from the <a href="https://github.com/weaveworks/flux">weave flux repository</a></div>

This pull-based approach is [arguably more secure](https://www.weave.works/blog/how-secure-is-your-cicd-pipeline) than a push-based one.
The CI cluster doesn't know the credentials of the production cluster (though it has access to the image registry).
The interface between CI and CD is the Docker image. This interface makes it really easy to reason about the boundaries of CI and CD.

## Learnings

The GitOps approach enables our teams to propose service deployments or their changes via pull requests.
Errors can be spotted by more experienced DevOps engineers.
Still, they can make mistakes which will only be spotted after the configuration has been applied.
Other than with application code, there are no automated tests for our infrastructure.
You'd essentially need a mirror of the production cluster _for every single PR_.

The clear cut between CI and CD is nice and easy on a conceptional level. However, with multiple services,
some tests (e.g. end-to-end tests) require those services to be available and hence need to run in a shared environment with those.
This is not easy with flux right now because environments are static. Our approach right now is to continuously deploy into a staging environment
and manually promote this to the production environment.
[Jenkins X](https://jenkins.io/projects/jenkins-x/) claims to solve this by dynamically creating environments and automate their promotion.


<!--
### Related literature
* [How secure is your CI/CD pipeline?](https://www.weave.works/blog/how-secure-is-your-cicd-pipeline)

* GitOps: Declare cluster state in git repository, automated diff and sync of definition vs actual state
* Secure: Nothing connects to the cluster from the outside (link to weaveworks CI/CD security article)

* Terraform

* Learnings:
    * Environments are static
    * Split up services make it necessary to test after deployment (e.g. e2e tests) => link to JenkinsX?

-->
