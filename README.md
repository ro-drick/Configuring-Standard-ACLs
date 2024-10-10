
## Overview

In this Packet Tracer lab, we will configure **OSPF (Open Shortest Path First)** routing protocol on **R1** and **R2** routers to enable full connectivity between all PCs and servers. Additionally, we will configure **Standard Numbered ACLs** on **R1** and **Standard Named ACLs** on **R2** to enforce the following network security policies:

1. Only **PC1** and **PC3** are allowed to access the **192.168.1.0/24** network.
2. Hosts in the **172.16.2.0/24** network cannot access the **192.168.2.0/24** network.
3. **172.16.1.0/24** cannot access the **172.16.2.0/24** network, and vice versa.

---

## Lab Topology

- **PC1 (172.16.1.1)**, **PC2 (172.16.1.2)**: Hosts in the **172.16.1.0/24** network.
- **PC3 (172.16.2.1)**, **PC4 (172.16.2.2)**: Hosts in the **172.16.2.0/24** network.
- **Server 1 (192.168.1.100)**: Located in the **192.168.1.0/24** network.
- **Server 2 (192.168.2.100)**: Located in the **192.168.2.0/24** network.
- **R1 (203.0.113.1/30)** and **R2 (203.0.113.2/30)**: Routers with serial connectivity between them.
- **Switches (SW1, SW2, SW3, SW4)**: Connect hosts and servers to the routers.

<img src= "https://github.com/ro-drick/Configuring-Standard-ACLs/blob/main/standard-acls.jpg">

---

## Objectives

1. Configure **OSPF** on R1 and R2 for full network connectivity.
2. Implement **Standard Numbered ACLs** on R1 to:
   - Only allow **PC1** and **PC3** to access the **192.168.1.0/24** network.
3. Implement **Standard Named ACLs** on R2 to:
   - Block access from **172.16.2.0/24** to the **192.168.2.0/24** network.
   - Block access from **172.16.1.0/24** to **172.16.2.0/24**, and vice versa.

---

## Network IP Addressing

| Device         | Interface      | IP Address       | Subnet Mask       |
|----------------|----------------|------------------|-------------------|
| **PC1**        | Fa0/1           | 172.16.1.1       | 255.255.255.0     |
| **PC2**        | Fa0/2            | 172.16.1.2       | 255.255.255.0     |
| **PC3**        | Fa0/1            | 172.16.2.1       | 255.255.255.0     |
| **PC4**        | Fa0/2            | 172.16.2.2       | 255.255.255.0     |
| **Server 1**   | Fa0/1            | 192.168.1.100    | 255.255.255.0     |
| **Server 2**   | Fa0/1            | 192.168.2.100    | 255.255.255.0     |
| **R1**         | Serial 0/0/0   | 203.0.113.1      | 255.255.255.252   |
| **R2**         | Serial 0/0/0   | 203.0.113.2      | 255.255.255.252   |
| **R1**         | G0/0, G0/1     | 172.16.1.254, 172.16.2.254 | 255.255.255.0 |
| **R2**         | G0/0, G0/1     | 192.168.1.254, 192.168.2.254 | 255.255.255.0 |

---

## Configuration Steps

### Step 1: Configure OSPF on R1 and R2

The goal is to allow full connectivity between all devices using OSPF. We will configure OSPF on both **R1** and **R2**, and enable OSPF for the relevant networks.

#### R1 OSPF Configuration

1. Enter OSPF configuration mode.
2. Assign OSPF process ID 1 and advertise the networks directly connected to R1.

```bash
R1(config)# router ospf 1
R1(config-router)# network 172.16.1.0 0.0.0.255 area 0
R1(config-router)# network 172.16.2.0 0.0.0.255 area 0
R1(config-router)# network 203.0.113.0 0.0.0.3 area 0
```

#### R2 OSPF Configuration

1. Enter OSPF configuration mode.
2. Assign OSPF process ID 1 and advertise the networks directly connected to R2.

```bash
R2(config)# router ospf 1
R2(config-router)# network 192.168.1.0 0.0.0.255 area 0
R2(config-router)# network 192.168.2.0 0.0.0.255 area 0
R2(config-router)# network 203.0.113.0 0.0.0.3 area 0
```

