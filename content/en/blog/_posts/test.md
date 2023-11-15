---
layout: blog
title: "Kubernetes 1.29: Decouple TaintManager from NodeLifecycleController"
date: 2023-11-15
slug: kubernetes-1-29-decoupled-taint-manager
---

## Background

In Kubernetes 1.29, an  improvement has been introduced to enhance the handling of taint-based pod eviction on nodes. This blog discusses the changes made to the `NodeLifecycleController` to separate its responsibilities and improve overall code maintainability.

### Summary of Changes

The `NodeLifecycleController` previously combined two independent functions:

- Adding a pre-defined set of `NoExecute` taints to nodes based on the node conditions.
- Performing pod eviction on NoExecute taints

With the latest release, the taint-based eviction implementation has been moved out of the `node-lifecycle-controller` into a separate and independent component called `TaintEvictionController`. This separation aims to disentangle code, enhance code maintainability, and facilitate future extensions to either component.

### How to Use the New Feature?

A new feature gate, `SeparateTaintManager`, has been added. To enable the new feature, users can use the following flags:

- kube-apiserver: `--feature-gates=SeparateTaintManager=true`
- kube-controller-manager: `--controllers=kube-eviction-controller`

For compatibility, the legacy `TaintManager` can still be used with the `--taint-manager` flag.

### FAQ

**Does this feature change the existing behavior of taint-based pod evictions?**

No, the taint-based pod eviction behavior remains unchanged. If the feature gate `SeparateTaintManager` is turned off, the legacy `NodeLifecycleController` with `TaintManager` will continue to be used.

**Will enabling/using this feature result in an increase in the time taken by any operations covered by existing SLIs/SLOs?**

It may slightly increase the communication overhead from applying node taints to performing pod eviction.

**Will enabling/using this feature result in a non-negligible increase in resource usage (CPU, RAM, disk, IO, ...) in any components?**

No.

### Learn More

For more details, refer to the [KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-scheduling/3902-decoupled-taint-manager/README.md).

### Acknowledgments

As with any Kubernetes feature, multiple community members have contributed, from writing the KEP to implementing the new controller and reviewing the KEP and code. Special thanks to:

- Aldo Culquicondor (@alculquicondor)
- Maciej Szulik (@soltysh)
- Wei Huang (@Huang-Wei)
- Sergey Kanzhelevi (@SergeyKanzhelev)
- Ravi Gudimetla (@ravisantoshgudimetla)
- Deep Debroy (@ddebroy)
