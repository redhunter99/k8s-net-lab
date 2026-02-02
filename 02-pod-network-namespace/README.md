## Pod Network Namespace & Veth

Steps:
1. kubectl apply -f pinger.yaml
2. kubectl apply -f app.yaml
3. ip link | grep cali
4. ip netns list
5. ip netns exec <netns> ip addr
6. tcpdump -i caliXXXX icmp
