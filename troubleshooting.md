Problems resolved: 

1) dnsmasq failed due to port 53 --> stopped systemd-resolved which was occupying port 53, in /etc/dnsmasq.conf, changed port = 0

2) Temporary failure in Name Resolution (DNS) --> manually set nameserver 8.8.8.8 in /etc/resolv.conf for client1-VM and client2-VM

3) Ping Issues --> Checked firewall rules, ensured routing enabled, NAT working 
