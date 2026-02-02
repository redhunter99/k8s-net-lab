# Kubernetes Networking Labs – From Scratch (Calico Focus)

This repository documents a **hands-on, kernel-level exploration of Kubernetes networking**.
Every concept here is validated using real Linux commands (`ip`, `lsns`, `tcpdump`, `iptables`)
— no theory-only explanations.

This repo is built step-by-step while learning how **pod networking actually works under the hood**.

---

## 🔥 What You Will Learn From This Repo

- What happens when a Pod runs
- Pause container & shared namespaces
- Linux network namespaces (`netns`)
- veth pairs (Pod ↔ Node)
- Pod → Pod traffic flow (Same Node)
- How Calico uses **iptables + Linux routing**
- How to PROVE packet flow using `tcpdump`

---

## 🧪 Lab Environment

- Kubernetes (kubeadm based)
- Nodes:
  - 1 Control Plane (schedulable)
  - 1 Worker
- CNI: **Calico (IPIP mode)**
- OS: Linux
- Container Runtime: containerd

---

## 📁 Repository Structure