---

### Step 2: Configure Standard Numbered ACLs on R1

We will configure Standard Numbered ACLs on **R1** to meet the following requirements:
- **172.16.1.0/24** should not be able to access **172.16.2.0/24**
- **172.16.2.0/24** should not be able to access **172.16.1.0/24**

#### Steps:

1. Create ACL 1 to deny **172.16.1.0/24**, and permit all other hosts.
   
```bash
R1(config)# access-list 1 deny 172.16.1.0 0.0.0.255
R1(config)# access-list 1 permit any
R1(config)# access-list 2 deny 172.16.2.0 0.0.0.255
R1(config)# access-list 1 permit any
```

2. Apply the ACL to the **G0/1** and **G0/0** interfaces of R1 (which connects to the **192.168.1.0/24** network).

```bash
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip access-group 1 out
```

```bash
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip access-group 2 out
```

---

### Step 3: Configure Standard Named ACLs on R2

We will configure Standard Named ACLs on **R2** to meet the following requirements:
- **Hosts in 172.16.2.0/24** should not access the **192.168.2.0/24** network.
- **172.16.1.0/24** should not access **172.16.2.0/24**, and vice versa.

#### Steps:

1. Create a named ACL **TO_192.168.2.0/24** to block **172.16.2.0/24** from accessing the **192.168.2.0/24** network.

```bash
R2(config)# ip access-list standard TO_192.168.2.0/24
R2(config-std-nacl)# deny 172.16.2.0 0.0.0.255
R2(config-std-nacl)# permit any
```

2. Apply the ACL to the **G0/1** interface (which connects to the **192.168.2.0/24** network).

```bash
R2(config)# interface GigabitEthernet0/1
R2(config-if)# ip access-group TO_192.168.2.0/24 out
```

3. Create another named ACL **TO_192.168.1.0/24** to permit **172.16.2.1/24** and **172.16.1.1/24** to access the **192.168.1.0/24** network.

```bash
R2(config)# ip access-list standard TO_192.168.1.0/24
R2(config-std-nacl)# permit 172.16.2.1 0.0.0.255
R2(config-std-nacl)# permit 172.16.1.1 0.0.0.255
R2(config-std-nacl)# deny any
```

4. Apply this ACL to the **G0/0** interface on R2 (which connects to the **172.16.2.0/24** network).

```bash
R2(config)# interface GigabitEthernet0/0
R2(config-if)# ip access-group TO_192.168.1.0/24 out
```

---

## Testing the Configuration

After configuring OSPF and ACLs, use **ping** commands to test network connectivity and verify that the ACLs are working correctly.

### Testing from PC1 and PC3:
- **PC1 (172.16.1.1)** and **PC3 (172.16.2.1)** should be able to ping **Server 1 (192.168.1.

100)**.
- **PC1** and **PC3** should NOT be able to access the **192.168.2.0/24** network (i.e., they cannot ping **Server 2 (192.168.2.100)**).

### Testing from PC2 and PC4:
- **PC2 (172.16.1.2)** and **PC4 (172.16.2.2)** should NOT be able to access **Server 1** in **192.168.1.0/24**.
- **PC2** should not be able to ping **PC3**, and **PC3** should not be able to ping **PC2**, confirming that **172.16.1.0/24** and **172.16.2.0/24** are isolated.

---

## Conclusion

In this lab, we successfully implemented OSPF routing and applied ACLs to enforce network security policies. The OSPF configuration ensured full network connectivity, while the ACLs restricted access between specific network segments as required.


## Acknowledgements


Special thanks to **Jeremy's IT Lab** for providing valuable resources and tutorials that greatly contributed to the completion of this exercise. His in-depth explanations and practical demonstrations have been instrumental in enhancing my understanding of Cisco networking concepts and the effective use of Packet Tracer.

For more information and additional resources, visit [Jeremy's IT Lab](https://jeremysitlab.com/) and check out his YouTube for the full course, [Jeremy's IT Lab Free CCNA 200-301 | Complete Course](https://www.youtube.com/playlist?list=PLxbwE86jKRgMpuZuLBivzlM8s2Dk5lXBQ)
