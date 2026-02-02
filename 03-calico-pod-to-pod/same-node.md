## Pod to Pod (Same Node)

Observed flow:
eth0 -> veth -> kernel(FORWARD) -> veth -> eth0

Proof:
- tcpdump on cali interface
- TTL reduced by 1
- tunl0 unused
