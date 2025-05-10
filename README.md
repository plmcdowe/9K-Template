> <b>1) Power on both switches.</b>    
> <b>2) Console into both 9Ks</b>
>
>> <b>2.1) DO NOT USE the "Setup Utility" or any "PowerOn Auto Provisioning"</b>    
>>
>>> <b>2.1a) You should be able to simply send n to any initial prompts - until you are eventually at "switch>" prompt.</b>    
>>
>> <b>2.2) Once at "switch>" - send sh ver</b>     
>>
>>> <b>2.2a) If either 9K is not xx.x(x), upgrade to xx.x, ensuring both are the same "dot" and "parenthecial" release.</b>    
>
> <b>3) If both 9Ks are a matching xx.x(x) release, begin connecting physical interfaces:</b>    
>
>> <b>3.1) Connect both 9Ks to each other on Eth 1/48 using a 10Gig Twinax cable, for - vPC Peer-Keepalive</b>    
>>
>> <b>3.2) Connect both 9Ks to each other for - vPC Peer-Link:</b>    
>>
>>> <b>3.2a) Peer-Link will be port-channeled. Interface speed of port-channel interfaces must match. Use any of the following legal combinations:</b>   
>>>
>>>> <b>- 10Gig SFP interfaces Eth 1/1 - 47</b>   
>>>> <b>- QSFP interfaces with the 10Gig QSFP -to- 10Gig SFP adapter Eth 1/49 - 52</b>   
>>>> <b>- QSFP interfaces with 40Gig (or up to 100G) QSFP Eth 1/49 - 52</b>   
>>>
>>> <b>3.2b) Ideally, the vPC Peer-Link is established on Eth 1/53 and 1/54</b>   
>
> <b>4) Console into both 9Ks - ideally, at the same time with each serial session in "split windows" of SecureCRT</b>
   
|9KA|9KB|
|---|---|
|`feature ssh`|`feature ssh`|
|`feature tacacs+`|`feature tacacs+`|
|`feature lacp`|`feature lacp`|
|`feature scp-server`|`feature scp-server`|
|`feature vpc`|`feature vpc`|
|`feature interface-vlan`|`feature interface-vlan`|
|`feature netflow`|`feature netflow`|
|`no feature telnet`|`no feature telnet`|
|`cli alias name wr copy run start`|`cli alias name wr copy run start`|
|`int mgmt0`|`int mgmt0`|
|`  vrf member management`|`  vrf member management`|
|`  ip address IPa.IPa.IPa.IPa/24`|`  ip address IPb.IPb.IPb.IPb/24`|
|`  no shutdown`|`  no shutdown`|
|`vpc domain 151`|`vpc domain 151`|
|`  role priority 1`|`  role priority 2`|
|`vlan 101`|`vlan 101`|
|`  name keepalive`|`  name keepalive`|
|`  vrf context keepalive`|`  vrf context keepalive`|
|`int vlan101`|`int vlan101`|
|`  vrf member keepalive`|`  vrf member keepalive`|
|`  ip address IPa.KEEP.ALIVE.IPa/24`|`  ip address IPb.KEEP.ALIVE.IPb/24`|
|`int e1/48`|`int e1/48`|
|`  description Peer-KeepAlive`|`  description Peer-KeepAlive`|
|`  switchport`|`  switchport`|
|`  switchport access vlan 101`|`  switchport access vlan 101`|
|`vrf context management`|`vrf context management`|
|`  ip route 0.0.0.0/0 MGMT.NET.RTR.IP`|`ip route 0.0.0.0/0 MGMT.NET.RTR.IP`|
|`int vlan101`|`int vlan101`|
|`  no shut`|`  no shut`|
|`int e1/48`|`int e1/48`|
|`  no shut`|`  no shut`|
|`vpc domain 151`|`vpc domain 151`|
|`  peer-keepalive destination IPb.KEEP.ALIVE.IPb source IPa.KEEP.ALIVE.IPa vrf keepalive`|`  peer-keepalive destination IPa.KEEP.ALIVE.IPa source IPb.KEEP.ALIVE.IPb vrf keepalive`|

