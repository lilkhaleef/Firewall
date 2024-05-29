# Firewall

# Creating a firewall using virtual machines in a home lab

# Network Setup
<img width="355" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/e7c731eb-9ee5-4d11-923f-f348a45157fa">


Step 1: To set up the firewall, 3 virtual machines and networks was set up on VMware. First step was to create a Kali Linux VM and clone it twice to have 3 virtual machines (Kali Linux, Clone 1 and Clone 2). 

 <img width="308" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/5e0124fe-b22a-4e06-a065-de433d371b64">

Figure 1: Creating three virtual machines.

Step 2: Also, NAT network adapter and host-only adapter was used to simulate the network. The adapter 1 for the first VM (Kali Linux) was configured to be attached to the NAT network, adapter 1 for the second VM (Clone 1) to be attached to the NAT network, adapter 2 for the second VM to be attached to the Host-Only adapter and the adapter 1 for the third VM (Clone 2) to be attached to the Host-Only adapter.


Step 3: After this, the IP addresses of the VMs was checked, where the first and third VM has one IP address due to single configuration of adapter while the second VM has two IP addresses due to having two adapter configurations (NAT network adapter and Host-only adapter) together to ensure the network was set up well.


Step 4: IP forwarding was enabled on the second VM (which was used as the firewall) for the VM to be able to forward traffic from one network interface to another (from the first VM to the third VM). 
This was done by running the command “echo 1 > /proc/sys/net/ipv4/ip_forward”. After enabling IP forwarding, the command “cat /proc/sys/net/ipv4/ip_forward” was ran which displayed 1. This is to ensure that IP forwarding is being enabled on the VM.  

<img width="412" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/26e4f756-1d28-4d65-bd37-9fdf63de1363">
 
Figure 2: Enabling IP forwarding on the second VM.

Step 5: NAT (Network Address Translation) was used to change the source IP of the packets going through the firewall, so that the destination knows where to send the response (to the firewall) using the command “ iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE ”.

In the given command, the iptables tool was used to configure NAT on the firewall. Specifically, the "-t nat" option was used to specify modification of the NAT table, and the "-A POSTROUTING" option to add a rule to the "POSTROUTING" chain, which is used for packets that are leaving the firewall.
The "-o eth0" option specifies the outgoing network interface, which is the interface that connects to the external network. Finally, the "-j MASQUERADE" option tells iptables to use a special form of NAT called "MASQUERADE" to modify the source IP address of outgoing packets to match the IP address of the firewall's outgoing interface. This allows the response from the destination to be sent back to the firewall, which can then forward it to the correct device on the internal network. The“ iptables -t nat -L” command was used to list the NAT rules active on the VM.

 <img width="370" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/96f7ee1c-52a5-4b6b-99c5-094451e26381">

Figure 3: Using NAT to change the source IP of the packets going through the firewall.

Step 6: After then, ping the first VM from the third VM using the command “ping 192.168.241.129” which shows network was unreachable because there is no route and first VM is not in the same network as third VM.

<img width="278" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/5212931d-2066-4f19-b7dc-2b5a21462248">
 
Figure 4: Pinging first VM from the third VM.

The third VM is unable to connect to the first VM directly because they are on different networks. However, it is possible to use the second VM as an intermediary to connect the third VM to the first VM.
To accomplish this, the network configuration must be adjusted so that the second VM is able to route traffic between the two networks.

Step 7: The command "ip route add default via 192.168.241.128" was used on the second VM to add a default route that tells the system to send all packets to the gateway with the IP address of 192.168.241.128. This gateway could be the router or firewall that connects the two networks.
After this route was added, the second VM is able to act as a router for traffic between the two networks. The third vm can send packets to the second VM, which will then forward them to the first VM. The first VM can respond by sending packets to the second VM, which will then forward them to the third vm.
This configuration allows the third vm to connect to the first VM indirectly through the second VM by adding a default route on the second VM that enables it to route traffic between the two networks.

 <img width="375" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/f8e9c32a-563a-4647-9099-6f3df3928405">

Figure 5: Adding route from the third VM to first VM through the second VM.

Step 8: A firewall rule was created on the second VM to deny ICMP messages using the command “ iptables -A FORWARD -p icmp --icmp-type echo-request -j DROP ”.
The command is used to configure the iptables firewall to drop incoming ICMP Echo Request packets that are forwarded between two different networks.
The command uses the "-A FORWARD" option to append a rule to the FORWARD chain, which is responsible for processing packets that are being forwarded between different networks or interfaces. The "-p icmp" option specifies that the rule applies to ICMP packets, and the "--icmp-type echo-request" option specifies that it applies specifically to ICMP Echo Request packets. The "-j DROP" option tells iptables to drop (i.e., discard) any packets that match the preceding conditions, effectively preventing ICMP Echo Request packets from being forwarded between the networks or interfaces.

<img width="386" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/554b17bc-5f54-45f9-a768-80046b446f0e">
 
Figure 6: Creating a rule to deny ICMP traffic on the second VM from being forwarded through the network.

Step 9: After the command, ping the first VM from the third VM which shows “destination host unreachable” indicating that ICMP messages has been blocked from going through due to the rule created on the second VM to block ICMP messages.

<img width="359" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/983a2b43-2456-4386-a434-bd14d10ff48c">
 
Figure 7: Pinging the first VM from the third one shows ICMP messages has been blocked by the second VM (firewall).

Step 10: HTTP traffic was also blocked from being forwarded through the second VM (firewall). To do this, install apache2 (a popular HTTP web server) on the first VM which can serve HTTP traffic with the command “ sudo apt-get install apache2 ” and start the service using the command “systemctl start apache2”.
 
<img width="288" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/dc351f22-64cd-42d5-82ec-f8b8f4597936">
 
Figure 8: Starting Apache2 web service on the first VM

Step 11: To test the HTTP connection, open a web browser on the third virtual machine and enter the URL: 172.16.48.136 to access the apache2 default page on the first VM. This shows that the firewall allows HTTP traffic to be forwarded.

<img width="363" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/0ce6c975-9eb4-41c5-a787-9921c3904bbc">
 
Figure 9: Accessing the apache2 webserver default page.

Step 12: To block HTTPS traffic with the firewall, run the command “iptables -A FORWARD -p tcp --dport 80 -m conntrack --ctstate NEW -j REJECT” on the second VM. This command blocks HTTP traffic from being forwarded through the firewall.

 <img width="302" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/45f0c52c-311b-4258-aad1-2110d79e728a">

Figure 10: Blocking HTTP traffic on the second VM.

Step 13: To confirm if the firewall blocks HTTP traffic, open a web browser on the third virtual machine and enter the URL: 172.16.48.136 to access the apache2 default page on the first VM. This does not go through, showing the firewall (second VM) has blocked all incoming HTTP traffic.

<img width="403" alt="image" src="https://github.com/lilkhaleef/Firewall/assets/145224160/851a8c79-d163-49f4-adfa-d2c17d24887b">
 
Figure 11: Connecting to First VM after rule creation.
