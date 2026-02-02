# Pod Network Namespace & veth – Deep Dive (Hands-on)

This lab explains **how a Kubernetes Pod gets network connectivity**
using **Linux network namespaces and veth pairs**.

No assumptions.
Every concept is verified using real Linux commands.

---

## 🎯 Objective

- Understand **what happens when a pod runs**
- Identify the **pause container**
- Inspect **pod network namespace**
- Map **pod eth0 ↔ node veth**
- Understand how CNI (Calico) wires networking

---

## 🧪 Prerequisites

- Kubernetes cluster running
- Calico installed as CNI
- Namespace `test` exists

```bash
kubectl create ns test


📦 Pod Manifests Used
pinger.yaml and app.yaml
kubectl apply -f pinger.yaml
kubectl apply -f app.yaml


Verify pods:

kubectl get pods -n test -o wide

🧠 Step 1: What happens when a Pod runs?

When a Pod is created:

Kubernetes first creates a pause container

Pause container holds:

network namespace

IPC namespace

UTS namespace

All containers in the pod share this namespace

🔍 Step 2: Identify the Pause Container

Create a sample pod:

kubectl run nginx --image=nginx


List namespaces:

lsns | grep nginx


Copy the PID and inspect it:

lsns -p <PID>


This confirms:

Pause container owns the network namespace

Application containers join it

🔍 Step 3: List All Network Namespaces
ip netns list


OR:

ls -lt /var/run/netns


Each entry represents one pod network namespace.

🔍 Step 4: Inspect Pod Network Namespace

Replace <namespace> with actual netns ID:

ip netns exec <namespace> ip link


You will see:

lo

eth0@ifX

📌 Meaning:

eth0 → pod interface

ifX → peer interface index on the node

🔍 Step 5: Inspect Pod Interfaces from Inside the Pod
kubectl exec -it pinger -n test -- ip addr


Observe:

Pod IP is /32

Interface shown as eth0@ifX

🔁 Step 6: Map Pod eth0 to Node veth

From the node:

ip link show


Search using interface index:

ip link | grep -A1 ^X:


You will find something like:

caliXXXX@ifY
link-netns <netns-id>


✅ This confirms:

Pod eth0  <---- veth pair ---->  caliXXXX (node)

🔍 Step 7: List All Calico veth Interfaces
ip link show | grep cali


Each caliXXXX represents one pod attached to the node.

🔬 Step 8: Verify veth Pair Connectivity
ethtool -S caliXXXX


Look for:

peer_ifindex


This proves:

Both ends belong to the same veth pair

🧠 Mental Model (IMPORTANT)
[ Pod Network Namespace ]
eth0@ifX
     |
     |  veth pair
     |
[ Node Network Namespace ]
caliXXXX


Pod traffic exits via eth0

Enters node via caliXXXX

Linux kernel handles forwarding

📌 Useful Commands Reference
ip link show
ip netns list
lsns
lsns -p <pid>
ls -lt /var/run/netns
ip netns exec <netns> ip link
kubectl exec -it <pod> -- ip addr
ip link | grep cali
ethtool -S caliXXXX

✅ What We Have Proven

Pods run inside isolated network namespaces

Pause container owns the namespace

Pod eth0 is connected to node using veth pairs

Calico creates caliXXXX interfaces on the node

Pod networking is pure Linux kernel networking


