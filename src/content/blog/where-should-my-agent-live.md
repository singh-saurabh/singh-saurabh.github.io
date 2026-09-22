---
title: "Where should my Agent Live?"
date: 2026-09-22
tags: ["agents", "systems", "infrastructure"]
description: "From a coding agent on your laptop to a cloud service: why the session, harness, and execution environment need different homes."
---

The easiest way to put a coding agent in the cloud is to give it a computer. Start a VM, clone a repository, install Claude Code or Codex, and hand it a task. It looks a lot like running the same agent on your laptop.

That’s a sensible starting point. The agent can use the shell, edit files, start a server, and run tests in one place. But it also ties the agent's ability to make progress to the health of that machine.

I've been reading about how Anthropic, Cursor, OpenAI, and Cognition build cloud agents. Their designs increasingly separate the software managing the work from the environment doing the work. To see why, let’s start with what makes up an agent.

## The four pieces

I find it useful to break an agent into four pieces:

**Agent = Session + Harness + Tools + Environment**, with a language model providing the reasoning.

The model reads the context it receives and produces a response or a request to use a tool. The rest of the system turns those requests into work.

**The session is the history.** Your request, the agent's replies, tool calls, and their results belong here. If the agent ran a test and saw a failure, that observation becomes part of the session. The saved history can be larger than the context sent to the model on any one call.

**The harness runs the loop.** It prepares context, calls the model, routes tool requests, checks permissions, and feeds results back. It also handles practical details such as stopping a turn or summarizing old context. Claude Code and Codex provide this machinery around the model.

**Tools are the actions the agent can request.** Read a file. Apply a patch. Run a command. Open a browser. Call an API. Some tools act on the agent's machine; others reach an external service.

**The environment is where work happens.** For coding, this usually means a filesystem, installed dependencies, running processes, and network access. It might be your laptop, a container, or a VM. An agent that only calls remote APIs may not need a dedicated machine at all. OpenAI's architecture guide makes that option explicit. [[1]](#reference-1)

These are separate responsibilities, even when one application packages them together.

## What happens on your laptop

Suppose you open a repository and ask a coding agent to fix a failing login test.

A typical local session looks like this:

1. The harness sends your request and relevant context to the model.
2. The model requests a tool call, perhaps a command to run the test.
3. The harness checks the applicable permissions and runs the tool locally.
4. The test output goes into the session and back to the model.
5. The model requests further reads, edits, or tests until it can respond.

This is a simplified view of the loop described in Claude Code's documentation. [[2]](#reference-2) Codex CLI likewise works against your local repository and installed tools. [[3]](#reference-3)

[![A local coding agent: the model runs remotely, while the session, harness, and development tools are on the laptop.](/images/posts/where-should-my-agent-live/local-agent.svg)](/images/posts/where-should-my-agent-live/local-agent.svg)

*“Local agent” usually describes where the harness and tools run. With a hosted model, inference still happens remotely.*

The model does not directly open your files. A local tool reads them and returns the result. Similarly, when the model asks to run a test, a process on your machine does the execution.

