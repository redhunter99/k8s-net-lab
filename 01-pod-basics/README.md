## Pod Basics & Shared Namespace

Topics:
- What happens when a pod runs
- Pause container
- Shared network namespace
- eth0@ifX and veth pairs

Commands:
ip netns list
lsns
lsns -p <pid>
kubectl exec -it shared-namespace -- ip addr
