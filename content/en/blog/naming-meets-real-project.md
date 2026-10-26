---
title: "The standard meets a real project"
date: 2026-10-22
weight: 1
description: "What surfaced when the mess.systems naming standard was applied to the infrastructure of a real product, and how that led to version 2.5."
---

A standard written in advance has a weak spot: it answers the questions its author thought to ask. To find the missing questions, you have to apply it to something real.

The mess.systems [naming standard](https://github.com/mess-systems/naming-standard) has now been through that test. It was used as the basis for the infrastructure repository of a real product: managed Kubernetes at a public cloud provider, plus a shared plane next to it that hosts sign-in for users and operators, a mesh network, a secrets store and CI runners. I do not name the project, the company or the provider here. Everything below is generalized and shown on the fictional product `x`.

The result of the review: the standard fit well, but 10 places had to be adapted or invented. They grew into [version 2.5](https://github.com/mess-systems/naming-standard/pull/5).

## Environment codes split into three sets

The standard defines short environment codes: `prd`, `stg`, `tst`, `dev`, `sbx`. In the project they showed up in three variants:

- in public DNS, `stage` and `test`, because they are easier for people to read;
- in most other places, `prd`, `stg`, `tst`, as the standard requires;
- in some accounts and secret paths, `prod` and `test`, inherited from the tools everything started with.

Each choice is reasonable on its own. Together they create a situation where one environment has three names, and an automated check can no longer tell that it is one environment.

Version 2.5 solves it this way. An environment alias is allowed, but only in public DNS: `api.stage.x.mess.systems` instead of `api.stg.x.mess.systems`, if the organization decides so. The choice is made once per organization and recorded in the catalog. Everywhere except public DNS, the code is used. Mixing code and alias in one organization is not allowed.

## An environment and an environment class are different things

Some resources in the project serve several environments at once: a cloud project for all non-production environments, a node pool, a shared database cluster. They were called `nonprod`. The standard had that word, but only as a SPIFFE trust domain, not as a naming axis.

Version 2.5 introduces the environment class: `prd`, `nonprod`, `sandbox`. Every environment code in the catalog has a class. A class is allowed in a name only for a resource that really serves several environments, for example the database cluster `x-records-store-nonprod`, while the database inside it is `x_records_store_stg`. If a resource belongs to one environment, a class in its name is forbidden, and a test catches it.

## The cluster name was not telling the truth

The project has one managed cluster for all environments, because that is cheaper for a small team. The cluster was named with the production suffix, and inside it there was also a node pool for non-production environments. The name promised something that was not there.

The standard does not forbid a shared cluster; that is an architecture decision. But it has to give such a cluster an honest name. The form is now `<product>-k8s-<env-or-class>` for a cluster of one environment or class, and `<product>-k8s-multi` for a cluster of several classes. Pools are named after the cluster: `x-k8s-multi-nonprod`. A separate rule: a cluster that contains a `nonprod` pool cannot be called `-prd`.

## External tokens were not covered

The standard had a naming rule for tokens of its own secrets store. In a real project, though, most tokens are external: personal access tokens for the git host, cloud API keys, keys for third-party services. The project came up with a good form for them, and it was worth taking into the standard almost unchanged:

`<issuer>-<consumer>-<scope>-<access>`, for example `forge-x-agent-infra-rw`.

The name comes with required metadata: `owner`, `consumer` and `expires` in ISO 8601. A token without an expiry date is not issued. The expiry in the metadata matters more than the name: it shows when rotation is due.

## Infrastructure as code and runners

In the build domain the standard knew about CI and the registry, but not about IaC. The project had to invent its own codes. Version 2.5 adds the `iac` service in the `build` domain, and the CI runner becomes a component of the `ci` service: `ci-runner.build.platform.prd.svc.mess.systems`. IaC stacks and the state key are described too: `<product-or-platform>/<stack>/<env>.tfstate`. The project invented that form as well.

## Where corp ends and platform begins

The standard has two contours: the corporate one (`corp`), home to employees and engineering systems, and the product one, which contains a separate product called `platform`. In the project, user sign-in, the mesh, secrets and runners first ended up in corporate planes, while ingress, GitOps and observability were in `platform`. The boundary followed history, not a rule.

The rule in 2.5: everything the product cannot run without in its own deployment belongs to `platform`. That covers sign-in for the product's customers and operators, the product's mesh, its secrets, runners and registry. If the product uses a shared corporate installation, that installation keeps its corporate name and the product refers to it. The corporate contour itself does not change. Four rules (external tokens, multi-role hosts, buckets, `iac`) are written as product contour rules, and corp can adopt them later by a separate decision.

## Three smaller things

- **Multi-role host.** At the start one virtual machine carries three roles. It is named after its main role; the others are listed in the `roles` metadata.
- **Images.** `<registry>/<product>/<service>[-<component>]:<tag>`, the `vX.Y.Z` tag is immutable, plus `main-<sha7>` and `pr-<n>`. The registry host lives in a single variable.
- **Buckets.** `<product>-<domain>-<service>-<env-or-class>`. If the provider requires globally unique names, an organization prefix is added. The decision is made once per provider and recorded in the catalog.

## One object, one environment code

The most useful finding was the smallest. In the pilot application the deployment object was named with `test` and its namespace with `tst`. The metadata schema did not catch it: each name on its own was valid.

So 2.5 adds a golden test. A resource has a `names` field with all its names, and the check requires the environment code to match in all of them. An alias is allowed only in public DNS keys. Mixing code and alias is an error. There were 5 golden cases; now there are 11, including 3 negative ones: mixed codes, an alias outside DNS, and a class on a single-environment resource.

## What I took from it

The standard survived contact with the project: the core (characters, environment codes, explicit `prd`, "a name grants no rights", provider kept out of names) did not need to change. What broke were the edges, where the standard was silent and the project filled the gap in its own way. Almost all the good decisions in 2.5 came from the project, not from the author's head.

What I do not know yet: whether 2.5 will survive a second project with a different cloud and a different team. If you apply a similar standard and it breaks in other places, [write to us](/about/). That is the next test.
