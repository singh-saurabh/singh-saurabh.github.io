---
title: "Lazy loading isn't the magic pill to fix AI Inference"
date: 2025-08-24
tags: ["infrastructure", "systems"]
description: "Why lazy loading container filesystems provides massive initial gains but degrades to modest 1.5-3x speedups when accounting for application startup and cache misses."
---

Long cold starts are an incredibly common problem for AI/ML workloads running on Kubernetes. A cold start occurs when a new container instance must pull and load an entire image with no caching available to speed up the process.

Since AI/ML container images are typically larger than 10 GB, pulling and loading up a new container takes several minutes. Any time savings are highly beneficial and can lead to thousands of dollars saved.

## Stages of Cold Start

Complete cold start time consists of two stages:

1. **Node provisioning:** When autoscaling from zero or adding nodes, cloud providers take 80-120 seconds to provision and boot new GPU instances. This is entirely cloud-dependent and outside user control.

2. **Container start:** Time to pull container image from registry, extract layers, and start the container.

We have little control over the first aspect, but we can vastly change the container start time as we will see below.

## Lazy Loading the filesystem

Analysis shows that for typical AI workloads, 76% of startup time is spent downloading container images, yet only 6.4% of image data is accessed during application startup. This extreme inefficiency creates an optimization opportunity: defer downloading unused files until actually accessed.

### Snapshotter Mechanisms

Snapshotters are containerd components that manage filesystem layers for containers. They determine when and how image data is retrieved from registries. The default snapshotter is called overlayfs. Some open source lazy loading snapshotters are Nydus, SOCI and eStarGZ.

### Expected Gains

When using a lazy loading snapshotter, the startup time for containers falls drastically. From our experiments with [fastpull](https://github.com/tensorfuse/fastpull), the initial gains lead to container startup times decreasing from minutes to seconds. This is a substantial gain, but as we see in the sections below, the gains we see initially degrade at every stage, leading to a much smaller eventual gain.

## Why do the initial Lazy Loading gains degrade

There are two important factors which reduce our initial 100x gain to a more modest 1.5-3x gain:

### 1. Application Startup time

As AI/ML workloads need to download large models, decompress them, load them into GPU memory and do model compilation, they realistically take several minutes to start up—i.e., on the same order of magnitude as the container startup time.

Let's consider a case where lazy loading should benefit by a large margin. Say the overlayfs container startup time is 10 mins, and lazy container startup time is 1 sec.

Let's look at the following graph to understand how this affects the speedup.

![Speedup degradation with application startup time](/images/posts/lazy-loading/speedup_degradation.png)

Even though we lose significant speedup due to application startup time, cache misses further reduce performance gains.

### 2. Lazy loading Cache Misses

On a high level, lazy loading starts two processes: one process starts to download all the files for the image from the remote registry, and the second process starts the container and uses the file if it has been downloaded already or fetches it from the remote registry. Thus the first process creates a cache, which the second uses.

In an ideal scenario, the first process runs ahead of the second process and has all the files already cached which the container needs.

In practice, cache misses occur when the container requests files before they're downloaded, causing lazy loading application startup times to be slower than overlayfs startup times.

For the same example, if the lazy loading application startup is 20% slower than overlayfs startup time, this is how the speedups look:

![Cache miss impact on speedup](/images/posts/lazy-loading/cache_miss_speedup2.png)

We can see that the gains decrease further due to cache misses.

Beyond performance considerations, implementing lazy loading introduces additional operational challenges:

## Intrusive Setup

Implementing lazy loading requires significant infrastructure changes:

### 1. CI/CD Changes

The build process now involves converting images from the overlayfs format to the custom snapshotter format. This increases build times, makes incremental builds slower, and requires extensive CI/CD pipeline changes. Maintaining both old and new snapshotter images also increases storage costs.

Additionally, you must verify that your registry is compatible with the new snapshotter (e.g., GAR does not support SOCI snapshotter).

### 2. Node Infrastructure Changes

You must install snapshotter plugins on all cluster nodes and configure containerd to use them. Different base OSes require different setup mechanisms, each of which must be tested to ensure optimal performance.

### 3. Testing Overhead

Every converted image must be tested to ensure it behaves identically to the original, and to check if it performs optimally.

## Provisioning Machines On Demand - A Challenge

Unless one reserves machines, which involves significant capital investment, every node which spins up will take an indefinite amount of time. This time will vary depending on:

1. The Cloud Provider (your choice of Hyperscaler or a NeoCloud)
2. Availability in the region you need the machine in

In our experience, the time to provision a GPU machine can vary from a minute to 15 minutes. In most cases, there aren't any SLAs for on demand compute, which makes it difficult to estimate how long it will take for a machine to become available.

Let's see how the lazy loading speedup changes with machine provisioning time. Going with our previous example and assuming application startup takes 5 minutes, overlayfs container startup time is 10 mins, lazy loading startup time is 1 second.

Application startup takes 5 minutes for overlayfs and 6 minutes (20% slower) for lazy loading:

![Provisioning time impact on speedup](/images/posts/lazy-loading/provisioning_time_speedup.png)

## Conclusions

Lazy loading of the container filesystem does provide a massive startup gain, especially for large images which are typical for AI/ML workloads. However, these gains diminish to more modest 1.25x - 3x speedups when we account for application startup times, and decrease further when considering on-demand machine provisioning times.

Along with these, implementing the lazy loading solution on a cluster involves significant work in changing CI/CD pipelines and making nodes compatible.

You can checkout our [fastpull tool on Github](https://github.com/tensorfuse/fastpull), which enables you to easily test your workflow with lazy loading snapshotters and comparing it with the standard overlayfs setup.