<b><ins>PAUSE HERE - ENSURE THAT THE peer-keepalive IS SUCCESSFULL with:</ins>:</b>    

`sh vpc peer-keepalive`    

You should see <i>"vPC keep-alive status    : peer is alive"</i>    

<ins>Now, configure all production vlans globably \{ VM networks, ESX & vMotion, host management, admin workstation, etc. \} -      
ALL VLANS for HOSTS must be configured on the PeerLink port-channel</ins>    

|<i>Comment</i>|9KA|9KB|
|---|---|---|
||`vlan 888`|`vlan 888`|
||`  name ChangeMe8`|`  name ChangeMe8`|
||`vlan 999`|`vlan 999`|
||`  name ChangeMe9`|`  name ChangeMe9`|
|<ins><i>Peer-Link int's:</i></ins>|`int e1/53-54`|`int e1/53-54`|
||`  description VPC-PeerLink`|`  description VPC-PeerLink`|
||`  channel-group 5 mode active`|`  channel-group 5 mode active`|
||`int Po5`|`int Po5`|
||`  description VPC-PeerLink`|`  description VPC-PeerLink`|
||`  switchport`|`  switchport`|
||`  switchport mode trunk`|`  switchport mode trunk`|
||`  switchport trunk allowed vlan 888,999`|`  switchport trunk allowed vlan 888,999`|
||`  spanning-tree port type network`|`  spanning-tree port type network`|
||`  vpc peer-link`|`  vpc peer-link`|
||`int e1/53-54`|`int e1/53-54`|
||`  no shut`|`  no shut`|
||`int Po5`|`int Po5`|
||`  no shut`|`  no shut`|

<b><ins>PAUSE HERE - ENSURE THAT THE peer-link IS SUCCESSFULL with:</ins>:</b>    

`sh vpc`    

You should see: <i>"Peer status    : peer adjacency formed ok"</i>    
Also verify that 9KA is "primary" for vPC role    

<ins>Now, prep the interfaces for the port-channeled trunk to CORE/root switch</ins>:    

|<i>Comment</i>|9KA|9KB|
|---|---|---|
|<ins><i>tunk int's:</i></ins>|`int e1/49-52`|`int e1/49-52`|
||`  description CORE_Po255_Trunk`|`  description CORE_Po255_Trunk`|
||`  channel-group 255 mode active`|`  channel-group 255 mode active`|
||`int Po255`|`int Po255`|
||`  description CORE_Po_49-52`|`  description CORE_Po_49-52`|
||`  switchport`|`  switchport`|
||`  switchport mode trunk`|`  switchport mode trunk`|
||`  switchport trunk native vlan 111`|`  switchport trunk native vlan 111`|
||`  switchport trunk allowed vlan 888,999`|`  switchport trunk allowed vlan 888,999`|
||`  vpc 255`|`  vpc 255`|    


> <b><ins>DO NOT "no shut" THE CORE TRUNKS or PORT CHANNEL UNTIL PHYSICALLY CONNECTED TO THE CORE</ins>:</b>    
> 
> |9KA|9KB|
> |---|---|
> |`int e1/49-52`|`int e1/49-52`|
> |`  no shut`|`  no shut`|
> |`int Po255`|`int Po255`|
> |`  no shut`|`  no shut`|


<b><ins>Now, prep the interfaces for the host connections: \{ VM-net Port Channels, ILO, and ESX/vMotion \}</ins></b>    

<ins>VM-net Port Channels</ins>:    

