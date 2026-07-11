**# Lab 1 — Multi-Subnet Routing with DHCP and Port Security

**Category:** Networking Lab **Tool:** Cisco Packet Tracer **Level:** Beginner–Intermediate **Author:** Sudipto (ECHO)

## Objective

Build two separate LANs — a "Home" network and an "Office" network — connected through a single router, each on its own subnet. The router performs inter-VLAN-style routing between directly connec[...]

This lab builds directly on Lab 1 (single-subnet static addressing) by adding: a second subnet, router-based DHCP, and switch port security.

## Topology
<img width="1137" height="810" alt="image" src="https://github.com/user-attachments/assets/aa049880-151e-4cca-beb7-c384c97a2667" />


|Device|Model|Network|Role|
|---|---|---|---|
|Router0|Cisco 2911|Both|Routes between Home and Office subnets, serves DHCP to Office|
|Switch (Home)|Cisco 2960-24TT|Home|Layer 2 connectivity for PC0–PC2, port security enabled|
|Switch1 (Office)|Cisco 2960-24TT|Office|Layer 2 connectivity for PC3–PC4, port security enabled|
|PC0, PC1, PC2|Generic PC|Home|Static IP addressing|
|PC3, PC4|Generic PC|Office|DHCP addressing|

## IP Addressing Table

### Home Network — `192.168.1.0/24` (Static)

|Device|Interface|IP Address|Subnet Mask|Gateway|
|---|---|---|---|---|
|PC0|FastEthernet0|192.168.1.10|255.255.255.0|192.168.1.1|
|PC1|FastEthernet0|192.168.1.11|255.255.255.0|192.168.1.1|
|PC2|FastEthernet0|192.168.1.12|255.255.255.0|192.168.1.1|
|Router0|GigabitEthernet0/0|192.168.1.1|255.255.255.0|—|

### Office Network — `192.168.2.0/24` (DHCP)

|Device|Interface|IP Address|Subnet Mask|Gateway|
|---|---|---|---|---|
|PC3|FastEthernet0|assigned via DHCP (e.g. 192.168.2.2)|255.255.255.0|192.168.2.1|
|PC4|FastEthernet0|assigned via DHCP|255.255.255.0|192.168.2.1|
|Router0|GigabitEthernet0/1|192.168.2.1|255.255.255.0|—|

`192.168.1.1` and `192.168.2.1` are excluded from the DHCP pool so the router never hands out its own gateway addresses.

## Router Configuration

**Interfaces (both subnets):**

```
Router>enable
Router#config t
Router(config)#interface gig0/0
Router(config-if)#ip address 192.168.1.1 255.255.255.0
Router(config-if)#no shutdown
Router(config-if)#exit
Router(config)#interface gig0/1
Router(config-if)#ip address 192.168.2.1 255.255.255.0
Router(config-if)#no shutdown
```

**DHCP for the Office network:**

```
Router(config)#ip dhcp excluded-address 192.168.2.1
Router(config)#ip dhcp excluded-address 192.168.1.1
Router(config)#ip dhcp pool Office
Router(dhcp-config)#network 192.168.2.0 255.255.255.0
Router(dhcp-config)#default-router 192.168.2.1
Router(dhcp-config)#exit
Router(config)#exit
Router#copy run start
```

Because Home and Office are both directly connected to Router0, no static routes or routing protocol are needed — the router automatically knows how to forward between them.

## Switch Configuration — Port Security

Applied to the PC-facing access ports on each switch:

```
Switch(config)#interface range fa0/1-24
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport port-security
Switch(config-if-range)#switchport port-security maximum 1
Switch(config-if-range)#switchport port-security mac-address sticky
Switch(config-if-range)#switchport port-security violation shutdown
Switch(config-if-range)#exit
Switch#copy run start
```

- `maximum 1` — only one MAC address allowed per port
- `mac-address sticky` — switch learns and locks to whichever device is currently connected
- `violation shutdown` — port automatically disables itself if an unauthorized device is plugged in

## Verification

**Router interfaces up and addressed correctly:
<img width="647" height="788" alt="image" src="https://github.com/user-attachments/assets/9aff0f2c-035b-4f2b-af7e-444be7396567" />

**Home network connectivity (static IPs) — PC0 pinging PC1 and PC2, 0% loss:** 
<img width="488" height="218" alt="image" src="https://github.com/user-attachments/assets/2ff02477-ff57-44ae-a8b2-a075dc51c81e" />

**DHCP pool and exclusions configured on the router:** 
<img width="444" height="541" alt="image" src="https://github.com/user-attachments/assets/1c4c1e45-2dfd-410f-b75e-80a8781fbbd8" />


**Office PC successfully leased an address from the router's DHCP pool:** 
<img width="1129" height="222" alt="image" src="https://github.com/user-attachments/assets/5d8072ed-7b6f-491f-ae0a-51144327433f" />


**Port security applied and saved on the switch:** 
<img width="488" height="218" alt="image" src="https://github.com/user-attachments/assets/8f6b6104-354b-4f50-b5ed-551c6f452ff2" />


## Files in This Folder

| File                           | Description                                  |
| ------------------------------ | -------------------------------------------- |
| `lab2-home-office-routing.pkt` | Packet Tracer save file                      |
| `topology.png`                 | Full network topology screenshot             |
| `router-gig-config.png`        | Router interface configuration output        |
| `pc0-ping-test.png`            | Ping verification on the Home network        |
| `dhcp-pool-config.png`         | DHCP pool setup on the router                |
| `gig0-1-dhcp-exclude.png`      | Gig0/1 addressing and DHCP exclusions        |
| `office-dhcp-verify.png`       | Office PC confirming successful DHCP lease   |
| `port-security-config.png`     | Port security applied on switch access ports |
| `README.md`                    | This report                                  |

## Skills Demonstrated

- Multi-subnet LAN design and addressing scheme planning
- Router-on-a-stick style routing between two directly connected networks
- Configuring and verifying router-based DHCP services (pool, exclusions, lease verification)
- Static vs. DHCP addressing decisions (Home = static, Office = DHCP)
- Layer 2 port security (sticky MAC learning, violation handling)
- End-to-end verification using `ping`, `show ip interface brief`, and IP Configuration checks

