Create a virtual lab
* 2 boxes: Kali and Windows. 
* VM will have 2 network adapters
  * 1 "Attached to" NAT
  * 1 "Attached to "Internal Network" with the name **Pentestlab** 

# Download Virtual Machines
- Download Windows Virtual Machines from https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/
- Download Kali Linux

# Create Virtual Machine's internal network

On the Host machine create a new DHCP Server for being used for VM's internal network

```
$ vboxmanage dhcpserver add --netname Pentestlab --ip 192.168.10.1 --netmask 255.255.255.0 --lowerip 192.168.10.2 --upperip 192.168.10.20 --enable 
$ vboxmanage list dhcpservers

NetworkName:    Pentestlab
IP:             192.168.10.1
NetworkMask:    255.255.255.0
lowerIPAddress: 192.168.10.2
upperIPAddress: 192.168.10.20
Enabled:        Yes
```
**For Kali linux the DHCP server name should be the same as the network on virtual box settings**

# Configure Network Adapters

Go to "Settings>Network>Adapter 2"  and attached the adapter to  Internal Network.
Assign the network name for your VM OS. In this example, I have set and configured my internal network name to "Pentestlab".
Configure the same network name as what is configured in Virtualbox host.

# Configure VM OS

On Kali use the command line to set up the second network
```
$ ifconfig eth1 192.168.10.15
```

[Reference](http://www.thegeeky.space/2015/08/how-to-set-and-run-internal-dynamic-host-configuration-protocol-DHCP-virtual-network-on-centos-kali-linux-windows-in-virtualbox-with-practical-example.html)