|9KA|9KB|
|---|---|
|`int e1/1`|`int e1/1`|
|`  description K8-01_Po11`|`  description K8-01_Po11`|
|`  channel-group 11 mode active`|`  channel-group 11 mode active`|
|`int Po11`|`int Po11`|
|`  description K8-01_e1/1`|`  description K8-01_e1/1`|
|`  switchport`|`  switchport`|
|`  switchport mode trunk`|`  switchport mode trunk`|
|`  switchport trunk allowed vlan 888,999`|`  switchport trunk allowed vlan 888,999`|
|`  spanning-tree port type edge trunk`|`  spanning-tree port type edge trunk`|
|`  no lacp suspend-individual`|`  no lacp suspend-individual`|
|`  vpc 11`|`  vpc 11`|
|`int e1/1`|`int e1/1`|
|`  no shut`|`  no shut`|
|`int Po11`|`int Po11`|
|`  no shut`|`  no shut`|

<ins>ILO</ins>:   

|<i>Comment</i>|9KA|9KB|
|---|---|---|
||`int e1/2`|`int e1/2`|
||`  description K8-01-ILO`|`  description K8-01-ILO`|
||`  switchport`|`  switchport`|
|<ins><i>site admin vlan:</i></ins>|`  switchport access vlan 15`|`  switchport access vlan 15`|
||`  no shutdown`|`  no shutdown`|

<ins>ESX</ins>:    

|<i>Comment</i>|9KA|9KB|
|---|---|---|
||`description K8-01-ESX`|`description K8-01-ESX`|
||`  switchport`|`  switchport`|
||`  switchport mode trunk`|`  switchport mode trunk`|
|<ins><i>all vlan IDs:</i></ins>|`  switchport trunk allowed vlan 888,999`|`  switchport trunk allowed vlan 888,999`|
||`  no shutdown`|`  no shutdown`|

<ins>Additional config once all interfaces are preped</ins>:    

|9KA|9KB|
|---|---|
|`username youraccount password YourLocalAccountPassword role network-admin`|`username youraccount password YourLocalAccountPassword role network-admin`|
|`username admin password ChangeDefaultAdminAccountPassword`|`username admin password ChangeDefaultAdminAccountPassword`|
|`password strength-check`|`password strength-check`|
|`hostname PRIM-9KA-IP.IP`|`hostname PRIM-9KB-IP.IP`|
|`crypto key generate rsa modulus 2048`|`crypto key generate rsa modulus 2048`|

<b><ins>PAUSE HERE for key generation - continue copy/paste after key is generated</b</ins>:    

|9KA and 9KB|
|---|
|`banner motd #|
|`YOUR BANNER HERE`|
|`#`|
|`ntp server IP.IP.IP.IP use-vrf manage prefer`|
|`ntp server IP.IP.IP.IP use-vrf manage`|
|`ntp server IP.IP.IP.IP use-vrf manage`|
|`ip access-list vty-acl-in`|
|`  permit tcp IP.IP.IP.IP/24 any eq 22`|
|`  permit tcp IP.IP.IP.IP/24 any eq 22`|
|`  permit tcp IP.IP.IP.IP/24 any eq 22`|
|`  permit tcp IP.IP.IP.IP/32 any eq 22`|
|`  permit tcp IP.IP.IP.IP/16 any eq 22`|
|`  5000 deny ip any any log`|
|`no ip domain-lookup`|
|`ip domain-name ng.ds.army.mil`|
|`ip name-server IP.IP.IP.IP IP.IP.IP.IP use-vrf management`|
|`tacacs-server host IP.IP.IP.IP key YourTACACSkeyHere`|
|`tacacs-server host IP.IP.IP.IP key YourTACACSkeyHere`|
|`tacacs-server host IP.IP.IP.IP key YourTACACSkeyHere`|
|`aaa group server tacacs+ ISE_TACACS`|
|`  server IP.IP.IP.IP`|
|`  server IP.IP.IP.IP`|
|`  server IP.IP.IP.IP`|
|`  use-vrf management`|
|`line console`|
|`  exec-timeout 5`|
|`line vty`|
|`  session-limit 15`|
|`  exec-timeout 5`|
|`  access-class vty-acl-in in`|
|`aaa authentication login default group ISE_TACACS local`|
|`aaa authentication login console local`|
|`login on-success log`|

WRITE THE CONFIGs NOW with: `wr`    
