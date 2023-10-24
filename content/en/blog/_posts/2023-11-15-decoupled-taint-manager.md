---
layout: blog
title: "Kubernetes 1.29: Decouple TaintManager from NodeLifecycleController"
date: 2023-11-15
slug: kubernetes-1-29-decoupled-taint-manager
---

**Authors:** Yuan Chen (Apple), Andrea Tosatto (Apple)

This blog discusses a new feature in Kubernetes 1.29 to improve the handling of taint-based pod eviction on nodes. 

## Background

In Kubernetes prior 1.29, `NodeLifecycleController` applies predefined `NoExecute` taints (e.g., Unreachable, NotReady) when the nodes are determined to be unhealthy. After the nodes get tainted, `TaintManager` does its due diligence to start deleting running pods on those nodes based on `NoExecute` taints, which can be added by anyone. 

`NodeLifecycleController` combines these two independent functions:
   * Adding a pre-defined set of `NoExecute` taints to nodes based on the node conditions
   * Performing pod eviction on `NoExecute` taints

## Summary of changes

In Kubernetes 1.29, we introduce a feature that decouples `TaintManager` that performs taint-based pod eviction from `NodeLifecycleController` and makes them two separate controllers: 

- `NodeLifecycleController` adds taints to unhealthy nodes 
- `TaintEvictionController` performs pod deletion on nodes tainted with the `NoExecute` effect.  

Splitting `NodeLifecycleController` based on the above functionalities and moving taint-based eviction implementation out of `node-lifecycle-controller` into a separate and independent `taint-eviction-controller` can help disentangle code, improve the code maintainability, and make future extensions to either component more manageable.

## How to use the new feature?

A new feature gate `SeparateTaintManager` is added. When the feature is turned on, a user can use a new `kube-controller-manager' flag `--controllers=kube-eviction-controller` to disable the old `TaintManager`. For compatibility, a flag `taint-manager` can be used as well.

### FAQ

##### Does this feature change the existing behavior of taint-based pod evictions?
No, the taint-based pod eviction behavior remains the same. Also, if the feature gate `SeparateTaintManager` is turned off, the legacy `NodelifecycleController` with `TaintManager` will be used. 

##### Will enabling / using this feature result in increasing time taken by any operations covered by existing SLIs/SLOs?
It may slightly increase the communication overhead from applying node taints to performing pod eviction.

##### Will enabling / using this feature result in non-negligible increase of resource usage (CPU, RAM, disk, IO, ...) in any components?
No

### How can you learn more?

See more details in the [KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-scheduling/3902-decoupled-taint-manager/README.md).

### Acknowledgments

As with any Kubernetes feature, multiple people in the community have made contributions, from writing the KEP, implementing the new controller, to reviewing the KEP and code. Special thanks to Aldo Culquicondor (@alculquicondor), Maciej Szulik (@soltysh), Wei Huang (@Huang-Wei), Sergey Kanzhelevi (@SergeyKanzhelev), Ravi Gudimetla (@ravisantoshgudimetla), Deep Debroy (@ddebroy).
