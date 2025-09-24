<p align="center">
  <img src="https://i.postimg.cc/QdV16pcB/IMG-4837.jpg" alt="VANTA Banner" width="90%" />
</p>

<h2 align="center"><strong>VANTA Architecture LLD - Three-Node Deployed Brain</strong></h2>
<p align="center"><em>Authoritative low-level design for the orchestrated Alpha / Markets / Executor topology - redacted for external sharing.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/Status-Architecture%20Spec-blue" />
  <img src="https://img.shields.io/badge/Security-Redacted%20%26%20Safe-green" />
  <img src="https://img.shields.io/badge/Version-LLD%20v1.0-purple" />
  <img src="https://img.shields.io/badge/License-All%20Rights%20Reserved-red" />
</p>

<p align="center">
  <a href="https://github.com/qstackfield/vanta-capital-intelligence-os"><b>VANTA OS (Core)</b></a> ·
  <a href="https://github.com/qstackfield/vanta-platform"><b>VANTA Platform (Mirroring)</b></a> ·
  <a href="https://qstackfield.github.io/vanta-capital-intelligence-os/"><b>Investor Overview</b></a> ·
  <a href="https://github.com/qstackfield/vanta-capital-intelligence-os/discussions"><b>Discussions</b></a>
</p>

---

> ⚠️ **Read First — Scope & Safety**
>
> This document describes **how VANTA is deployed and operated** (topology, storage, orchestration, controls) without exposing secrets, credentials, or proprietary algorithms.  
> Paaths and module names are **illustrative** and may be renamed in production deployments.

## 📌 Executive Summary

The **VANTA Three-Node Brain** is a **production-grade, multi-node capital intelligence system** designed for resiliency, auditability, and scale.  
It separates responsibilities into three dedicated roles:

- **Alpha** → Orchestrator & Control Plane (truth source, trackers, operator tools)  
- **Markets** → High-throughput ingestion & reflection (signals, conviction, reason vectors)  
- **Executor** → Deterministic trade routing, vault enforcement, broker adapters  

This separation ensures:
- **Security** → No single node carries full state.  
- **Replayability** → Any decision path can be reconstructed.  
- **Scale** → Collector and executor capacity grows independently.  
- **Governance** → Vault overlays, personas, and flip-modes constrain execution.  

---

## 🎯 Scope

This LLD documents the **operational architecture** of VANTA across its three nodes:
- Topology & roles  
- Filesystem layout  
- Data flows (harvest → reflect → queue → execute → audit)  
- Personas & overlays (where applied)  
- Observability, risk controls, and operator UX  

---

## 🚫 Non-Scope

Not included in this document:
- Proprietary model architectures (classifiers, forecasters, embeddings)  
- Prompt templates or inference logic  
- Live broker credentials, API keys, or tenant-specific configs  
- Platform-side subscription & mirroring (documented in **VANTA Platform LLD**)  

---

## 🗺️ Topology (Authoritative)

The VANTA brain is deployed as a **three-node distributed system**. Each node has a precise, non-overlapping role, ensuring separation of duties and fault tolerance.

### Node A — **Blackglass-Alpha (Orchestrator / Brain)**
- Acts as the **control plane** and authoritative configuration source.  
- Hosts the **assistant, tracker, and diagnostics suite** used by operators.  
- Maintains the **registry of vaults, overlays, and thread metadata**.  
- Exports a shared `/opt/vanta/build` directory via NFS to Markets and Executor.  

### Node B — **Vanta-Markets (Harvest & Reflection)**
- Handles **high-throughput collection and reflection** of signals: Reddit, Twitter, SEC, news, crypto, options flow.  
- Normalizes, deduplicates, and aligns raw feeds.  
- Computes **reason vectors, conviction scores, and market overlays**.  
- Produces authoritative artifacts like `trade_signals.json` and reflection logs.  

### Node C — **Vanta-Executor (Execution Plane)**
- Consumes **autotrade_queue.json** as the canonical intent stream.  
- Routes orders through **risk fences, vault overlays, and persona constraints**.  
- Interfaces with brokers/exchanges via pluggable adapters (Alpaca, Tradier, Coinbase, IBKR, Bybit).  
- Maintains **open_orders.json** and append-only `trade_log.jsonl` for replay.  

---

### Cross-Node Principles
- **Separation of Concerns:** no single node contains ingestion + reflection + execution.  
- **Shared Utilities:** Alpha distributes assistants/tools via NFS, ensuring all nodes run consistent binaries.  
- **Local State:** each node owns its `/opt/vanta/memory` and `/opt/vanta/logs` directories; nothing critical is centralized.  
- **Governance Layer:** vault overlays and persona biases flow across nodes but remain deterministic.  
- **Auditability:** append-only logs exist per node; together, they reconstruct every decision path.  

---