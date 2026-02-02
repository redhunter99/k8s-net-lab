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

This lab uses two Pods, both scheduled on the same node to study networking behavior.

pinger.yaml

app.yaml

Apply the manifests:

kubectl apply -f pinger.yaml
kubectl apply -f app.yaml


Verify Pod status and node placement:

kubectl get pods -n test -o wide


This confirms:

Pods are running

Pods are attached to the expected node

🧠 Step 1: What Happens When a Pod Runs?

When a Pod is created, Kubernetes performs the following:

Kubernetes first creates a pause container

The pause container owns:

Network namespace

IPC namespace

UTS namespace

Application containers join the pause container’s namespaces

Why this matters:

All containers in a Pod share the same IP

All containers share the same network interfaces

Pod networking is namespace-based, not container-based

🔍 Step 2: Identify the Pause Container

Create a temporary Pod:

kubectl run nginx --image=nginx


List namespaces and locate the pause container:

lsns | grep nginx


Copy the PID and inspect it:

lsns -p <PID>

What this proves:

Pause container owns the network namespace

Application containers attach to it

Pod networking is NOT container-specific

🔍 Step 3: List All Network Namespaces on the Node

List namespaces using either method:

ip netns list


OR:

ls -lt /var/run/netns


Each entry corresponds to one Pod network namespace created by the CNI plugin.

🔍 Step 4: Enter a Pod Network Namespace from the Node

Replace <namespace-id> with an actual namespace ID:

ip netns exec <namespace-id> ip link


Expected output:

lo
eth0@ifX

Meaning:

eth0 → Pod network interface

ifX → peer interface index on the Node

👉 This proves the Pod is connected via a veth pair, not directly to the NIC.

🔍 Step 5: Inspect Network Interfaces from Inside the Pod

Run inside the Pod:

kubectl exec -it pinger -n test -- ip addr


Observe:

Pod IP assigned as /32

Interface shown as eth0@ifX

👉 Pod routing is handled by the Node.

🔁 Step 6: Map Pod eth0 to Node veth Interface

From the Node, list interfaces:

ip link show


Search using the interface index from eth0@ifX:

ip link | grep -A1 ^X:


You will find something like:

caliXXXX@ifY
link-netns <netns-id>

This confirms:
Pod eth0  <---- veth pair ---->  caliXXXX (Node)

🔍 Step 7: List All Calico veth Interfaces
ip link show | grep cali


Each caliXXXX interface represents one Pod attached to the Node.

🔬 Step 8: Verify veth Pair Connectivity

Verify the veth pair using ethtool:

ethtool -S caliXXXX


Look for:

peer_ifindex

This proves:

Both ends belong to the same veth pair

Pod ↔ Node connectivity is direct

🧠 Mental Model (IMPORTANT)
[ Pod Network Namespace ]
eth0@ifX
     |
     |  veth pair
     |
[ Node Network Namespace ]
caliXXXX


Pod traffic exits via eth0

Enters the Node via caliXXXX

Linux kernel handles forwarding

📌 All Commands Used in This Lab (Single Place)
kubectl create ns test
kubectl apply -f pinger.yaml
kubectl apply -f app.yaml
kubectl get pods -n test -o wide

kubectl run nginx --image=nginx

lsns
lsns -p <pid>

ip netns list
ls -lt /var/run/netns
ip netns exec <netns> ip link

kubectl exec -it <pod> -- ip addr

ip link show
ip link | grep cali
ip link | grep -A1 ^X:

ethtool -S caliXXXX

✅ What We Have Proven in This Lab

Pods run inside isolated network namespaces

Pause container owns the namespace

Pod eth0 is one end of a veth pair

Node caliXXXX is the other end

Calico uses Linux kernel networking

Pod networking is NOT Kubernetes magic



