## EIGRP Debugging
Refer to: [[Dynamic Routing]], [[RIP & EIGRP]]
```
Router A# debug eigrp packets 
...
01:39:13: EIGRP: Received HELLO on Serial0/0 nbr 10.1.2.2 
01:39:13: AS 100, 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/0 
01:39:13:      K-value mismatch
```
Router A has received a HELLO packet from a neighbour but is encountering an issue due to a K-value mismatch.
- The output from the **`debug eigrp packets`** command provides detailed information about EIGRP packet exchanges.
- In this case, it shows that Router A has received a HELLO packet from a neighbour but is encountering an issue due to a K-value mismatch.
--- 
1. **`01:39:13: EIGRP: Received HELLO on Serial0/0 nbr 10.1.2.2`**
    - At **01:39:13**, Router A received an **EIGRP HELLO packet** on the **Serial0/0** interface from a neighbour with the IP address **10.1.2.2**.
    - HELLO packets are used in EIGRP to discover and maintain neighbour relationships between routers.
2. **`01:39:13: AS 100, 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/0`**
    - **AS 100**: This refers to the EIGRP **Autonomous System number** that Router A is operating in, which is **100**.
    - The rest of this line (`0x0, Seq 0/0 ...`) represents internal EIGRP debugging information, including sequence numbers, queues, and reliability counters. This is primarily useful for Cisco engineers or deep diagnostics, but it is less relevant for common troubleshooting.
3. **`01:39:13: K-value mismatch`**
    - This message indicates that Router A has detected a **K-value mismatch** with the neighbour at IP address **10.1.2.2**.
    - **K-values** are part of EIGRP's metric calculation formula. EIGRP uses five K-values (K1 through K5) to define how metrics like bandwidth, delay, load, and reliability influence route selection.
    - For two routers to form a neighbour relationship, their K-values must match. A mismatch will prevent EIGRP from establishing or maintaining an adjacency with the neighbour, meaning Router A and its neighbour won't exchange routing information properly.
