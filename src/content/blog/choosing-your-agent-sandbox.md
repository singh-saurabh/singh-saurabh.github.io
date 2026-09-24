---
title: "The decisions you need to make before choosing your agent sandbox"
date: 2026-09-23
tags: ["agents", "systems", "infrastructure"]
description: "Choose an agent sandbox around four decisions: access, runtime, persistence, and where work and data can go."
coverImage:
  src: "/images/social/choosing-your-agent-sandbox.jpg"
  alt: "Blue path branching through distinct architectural frames"
---

You're building or integrating a cloud coding agent. Its sandbox is the isolated environment where it runs commands and edits files. Before comparing providers, answer four questions about the work it will do.

Take a bug-fixing agent that edits a repository, runs tests, opens a preview, and waits overnight for review.

## 1. What must the code be unable to reach?

Our agent installs dependencies. Suppose an install script is malicious. List what it must not reach: other customers' workspaces, the host, production credentials, or your private network.

Ordinary containers share a host kernel. gVisor intercepts system calls through its own application kernel; a VM runs a guest kernel behind virtualized hardware. These choices affect compatibility as well as isolation. [[1]](#reference-1)

A VM doesn't protect a production token that you put inside it. Decide which credentials the task gets, when they expire, and which actions need a separately authorized tool.

Set resource limits too. Docker containers have no CPU or memory limits by default; those require explicit configuration. [[2]](#reference-2)

## 2. Can it run the whole task at the speed you need?

Start with the agent's actual repository. Clone it, install dependencies, start its services, and reproduce the bug. Include any need for Docker, a GPU, a browser, or a particular operating system. For example, Devin's macOS environment includes Xcode and the Simulator. [[3]](#reference-3)

Then check how the agent controls those commands. The [first article](/blog/where-should-my-agent-live/) separated the harness, which manages the agent loop, from the environment executing commands. Anthropic moved Cowork's loop outside its VM so the assistant could still respond when VM startup failed. [[4]](#reference-4) An external harness needs output streaming, cancellation, and reconnection. It must distinguish a failed command from a dropped connection while the command keeps running.

Measure time to useful work. Blaxel advertises provisioning and connection in under a second, and standby resume in about 25 milliseconds. [[5]](#reference-5) A fresh environment may still need repository setup; a resumed one may already have it. Neither advertised timing tells you when your task is ready.

Test the concurrency you expect. Record slow starts, failures, and total task time. For our interactive agent, measure the delay from a user request to useful work. For a nightly fleet, measure how many tasks finish before the deadline.

## 3. What must survive a wait or a failure?

Our agent now waits overnight for review. Decide whether compute keeps running, pauses, or gets rebuilt. Check how the provider detects idleness: a build can be busy without generating network traffic.

Specify what survives each transition. Freestyle's pause preserves memory and processes; stopping a persistent VM retains disk but discards memory. [[6]](#reference-6) E2B's default pause saves filesystem and memory, but clients connected to services inside must reconnect after resume. [[7]](#reference-7)

[![Example lifecycle contracts: pausing with memory preserves files and process state; stopping with retained disk requires new processes; rebuilding recovers only saved files. Behavior depends on the provider and settings.](/images/posts/choosing-your-agent-sandbox/lifecycle.svg)](/images/posts/choosing-your-agent-sandbox/lifecycle.svg)

Write a recovery requirement: “After waiting overnight, continue with uncommitted changes.” Set task limits, retention, and cleanup rules. E2B documents that paused sandboxes remain until explicitly deleted, so cleanup needs an owner. [[7]](#reference-7) The [storage and recovery article](/blog/agent-sandbox-storage/) covers these choices in more detail.

## 4. Where can work and data go?

Map the agent's connections. Its reviewer needs access to the preview; its tests need packages and perhaps a test database. If another sandbox runs browser tests, check that connection too.

[![Sandbox network access: authenticate inbound previews, control outbound destinations and protocols, and check whether connections between sandboxes are supported and allowed.](/images/posts/choosing-your-agent-sandbox/connections.svg)](/images/posts/choosing-your-agent-sandbox/connections.svg)

Upstash Box's public preview URLs have optional authentication. [[8]](#reference-8) Modal allows outbound connections to public IPs by default; its domain allowlist is **Beta** and covers TLS on port 443. [[9]](#reference-9) Check defaults and protocol coverage against your requirements.

Network access is one part of the decision. Also map where execution, orchestration, model inference, logs, and saved state live. Anthropic's **Beta** self-hosted Managed Agents runs tools on your infrastructure while sending tool inputs and outputs to its orchestration service, called the control plane. [[10]](#reference-10) Private network access and keeping all data in your account are separate requirements.

If several agents work together, define who owns each edit and how they hand off results. Upstash Boxes have no shared filesystem and cannot communicate directly, so coordination needs another route. [[11]](#reference-11) Shared storage still needs rules for conflicting edits and missing workers.

## Test the answers

For our bug-fixing agent, a starting requirement set might be:

- No production credentials or access to other workspaces; explicit CPU and memory limits.
- Run this repository's tests and preview, including its browser dependencies.
- Preserve unfinished edits overnight; recover after machine loss with at most five minutes of lost work.
- Authenticate the preview and allow only the destinations the task needs.

Run each candidate through the task, an overnight wait, and a burst at your expected concurrency. Interrupt a connection and terminate an environment. Check whether recovery repeats an external action, such as posting the same review comment twice.

Keep the model and acceptance test consistent. Compare completion rate, elapsed time, recovery effort, and total cost, including idle capacity, storage, failed attempts, and maintenance for systems you host yourself.

## References

1. <span id="reference-1"></span> [gVisor: What is gVisor?](https://gvisor.dev/docs/)
2. <span id="reference-2"></span> [Docker: Resource constraints](https://docs.docker.com/engine/containers/resource_constraints/)
3. <span id="reference-3"></span> [Cognition: Bringing macOS to Devin](https://devin.ai/blog/devin-gets-a-mac)
4. <span id="reference-4"></span> [Anthropic: How we contain Claude across products](https://www.anthropic.com/engineering/how-we-contain-claude)
5. <span id="reference-5"></span> [Blaxel: Sandboxes](https://blaxel.ai/platform/sandboxes)
6. <span id="reference-6"></span> [Freestyle: VM lifecycle](https://www.freestyle.sh/docs/vms/lifecycle)
7. <span id="reference-7"></span> [E2B: Sandbox persistence](https://docs.e2b.dev/sandbox/persistence)
8. <span id="reference-8"></span> [Upstash Box: Public URLs](https://upstash.com/docs/box/overall/preview)
9. <span id="reference-9"></span> [Modal: Networking and security](https://modal.com/docs/guide/sandbox-networking)
10. <span id="reference-10"></span> [Anthropic: Self-hosted sandboxes](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes)
11. <span id="reference-11"></span> [Upstash Box: Box basics](https://upstash.com/docs/box/overall/how-it-works)
