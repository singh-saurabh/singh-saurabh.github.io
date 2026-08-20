---
title: "Building a Container Snapshotter for Sub-Second Cold Starts"
date: 2026-08-15
tags: ["infrastructure", "systems"]
description: "How we built Fastpull to achieve sub-second container cold starts using lazy filesystem loading."
---

At TensorFuse, every millisecond of container cold start time directly translates to user-perceived latency. This post walks through how we built Fastpull, our container snapshotter that achieves sub-second cold starts by lazily loading container filesystem layers on demand.

## The Problem with Traditional Container Starts

A typical container start involves pulling all image layers, unpacking them into an overlay filesystem, and then starting the process. For a 4GB PyTorch image with CUDA libraries, it's painfully slow.

The key insight is that most containers don't actually read their entire filesystem at startup.
