## Pod to Pod (Different Node)

Expected flow:
eth0 -> veth -> iptables -> tunl0 (IPIP) -> tunl0 -> veth -> eth0

Tools:
tcpdump -i tunl0
ip route
iptables -L FORWARD -v
