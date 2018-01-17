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
Create new resource gourp (fortigate). <br>


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


Create an Azure Active Directory application 
---------------------------------

New application registration.<br>
![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/AAD.PNG)

Provide a name and URL for the application. Select Web app / API for the type of application you want to create.<br> 
![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/AppReg.PNG)

After create the application, write down the application ID and setup the key.<br>

![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/APPID.PNG)

Create the key for Application and write down.<br>
![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/KEY.PNG)

Assign the access control to application
-----------------------------------------
Choose the resource group (forigate) and check IAM to assign privilidge.
![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/IAM.PNG)

Add application "fotigate" as "network contributor" role 
![](https://github.com/yinghli/Fortigate-NVA-HA/blob/master/IAM2.PNG)

Fortigate NVA Azure configuration
-----------------------------------
In Fortigate configuration, you need to setup "tenant-id", "subscription-id" and "resource-group". Those information you can check it in portal.<br>
"Client-id" and "Client-secret" is from AAD application Setup.
"public-nic", "public-ip", "route-table", "route" and "next-hop" is from Azure setup.
```
config system azure
    set tenant-id "942b80cd-1b14-42a1-8dcf-4b21dece61ba"
    set subscription-id "2f96c44c-cfb2-4621-bd36-65ba45185e0c"
    set resource-group "fortigate"
    set client-id "14dbd5c5-307e-4ea4-8133-68738141feb1"
    set client-secret 
    set public-nic "fgtaport1"
    set public-ip "ClusterPublicIP"
    set route-table "default-udr"
    set route "defaultroute"
    set next-hop "20.0.1.70"

```
