Network Configuration Project

This project involves setting up a virtual network with firewalls, DHCP, NAT and DNS configuration on a Linux Server using Oracle VirtualBox

Steps involved: 

1) Set up the VM's (firewall-VM, Client1-VM, Client2-VM)
2) Configure DHCP with 'dnsmasq' on firewall-VM
3) Set up NAT and Firewall rules
4) Ensure DNS resolution works for Client1 and Client2
5) Test for network Connectivity

Technologies Used:
1) Linux (Ubuntu Server 24.04)
2) Oracle VirtualBox
3) Networking principles (DHCP, NAT, IP Forwarding)
4) dnsmasq (DHCP Server)
5) iptables (firewall, NAT rules)

=======================================================================================================

STEP BY STEP process: 
----------------------
Need 3 Virtual Machines i.e., firewall-VM, Client1-VM, Client2-VM

firewall-VM ---> controls access between networks, Acts like Router + Firewall

Client1-VM ---> simulates a private server (like office computer), Acts like a PC connected to LAN

Client2-VM ---> another private server (fire server, database), Acts like a PC connected to LAN


NOTE: While Installation of Linux on all these VM's, Select Language, Keyboard, Proxy, Mirror, File partition [default], Server name should be your VM's name, set up a username and password. If installation goes well, username@VM-name will pop up on command line, then you are good to go.


Turn off all the VM's. 

For firewall-VM, goto settings--> Network --> Adapter-1 --> Enable NAT, then goto Adapter-2 --> Enable network --> on dropdown look for Internal NEtwork, then name intnet. Click OK

For Client1-VM and Client2-VM, go to Settings --> Network --> Adapter-1 --> Enable Network Adapter --> from dropdown --> look for Internal Network --> name it intnet --> Click OK

=======================================================================================================

NETWORK CONFIGURATION FOR firewall-VM:
--------------------------------------
After login, 

=> sudo apt update

=> ip a 
this gives the static ID or network interface like enp0s3 and enp0s8. As enp0s3 is set to NAT, we don't need to change it. We have to configure enp0s8 as it is a internal network

=> sudo nano /etc/netplan/00-installer-config.yaml

network:
  version:  2
  ethernets:
    enp0s3:
      dhcp4:  true
    enp0s8:
      dhcp4:  false
      addresses:
        -  192.168.10.1/24

Save this and Exit nano editor

=> sudo netplan apply 

to apply this configuration.

=> ip a 

this now should show enp0s8 with inet 192.168.10.1/24 and enp0s3 with dynamic static ID

=> sudo apt install ufw

=> sudo ufw status
if inactive

=> sudo ufw enable
This enables the ufw (uncomplicated firewall) status -> active
------------------------------------------------------------------------------------------------------------

NETWORK CONFIGURATION FOR Client1-VM and Client2-VM:
----------------------------------------------------

=> ip a 
Check for enp0s3 static ID

configure it
=> sudo nano /etc/netplan/00-installer-config.yaml

in Nano editor,

network:
  version:  2
  ethernets:
    enp0s3:
      dhcp4:  false
      addresses:
        -  192.168.10.2/24

save it and exit

=> sudo netplan apply
to apply this configuration

=> ip a
check for enp0s3 192.168.10.2/24 
--------------------------------------------------------
For Client3-VM, keep IP as 192.168.10.3/24 and repeat the above
-----------------------------------------------------------------------------------------------------------
NOTE: For Firewall-VM, enp0s3 and enp0s8 Static ID's should be different. 

Once all the virtual machines are connected, check for ping.

On firewall-VM, check ping 192.168.10.2, ping 192.168.10.3
ON Client1-VM, check ping 192.168.10.3, ping 192.168.10.1
On Client3-VM, check ping 192.168.10.1, ping 192.168.10.2

if All are connected, proceed further. 
-----------------------------------------------------------------------------------------------------------

On firewall-VM, install and configure dnsmasq

=> sudo apt update
=> sudo apt install dnsmasq

Configure DHCP

=> sudo nano /etc/dnsmasq.conf

Save it and restart dnsmasq

=> sudo systemctl restart dnsmasq

dnsmasq acts as lightweight DHCP server & automatically gives IP adresses to Client VM's (no manual IP setup)
-------------------------------------------------------------------------------------------------------------
ENABLING IP FORWARDING (ALLOW ROUTING)

for temporary change: 
=> sudo sysctl -w net.ipv4.ip_forward=1

for permanent change:
=> sudo nano /etc/sysctl.conf

UNCOMMENT: 
net.ipv4.ip_forward=1
-------------------------------------------------------------------------------------------------------------
SET UP NAT (iptables, Network Address translation)

=> sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

-> this hides all internal client IP's behind firewall-VM's public IP. Its called Network Address Translation (NAT) like home wifi

Now we need to save those ip rules

=> sudo apt install iptables-persistent
=> sudo netfilter-persistent save
--------------------------------------------------------------------------------------------------------------
Check if ufw is installed or not, check ufw status

=> sudo ufw status

if active, disable ufw by => sudo ufw disable

Now, we need to allow internal traffic in and allow traffic out to the internet

=> sudo ufw allow in on enp0s8
=> sudo ufw allow out on enp0s3

=> sudo ufw enable

-------------------------------------------------------------------------------------------------------------

Client1-VM & Client2-VM Setup:

Set to DHCP

=> sudo nano /etc/netplan/*.yaml

network:
  version:  2
  ethernets:
    enp0s3:
      dhcp4:  true

Save it and exit.

=> sudo netplan apply

Also, 

=> sudo nano /etc/resolv.conf

nameserver 8.8.8.8

save it and exit
-------------------------------------------------------------------------------------------------------------

Test connectivity on both the client VM's:

=>ping 192.168.10.1
=> ping 8.8.8.8
=> ping google.com

if you are able to ping everything, then all your virtual machines are vitually connected.
-------------------------------------------------------------------------------------------------------------
