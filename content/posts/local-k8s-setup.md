+++
title = "Local K8s Setup"
date = 2026-04-26T12:30:36-05:00
categories = ["dev-ops", "homelab", "kubernetes"]
+++

## Why local Kubernetes?

Kubernetes has been orbiting my work for years—but I’ve never really *needed* it.

Most of the platforms I support are either legacy-heavy or simple enough that Docker Compose gets the job done. Even when I wanted something more “cluster-like,” Docker Swarm was usually sufficient.

But I kept running into the same friction:

I didn’t have a unified platform for running *everything* locally.

Different tools, different configs, different deployment patterns.

So I finally decided to fix that—with a local Kubernetes setup using k3s.

---

## Who this is for

This setup makes sense if:

- You want a realistic Kubernetes environment locally
- You’re experimenting with GitOps workflows
- You want internal services with proper TLS (without public exposure)
- You’re okay trading “perfect production architecture” for simplicity

---

## What I started with

A few months ago, I switched my desktop from Windows 11 to Fedora 43.

(As it turns out, Factorio and RimWorld both run natively on Linux—so there wasn’t much keeping me on Windows.)

Since I’m already running Linux, I decided to use my desktop as the primary Kubernetes node. This is a homelab, so if it goes down for 15 minutes during upgrades, it’s not a big deal.

The hardware is also well-suited for this:

- 128 GB RAM  
- AMD Ryzen 7 (8 cores / 16 threads with SMT)

---

## K3s vs Minikube

My first attempt used `minikube`, which was incredibly easy to get running.

The issue I ran into was networking.

Because it runs through a VM (QEMU in my case), there’s an extra layer of indirection. IPs weren’t where I expected them to be, and exposing services cleanly took more effort than I wanted.

I switched to `k3s`, which runs directly on the host.

That gave me:

- A more “real” cluster feel
- Simpler networking
- A path to multi-node setups later

That tradeoff—slightly more setup for much more realism—was worth it.

---

## Mullvad vs k3s (the unexpected fight)

One thing I didn’t anticipate: `k3s` did not get along with Mullvad VPN.

I run Mullvad on my desktop for privacy, and it introduced routing conflicts with the pod network.

After some trial and error, I landed on a workable setup:

- Pod traffic bypasses the VPN tunnel
- Host traffic still goes through Mullvad

---

## High-level design

The goal is simple:

Run internal services under `*.home.bearl.me` with no public exposure.

Access requires:

- Being on my LAN  
- Or connecting through VPN  

{{<mermaid>}}
flowchart LR
    User[User Device] -->|LAN or VPN| K8sNode[Local k3s Node]

    subgraph Kubernetes Cluster
        ArgoCD[ArgoCD]
        StepCA[Step CA]
        Apps[Internal Apps]
    end

    K8sNode --> ArgoCD
    K8sNode --> StepCA
    K8sNode --> Apps

    StepCA -->|ACME / Certs| Apps
{{</mermaid>}}

---

## Why Step CA?

The first real workload I wanted was Step CA.

I’ve been wanting a clean way to:

- Issue internal certificates  
- Use ACME for automatic renewal  
- Avoid self-signed cert chaos  

---

## Overall configuration

https://github.com/lbearl/k8s

1. Core Kubernetes setup (k3s)  
2. ArgoCD  
3. Step CA  
4. Backup jobs  

{{<mermaid>}}
flowchart LR
    GitRepo[Git Repository] --> ArgoCD
    ArgoCD --> Apps[Kubernetes Apps]
    Apps --> ClusterState[Cluster State]

    ArgoCD -->|Sync Loop| ClusterState
{{</mermaid>}}

---

## Gotchas

- Minikube networking complexity  
- VPN conflicts  
- Single-node limitations (but I'm curious to see how `k3s` handles multi arch hosts in a heterogenous cluster) 

---

## Next steps

Now that ArgoCD is in place, everything is Git-driven.

The long-term goal is something that feels like Heroku:

- Commit config  
- Push code  
- Watch it deploy  

---

## Final thoughts

This setup isn’t production-grade—and that’s intentional.

It’s a playground for learning Kubernetes properly and building a consistent deployment model.