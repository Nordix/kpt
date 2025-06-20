---
title: The Rationale behind kpt
linkTitle: The Rationale behind kpt
description:
toc_hide: true
menu:
  main:
    parent: "Guides"
---

Most Kubernetes users either manage their resources using conventional imperative graphical user interfaces,
command-line tools (kubectl), and automation (e.g., Operators) that operate directly against Kubernetes APIs, or
declarative configuration tools, such as Helm, Terraform, cdk8s, or one of the
[dozens of other tools](https://docs.google.com/spreadsheets/d/1FCgqz1Ci7_VCz_wdh8vBitZ3giBtac_H8SBw4uxnrsE/edit#gid=0).
At small scale, this is largely driven by preference and familiarity.

As companies expand the number of Kubernetes development and production clusters they use, creating and enforcing
consistent configurations and security policies across a growing environment becomes difficult. At that point, the
choice of management surface is no longer driven by preference, but by capabilities. To address this challenge, it is
increasingly common for platform administrators to use “GitOps” methodology to deploy configuration consistently across
clusters and environments with a version-controlled deployment process. Using the same principles as Kubernetes itself,
GitOps reconciles the desired state of clusters with a set of Kubernetes declarative configuration files in a source
control system, namely git.   

The imperative and declarative paradigms are not interoperable because most declarative configuration tools use a
generator-only approach that assumes exclusive actuation, and other changes to the live state are considered to be
undesirable configuration drift. One of the main features of GitOps tools is that their continuous reconciliation
mechanisms automatically detect and remediate drift.

Imperative tools are generally considered easier to use and adopt, but become toilsome to use at scale due to the lack
of reusability and automation. On the other hand, Infrastructure as Code provides more power and control, but the use of
complex code or code-like representations, such as templates, domain-specific configuration languages, and
general-purpose programming languages, requires manual human authoring and editing. This has been described as artisanal
automation due to the expertise and time required. Much of this effort is redundant, as the same patterns recur in every
package in which the same resource types appear.

Additionally, code-like representations whose consumption interfaces to configuration packages consist of package
parameters (e.g., [values.yaml](https://helm.sh/docs/chart_template_guide/values_files/) for Helm charts and
[input variables](https://github.com/terraform-google-modules/terraform-google-kubernetes-engine#inputs) for Terraform
modules) suffer from the problem of
[excessive parameterization](https://github.com/kubernetes/design-proposals-archive/blob/main/architecture/declarative-application-management.md#parameterization-pitfalls).
The inherent
[tradeoff between flexibility and usability / simplicity](https://en.wikipedia.org/wiki/Flexibility%E2%80%93usability_tradeoff)
drives off-the-shelf packages to degenerate into
[struct constructors](https://docs.google.com/presentation/d/1w4fkDNcYjvxie4GRqYuoE1oLTybsZk1anLjirN1aVnc/edit?ts=5fc7e108&pli=1#slide=id.gaeddea60e5_1_173),
as complex as the resource types they contain, but obfuscating their formal APIs. Even a thin abstraction obstructs the
ability to leverage the ecosystem of tools that can be used with the built-in Kubernetes resource types, and closes the
door to other automation. 

It is an emerging trend to provide GUIs over GitOps, but their ability to automate changes to configurations in git so
far has been quite limited and narrow, such as to change template parameter values or configure the GitOps
reconciliation process itself, because the representations are generaly so hostile to mechanical manipulation. The
combination of these limitations creates friction in managing infrastructure across multiple teams and applications.
Cross-functional collaboration across platform and application teams can quickly become a bottleneck especially as the
needs of individual teams differ from one another, requiring frequent template changes that potentially affect all uses
of the templates.

Fortunately, code-like representations are not a requirement in order to represent the desired state declaratively. The
Kubernetes API was
[designed to be natively declarative](https://github.com/kubernetes/design-proposals-archive/blob/main/architecture/resource-management.md#declarative-configuration).
The serialized configuration format of resources is
[identical to their API wire format](https://github.com/kubernetes/design-proposals-archive/blob/main/architecture/declarative-application-management.md#configuration-using-rest-api-resource-specifications).
These resource representations were intended to form the core of a declarative data model, which is sufficient in order
to support pre-deployment validation, preview, and approval and post-deployment auditing, versioning, and undo. 

kpt supports management of
[Configuration as Data](https://github.com/kptdev/kpt/blob/main/docs/design-docs/06-config-as-data.md).
The core ideas are simple:

* uses a uniform, serializable data model to represent configuration
  ([KRM](https://github.com/kubernetes/design-proposals-archive/blob/main/architecture/resource-management.md))
* makes configuration data (packages) the source of truth, stored separately from the live state 
* separates code that acts on the configuration (functions) from the configuration data
* abstracts the storage layer (using the
  [function I/O spec](https://github.com/kubernetes-sigs/kustomize/blob/master/cmd/config/docs/api-conventions/functions-spec.md))
  so that clients manipulating configuration data don’t need to directly interact with it

kpt builds on our learnings from [Kustomize](https://kustomize.io). Like kustomize, kpt uses a transformation-based
approach, but optimizes for in-place configuration transformation rather than out-of-place transformation. As with
typical API clients, this enables interoperability of a variety of generators, transformers, and validators. One doesn't
need to make all changes through a monolithic generator implementation. 

kpt also extends its capabilities in areas that are
[out of scope](https://github.com/kubernetes/design-proposals-archive/blob/main/architecture/scope.md#examples-of-projects-and-areas-not-in-scope),
notably packaging, and provides a
[package orchestration service](https://github.com/kptdev/kpt/blob/main/docs/design-docs/07-package-orchestration.md)
in addition to a client-side CLI.

kpt enables WYSIWYG management of configuration similar to how the live state can be modified with traditional
imperative tools, thus eliminating this dichotomy:
<img src="https://raw.githubusercontent.com/kptdev/kpt/main/docs/design-docs/CaD%20Overview.svg">

Configuration as Data is a novel approach that doesn’t sacrifice usability or the potential for higher-level automation
in order to enable reproducibility. Instead, it supports an interoperable, WYSIWYG, automatable configuration authoring
and editing experience.