Claude Code saves local conversation history so a session can be resumed. That history is separate from the repository: remembering that a command ran does not recreate the process it started. [[2]](#reference-2)

There can also be sandbox restrictions inside this local setup. Sharing a laptop does not mean every command has unrestricted access to it.

## The first cloud version is easy to picture

Move that setup into a VM. Give it a checkout, the right runtime, a database for tests, and an agent process. Expose the conversation through a web UI.

Here, the VM is the sandbox: the boundary containing the agent's execution. “VM” and “sandbox” describe different things, so a sandbox can be a full VM.

[![A cloud VM contains the harness, session files, tool runner, repository, dependencies, and services. The UI and model are outside it.](/images/posts/where-should-my-agent-live/colocated-cloud.svg)](/images/posts/where-should-my-agent-live/colocated-cloud.svg)

*The simplest deployment keeps the harness and its working environment on the same machine. This diagram also puts session storage there; that is a choice, not a requirement.*

If the harness reads `/workspace/app/package.json`, it gets the same file the shell sees. If it starts a development server, a local browser can reach it on `localhost`. The harness can track subprocesses and collect their output using ordinary operating-system interfaces.

You can also reuse an existing coding CLI. You don’t have to design a remote interface for every file operation and command first.

A prepared VM still needs a complete development environment. Cursor reports that missing tools or dependencies can show up as worse output rather than a clear error: the agent cannot properly execute or verify its work. [[4]](#reference-4)

For a short job, or an internal tool you can restart manually, this arrangement may be enough. The pressure to separate it grows when tasks run unattended and machines become replaceable.

## Where the coupling starts to hurt

Now, let's consider a few failure scenarios. These are consequences of the design, not claims that every provider has experienced each incident.

### The machine fails while the agent is working

Imagine a build runs out of memory and the platform destroys the VM. If the harness lived there too, the process deciding what to do next is gone. If the only session record was on its disposable disk, the history is gone as well.

With a session log stored elsewhere, another service can restart the harness. But that already separates the agent's lifetime from the machine's lifetime.

Startup can fail too. Anthropic describes this problem in Cowork: when its whole agent loop depended on the VM starting, a startup failure made the assistant unusable. Moving the loop outside let it respond and help diagnose the failure while code execution remained inside. That is a local product example of the same boundary problem. [[5]](#reference-5)

### The code can reach credentials it should not have

A VM isolates its contents from the host. It does not automatically isolate two processes inside it from each other.

If the harness puts a powerful credential in a place that generated code can read, that code may gain the harness's authority. Moving the harness out creates a useful separation, provided those credentials stay out too. A mounted secret or an unrestricted remote tool can undo the benefit.

Anthropic's security account describes keeping Cowork credentials in the host keychain and giving the VM a separately revocable, scoped token. This is a concrete authority boundary in addition to a machine boundary. [[5]](#reference-5)

### Every conversation depends on starting a machine

Suppose the next message only asks the agent to explain its previous change. If the harness is stopped along with the VM, the platform has to wake that environment before the agent can answer.

Keeping the harness available independently makes it possible to answer from the saved session and start execution only when a tool needs it. Whether that saves money or time depends on the workload and the platform's startup costs.

### The task needs more than one environment

A coding task might need a Linux build machine, a browser environment, and a service inside a customer's private network.

A harness inside one VM can still reach all three. The complication is ownership: should losing that one VM interrupt orchestration for every other environment? Should each new environment have to host another copy of the agent loop?

Once tools can reach multiple places, keeping the harness beside one particular workspace becomes less useful.

## How the cloud designs separate the pieces

The designs below separate the agent loop from the environment executing its commands, with different approaches to managing state.

[![Separated cloud architecture: durable session storage and a hosted harness sit outside the execution environment. The harness calls the model and sends tool requests to an executor inside a replaceable sandbox.](/images/posts/where-should-my-agent-live/separated-cloud.svg)](/images/posts/where-should-my-agent-live/separated-cloud.svg)

*The harness can still run in the cloud. It lives outside the sandbox that executes the task's code. The boxes show responsibilities and failure boundaries, not necessarily separate physical servers.*

### Anthropic: replace the harness or the sandbox independently

Anthropic's Managed Agents started with the session, harness, and execution together in a container. Its redesign moves the harness out and gives the session its own durable event log.

A failed sandbox becomes a tool error the harness can handle. A failed harness can be replaced and recover from saved events. Sandboxes are provisioned when needed, rather than before every session can begin. Anthropic reports lower first-response latency after this change. [[6]](#reference-6)

### Cursor: separate the loop, machine, and conversation

Cursor moved its cloud-agent loop into Temporal, a workflow system that records progress and supports recovery across worker failures. It also separated conversation storage and streaming from the loop.

That lets Cursor manage execution machines independently, including hibernating or replacing them. The conversation can remain available to clients as the underlying work is retried. This is a published implementation account, not a claim that every failed action can safely be repeated. [[4]](#reference-4)

### OpenAI: a hosted harness with an optional environment

OpenAI's Agents API exposes this separation directly. OpenAI runs the Codex harness and maintains the session. Your application submits work and receives events.

The execution environment can be OpenAI-hosted or supplied by you. If you supply it, you start the environment and connect an executor that runs the harness's requested commands. You own provisioning, reconnection, shutdown, and preserving files. For tasks that only need external tools, the dedicated environment can be omitted entirely. [[1]](#reference-1)

### Cognition: keep the agent loop in the cloud, choose the machine

Devin makes the boundary explicit through Outposts. Its agent loop stays in Devin Cloud, while commands, file edits, and repository operations run on a machine you control. That could be a VM in your private network, a GPU machine, or a Mac mini. A worker on that machine connects outward to Devin and carries out tool calls. [[7]](#reference-7)

The machine still needs the right capabilities. Cognition's hosted macOS environments run on AWS EC2 Mac hosts, with Xcode, the iOS Simulator, and desktop permissions prepared in advance. Consider an iPhone game that should remember your position after you quit. Writing Swift and getting a successful build cannot verify that behavior. Devin needs to play, quit, reopen the app, and check the result. It uses accessibility information and screenshots to interact with native apps. [[8]](#reference-8)

This is why separating the harness does not mean giving the agent a less capable computer. The environment can still be a full development VM, with the OS and tools the task requires.

These systems differ in implementation, but they share a direction: the agent loop does not have to live inside the machine executing the task's code.

## The separation has a cost

A local command is easy to picture: start a process and wait for its result. A remote command needs more rules.

Did it start before the connection dropped? Is it still running? Can a reconnect recover its output? If the harness retries, will it run twice? Which environment does a path belong to, and how does the browser reach the server?

The tool interface needs to answer those questions. An external harness can retain a shell, file tools, and browser control, but their behavior across the boundary must be explicit.

Keeping history also does not make every action replayable. A tool might update an external service and crash before reporting success. Blindly retrying can repeat the update. Temporal's documentation calls out this distinction and recommends idempotent activities: operations designed so repetition does not duplicate the effect. [[9]](#reference-9)

Finally, a replacement VM does not inherit the old one's files or running processes merely because the session remembers them. Workspace recovery needs its own storage and checkpointing design. OpenAI documents that even reusing an environment ID does not restore files on replacement compute. [[10]](#reference-10)

Cognition reports using full-machine snapshots to preserve memory, processes, and files while a session waits for CI or review. But that is not a promise shared by every Devin environment: its macOS article describes disk snapshots, while Cloudflare's Outposts integration archives selected directories. Recovering files and resuming a running process are different capabilities. [[11]](#reference-11) [[8]](#reference-8) [[12]](#reference-12)

## Conclusions

Local coding agents are still great for working hands-on with your code, using the tools and environment you already know. But it’s worth starting to move longer tasks to cloud agents such as Amp and Cursor. Amp’s Orbs and Cursor’s Projects can keep working after you close your laptop, and let you return to review the results. [[13]](#reference-13) [[14]](#reference-14)

The capabilities are expanding beyond one agent working on one task. Cursor has reported experiments with hundreds of concurrent agents, while Projects is designed to delegate work to thousands of subagents. Coordinating that work remains a challenge, but it no longer has to fit on one developer’s machine. [[15]](#reference-15) [[14]](#reference-14)

Devin follows the same shift toward delegated work. A coordinating session can assign tasks to other Devins, each with its own VM and test environment. Scheduled sessions can carry notes between runs and launch these workers for recurring work, such as a weekly QA pass. [[16]](#reference-16) [[17]](#reference-17)

The architectures from Anthropic, Cursor, OpenAI, and Cognition show where the industry is moving: durable session history, an independently managed harness, and execution environments that can be started, stopped, or replaced as the work requires. The harness lives **outside the execution sandbox**, even when both run in the cloud. [[6]](#reference-6) [[4]](#reference-4) [[1]](#reference-1) [[7]](#reference-7)

As cloud agents take on longer tasks and more parallel work, keeping your MacBook open just so an agent can continue is becoming less necessary. For work running in the cloud, you can already close the lid and come back when there’s something to review.

## References

1. <span id="reference-1"></span> [OpenAI: Agents API architecture](https://developers.openai.com/api/docs/guides/agents-api/architecture). The hosted harness, application server, and optional environment.
2. <span id="reference-2"></span> [Claude Code: How it works](https://code.claude.com/docs/en/how-claude-code-works). Local execution, sessions, tools, and the agent loop.
3. <span id="reference-3"></span> [OpenAI: Codex CLI](https://learn.chatgpt.com/docs/codex/cli). Working against a local repository and installed tools.
4. <span id="reference-4"></span> [Cursor: What we've learned building cloud agents](https://cursor.com/blog/cloud-agent-lessons). Development environments, durable execution, and conversation state.
5. <span id="reference-5"></span> [Anthropic: How we contain Claude](https://www.anthropic.com/engineering/how-we-contain-claude). VM boundaries, startup failures, and credentials.
6. <span id="reference-6"></span> [Anthropic: Scaling Managed Agents](https://www.anthropic.com/engineering/managed-agents). Separating session storage, the harness, and execution.
7. <span id="reference-7"></span> [Devin: Outposts overview](https://docs.devin.ai/cloud/outposts/overview). A cloud agent loop with execution on customer-controlled machines.
8. <span id="reference-8"></span> [Cognition: Bringing macOS to Devin](https://devin.ai/blog/devin-gets-a-mac). Mac VMs, disk snapshots, prepared tools, and native application verification.
9. <span id="reference-9"></span> [Temporal: Activity idempotency](https://docs.temporal.io/activity-definition#idempotency). Why durable execution still needs safe retries.
10. <span id="reference-10"></span> [OpenAI: Sandbox lifecycle](https://developers.openai.com/api/docs/guides/agents-api/environments/lifecycle). Reconnection, replacement compute, and cleanup.
11. <span id="reference-11"></span> [Cognition: What We Learned Building Cloud Agents](https://cognition.com/blog/what-we-learned-building-cloud-agents). Machine snapshots, asynchronous work, and VM orchestration.
12. <span id="reference-12"></span> [Cloudflare: Run Devin Outposts on Cloudflare](https://developers.cloudflare.com/sandbox/tutorials/devin-outposts/). Directory checkpoints and their recovery limits.
13. <span id="reference-13"></span> [Amp: Orbs overview](https://ampcode.com/docs/orbs). Remote environments that keep working while your laptop is closed.
14. <span id="reference-14"></span> [Cursor: Introducing Projects](https://cursor.com/blog/projects). Cloud execution, ongoing work, and delegation to thousands of subagents.
15. <span id="reference-15"></span> [Cursor: Scaling long-running autonomous coding](https://cursor.com/blog/scaling-agents). Experiments with hundreds of concurrent agents and the challenges of coordinating them.
16. <span id="reference-16"></span> [Cognition: Devin can now Manage Devins](https://cognition.com/blog/devin-can-now-manage-devins). Delegating tasks to separate sessions and VMs.
17. <span id="reference-17"></span> [Cognition: Devin can now Schedule Devins](https://cognition.com/blog/devin-can-now-schedule-devins). Recurring sessions, retained notes, and parallel workers.

*Diagrams are simplified illustrations, not vendor deployment schematics.*
