---
title: "The storage story: what your agent sandbox gives you"
date: 2026-09-23
tags: ["agents", "systems", "infrastructure", "storage"]
description: "A sandbox gives your agent a filesystem. What survives sleep, replacement, or a second agent depends on a separate storage contract."
coverImage:
  src: "/images/social/agent-sandbox-storage.jpg"
  alt: "Stacked translucent layers around a blue core"
---

A bug-fixing agent edits three files, installs a package, runs tests, and starts a preview server. It waits overnight while you review the change. What will still be there tomorrow?

“The sandbox has a filesystem” doesn't answer that. The conversation might remember the work while a replacement machine has none of it. **Check which bytes survive which events, and how another machine gets them back.** This is the storage contract introduced in [Choosing your agent sandbox](/blog/choosing-your-agent-sandbox/).

## Five ways to keep state

From inside a shell, `/workspace/report.md` is just a path. Underneath, several arrangements are possible:

| Arrangement | What it gives you | What to check |
| --- | --- | --- |
| Scratch filesystem | Working files for this instance | When its writes disappear |
| Bundled persistent filesystem | A workspace restored by the platform | Which lifecycle transitions it survives |
| Attachable volume | Storage with a lifetime separate from compute | Where it attaches and who can write |
| Snapshot or image | A saved state for reconstruction | What it captures and how long it remains usable |
| External object storage, Git, or database | State published outside the sandbox | What gets published and how recovery uses it |

These can coexist: boot from an image, build in scratch space, edit on a volume, and upload the result. A dependency cache and the only copy of an unfinished patch deserve different treatment.

[![Local sandbox files, an optional durable volume, and explicitly published external state have separate storage contracts.](/images/posts/agent-sandbox-storage/storage-layers.svg)](/images/posts/agent-sandbox-storage/storage-layers.svg)

## Local files can disappear

Docker's writable container layer disappears when the container is destroyed; a Docker volume survives container removal. The path alone doesn't reveal that distinction. [[1]](#reference-1)

Cloudflare's Sandbox SDK starts a fresh container after an idle stop. Local files and processes are lost even if you reuse the sandbox ID. A patch saved only in `/workspace` disappears with them. [[2]](#reference-2)

Cloudflare offers directory backups to R2 object storage with an explicit restore operation. Your application must arrange those backups. Backing up `/workspace` won't capture a package installed elsewhere, or an edit made after the backup. [[3]](#reference-3)

## Persistent files don't guarantee running processes

Sprites preserves files, packages, and repositories when a stopped sandbox starts again, continuously syncing its filesystem to durable object storage. No separately attached volume is needed. [[4]](#reference-4)

Our agent's edits can survive, but its preview server needs to restart. Sprites Services restart managed processes; a manually launched process needs something else to bring it back. Persistence also keeps accidental changes, so a durable workspace still benefits from checkpoints. [[4]](#reference-4)

## Volumes have different recovery boundaries

Tensorlake Cloud Volumes can mount across sandbox providers, cloud machines, and local Linux or macOS machines. Writes are published asynchronously: another machine recovers through the last autosave checkpoint. Later writes may still exist only on the original machine. [[5]](#reference-5)

[![Example of asynchronous checkpoint recovery: patch A reaches a confirmed checkpoint, patch B remains local, and a replacement machine recovers only A unless B was saved separately.](/images/posts/agent-sandbox-storage/checkpoint-recovery.svg)](/images/posts/agent-sandbox-storage/checkpoint-recovery.svg)

Fly Volumes have a different placement contract. Each lives on one physical server and attaches to one Machine at a time. It can outlive that Machine, but Fly doesn't automatically replicate data between volumes. Surviving a compute restart doesn't establish protection against the storage device failing. [[6]](#reference-6)

## Check what a snapshot captures

A snapshot might capture a directory, filesystem, volume, or running machine. Check its scope, timing, and retention.

Cloudflare rejects directory backups after their restore deadline passes, even if the underlying R2 objects remain. Tensorlake distinguishes recent autosave recovery points from permanent snapshots retained until deletion. [[3]](#reference-3) [[7]](#reference-7)

Memory has its own rules. Fly's suspend operation saves CPU and memory state separately from volumes. Deployments or host migration can discard that execution state and force a cold start while volume data remains. Recovery must handle files returning without their programs still running. [[8]](#reference-8)

## Shared storage needs an editing policy

First establish whether anything is shared. In Claude Managed Agents, reusing an environment ID reuses configuration; each cloud session gets a fresh isolated container. [[9]](#reference-9)

On Tensorlake shared mounts, changes to different paths merge. Overlapping writes to the same file use last-writer-wins behavior without conflict markers. Other mounts see published changes as they converge. [[7]](#reference-7)

For parallel repository edits, give agents separate checkouts or Git worktrees, each on its own branch, then integrate their work explicitly. Branches alone do not separate working files. [[10]](#reference-10) Direct edits to shared state need ownership or locking rules.

Restrict mounts too: a dataset reader may only need read access to one directory. Cloudflare bucket mounts support selected prefixes and read-only access. [[11]](#reference-11)

## Test recovery on the next machine

For the bug-fixing agent, keep enough configuration to rebuild tools and restart services, preserve unfinished edits in a durable workspace, and publish completed results outside disposable compute.

A local Git commit still lives on local disk until copied or pushed elsewhere. Git's remote workflow makes publication an explicit step. [[12]](#reference-12) An artifact upload similarly preserves the selected artifact, not the whole environment.

Storage can also keep costing money after compute stops. Sprites charges for stored bytes while idle; Fly bills volumes while their Machines are stopped or suspended. [[4]](#reference-4) [[8]](#reference-8)

Before relying on a storage setup, save an edit, stop or replace the environment, and check files and processes separately. Repeat from a second machine and record which checkpoint it recovered. That tells you what work the agent can safely leave behind.

## References

1. <span id="reference-1"></span> [Docker: Storage](https://docs.docker.com/engine/storage/)
2. <span id="reference-2"></span> [Cloudflare: Sandbox lifecycle](https://developers.cloudflare.com/sandbox/concepts/sandboxes/)
3. <span id="reference-3"></span> [Cloudflare: Backup and restore](https://developers.cloudflare.com/sandbox/guides/backup-restore/)
4. <span id="reference-4"></span> [Sprites: Lifecycle and Persistence](https://docs.sprites.dev/concepts/lifecycle/)
5. <span id="reference-5"></span> [Tensorlake: Cloud Volumes](https://docs.tensorlake.ai/filesystems/introduction)
6. <span id="reference-6"></span> [Fly.io: Fly Volumes overview](https://fly.io/docs/volumes/overview/)
7. <span id="reference-7"></span> [Tensorlake: Core Concepts](https://docs.tensorlake.ai/filesystems/core-concepts)
8. <span id="reference-8"></span> [Fly.io: Machine Suspend and Resume](https://fly.io/docs/reference/suspend-resume/)
9. <span id="reference-9"></span> [Anthropic: Cloud environment setup](https://platform.claude.com/docs/en/managed-agents/environments)
10. <span id="reference-10"></span> [Git: git-worktree](https://git-scm.com/docs/git-worktree)
11. <span id="reference-11"></span> [Cloudflare: Mount buckets](https://developers.cloudflare.com/sandbox/guides/mount-buckets/)
12. <span id="reference-12"></span> [Git: Working with Remotes](https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes)
