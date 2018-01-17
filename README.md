How to Setup Fortigate Active-Passive mode at Azure
======================================================

If customer want to support high availability deployment about network virtual appliance(NVA) at Azure, they can either using load balancer to support active-active or using vendor specify method to support active-passive. This article will introduce how to setup Fortigate firewall active-passive mode at Azure. <br>

Overall design is that customer deploy 2 Fortigate firewall NVA, those two devices configured as active-passive pair. Customer use user define route(UDR) to point all traffic go to active firewall and cluster IP is associated with active firewall. When active devices fail, passive device use Azure service principle to modify UDR to passive firewall and associate cluster IP to passive firewall. <br>

Topology
-------------------------------------
![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/Drawing1.png)


Azure Network Setup
------------------------------------------
Setup VNET with 4 subnet. Public subnet 20.0.0.0/24, Internal subnet 20.0.1.0/24, Sync subnet 20.0.2.0/24, Management subnet 20.0.3.0/24. <br>
Create one public IP (ClusterPublicIP) and associate it in fgta P1 interface. <br>
Create UDR(default-udr) and point the default route(0.0.0.0/0) to fgta P2 interface (20.0.1.70). <br>


Fortigate NVA Setup
-------------------------------------
We will use 4 NIC virtual machine. One NIC is for public interface, one NIC is for internal interface, one NIC is for session sync interface and one NIC is for out of band management interface.

Parameters            | Active NVA    | Passive NVA
----------------------| ------------- | ------------
Fortigate NVA         | fgta          | fgtb
Public NIC     (P1)   | 20.0.0.70/24  | 20.0.0.80/24
Internal NIC   (P2)   | 20.0.1.70/24  | 20.0.1.80/24
Sync NIC       (P3)   | 20.0.2.70/24  | 20.0.2.80/24
Management NIC (P4)   | 20.0.3.70/24  | 20.0.3.80/24


Create an Azure Active Directory application and service principal
---------------------------------
## Create an Azure Active Directory application
New application registration.<br>
![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/AAD.PNG)
Provide a name and URL for the application. Select Web app / API for the type of application you want to create.<br> 
![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/AppReg.PNG)
After create the application, write down the application ID and setup the key.
![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/APPID.PNG)
Create the key for Application and write down.
![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/KEY.PNG)
