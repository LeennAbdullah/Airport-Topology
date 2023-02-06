# Airport-Topology
The project is to design a proposal for setting up a network in an airport. The airport has 
three departments: Airport Authority, Flight Service Providers, and Guests. The airport 
authority maintains a server that handles flight management controls. The flight service 
providers should have access only to the specific server in the airport authority network 
and not to any other systems. The guest users should have wireless access to a high-speed 
internet connection which should be shared among all the users in all the departments. 
The wireless access should be using a common password. The guest users should not have 
access to the other departments. The users should obtain IP addresses automatically. The 
airport authority has 20 users, the flight service providers have 40 users, and the 
maximum number of guests is estimated to be 100.
--------------------------------------------------------------------------------------------
1-The toology
----------------------------------------------------
2-Access point based on number of guests to be added in order
to connect to the internet. and they should be automatically assigned to 
the DHCP server
for each laptop we will add WPC300N module after powering off
the laptop and then power it on they will automatically connect
to any access point but to make able to connect to the nearest one 
we will click on each Ap 
A-change name AP1
B-cahnge the config to auto on interface port 0
C-SSID based on the name AP1
D-choose WPA2-PSK and set any password 
-All devices will be desconnected to the AP since password
is added for authentication-
Apply it to all Ap 
then for each laptob choose desktop->PC wireless->connect 
and choose the AP and write the appropriate password
for authentication.
Then color it since its belong to VLAN2 based on the 
choosen color.
-----------------------------------------------------
3-Add DHCP server
NOTE : ROUTER 192.168.1.0 -> network IP
	      192.168.1.1 -> Default Gateway	 
A-change the name 
B-set services in DHCP to on 
C-Default gateway will be 192.168.1.1
and set the start 192.168.1.100
D-Set max number of devices to be 50 set any number 
THEN IMPORTANT:
DHCP -> DESKTOP-> IP CONFIGRATION 
ipv4 192.168.1.200
subnet 255.255.255.0
gateway 192.168.1.1
We have to assign Static IP address for the DHCP server 
to not change since its DHCP Server not other service to assign an IP 
for the DHCP server.
-------------------------------------------------
4-Configure the router by desktop -> CLI
en
conf t
int 0/0
ip add 192.168.1.1 255.255.255.0
no shut 
exit
exit
copy ru st
----------------------------------------------
CLICK on all guests to refresh the DHCP server
----------------------------------------------
5-To seperate each switch we will use 
the switch in order to create the VLAN Database and will be applied
for all the switches for each switch to know the number of vlans to 
be able to map in case of connection
en 
conf t
A-
vlan 2
name Airport authority
exit
B-
vlan 3
name Flight service providers
exit
C-
vlan 4
name Guests
exit
exit 
copy ru st
APPLIED FOR ALL OF THE SWITCHES
----------------------------------------------
6-We will configure the router so that the fa0/0 will be 
divided to get connection from 3 VLANs in which each will
get its own encapsulation
A-
en 
conf t
int fa0/0.1
encapsulation dot1Q 2
ipp addr 192.168.2.1 255.255.255.0
no shu 
B-
en 
conf t
int fa0/0.2
encapsulation dot1Q 3
ipp addr 192.168.3.1 255.255.255.0
no shu
C-
en 
conf t
int fa0/0.3
encapsulation dot1Q 4
ipp addr 192.168.4.1 255.255.255.0
no shu
exit 
exit
copy ru st
---------------------------------------------------------
specify fast ethernet for each
--------------------------------------------------------
7-Let the DHCP know that there is 3 VLANs 
so we will create 3 POOLS each for VLAN 
A-
Airport authority
192.168.2.1
----------
192.168.2.2
255.255.255.0
20
ADD
B-
Flight service Provider
192.168.3.1
----------
192.168.3.2
255.255.255.0
40
ADD
C-
Guests
192.168.4.1
----------
192.168.4.2
255.255.255.0
100
ADD
-------------------------------------------------------
8-We want to tell Router that DHCP will assign IP addresses 
for each VLAN seperatedly by using the IP helper based on 
the sub interfaces.
#
en
conf t
int fa0/0.1
ip helper-address 192.168.1.200
int fa0/0.2
ip helper-address 192.168.1.200
int fa0/0.3
ip helper-address 192.168.1.200
exit
exit
copy ru st
---------------------------------------------------------
9-The ACL will be done on the router 
#We can use the extended not the standart which means that we are allowed to use extended 100-199 the others are reserved 
A-flight service providers should have access only to the specific server that handles flight management 
control in the airport authority network 
not to any other systems.
B-the Other that the guest users should not have access to the other department
#APPLIED ON THE MAIN ROUTE THAT CONNECTS ALL TO THE INTERNET:
en 
conf t
access-list 104 deny ip 192.168.4.0 0.0.0.255 192.168.2.0 0.0.0.255
access-list 104 deny ip 192.168.4.0 0.0.0.255 192.168.3.0 0.0.0.255
access-list 104 permit ip any any  # allows all instead of accessing other department 
interface fastethernet 0/0.3
ip access-group 104 in or ip access-group 104 inbound
#the last means that this fast ethernrt should be applied on it those restrictions
exit
#We start from the specific then the general 
MOHEMMM:
----------------------------------
Put the IP address for the server to be 
192.168.2.2 #based on the VLAN that the server belongs to 
255.255.255.0 #for the subnet mask
192.168.2.1 #for the default gateway 
-----------------------------------
access-list 105 permit ip 192.168.3.0 0.0.0.255 host
192.168.2.2 #IP address for the Server not the DHCP the other
in which the IP address should be assigned statically 
access-list 105 deny ip 192.168.3.0 0.0.0.255 192.168.2.0 0.0.0.255
access-list permit ip any any
int fa0/0.2
ip access-group 105 inbound
exit 
exit
copy ru st
---------------------------------------------------------------
WRITE THE DEFAULT CONFIG 
show ru 
enter 
#in order to view all of the already configured.
--------------------------------------------------------
10-We have to pecify the access port or whether its trunk port 
and refresh each section of devices from DHCP-> STATCI -> DHCP
to ensure that the device belongs to the right VLAN based on 
IPV4 that will be assigned 
---------------------------------------------------------
NOTES:
1-write the IP addresses for all of the devices 
2-FOR testing can be done by 
	A-PDU
	B-PING 
A-PING from any guest to Airport or providers
should return unreachable for the destination host
B-USE tracert for the guest to see that only reaches the interface only
because of restrictions
3-We can do trouble shooting by clicking on themessage to
see the layers where is pass throught it 
and check the outbound PDU details.
-----------------------------------------------------------
