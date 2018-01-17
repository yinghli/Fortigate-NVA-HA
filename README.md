How to Setup Fortigate Active-Passive mode at Azure
======================================================

If customer want to support high availability deployment about network virtual appliance(NVA) at Azure, they can either using load balancer to support active-active or using vendor specify method to support active-passive. This article will introduce how to setup Fortigate firewall active-passive mode at Azure. <br>

Overall design is that customer deploy 2 Fortigate firewall NVA, those two devices configured as active-passive pair. Customer use user define route(UDR) to point all traffic go to active firewall and cluster IP is associated with active firewall. When active devices fail, passive device use Azure service principle to modify UDR to passive firewall and associate cluster IP to passive firewall. <br>

Topology
-------------------------------------


Fortigate Setup
-------------------------------------
We will use 4 NIC virtual machine. One NIC is for public interface, one NIC is for internal interface, one NIC is for session sync interface and one NIC is for out of band management interface.

Parameters            | Values
----------------------| -------------
Public                | 20.0.0.0/24
Internal              | 20.0.1.0/24 
Sync                  | 20.0.2.0/24 
Management            | 20.0.3.0/24 
