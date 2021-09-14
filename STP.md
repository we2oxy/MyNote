# **STP（生成树协议）**

## 故障现象

在二层交换网络中，一旦存在环路就会造成报文在环路内不断循环和增生，产生**广播风暴**，并且会导致**MAC地址表震荡**，从而占用所有的有效带宽，使网络变得不可用。

### **STP的选举机制**

1. **选择根网桥**:在所有交换机中选举出一台作为**根网桥（root bridge）**；选举规则：Bridge-ID小的优先。Bridge-ID(桥ID,BID),用来标识交换机身份。格式：优先级+mac地址，优先级默认是32768,4096倍数。
2. **选择根端口**：每台非根网桥（交换机）选举出一个**根端口（root port）**;选举规则：1、到达根网桥开销小的优先；2、对端交换机BID小的优先；端口ID小的优先。  开销（cost），代表路径耗费的代价和成本，带宽越大，开销越小。
3. **选择指定端口**：每个物理网段选举出一个**指定端口（Designated port）**。选举规则：1、到达根网桥开销小的优先；2、本交换机BID小的优先；端口ID小的优先。  开销（cost），代表路径耗费的代价和成本，带宽越大，开销越小。4. 阻塞其它端口：剩下没有角色的端口就是阻塞端口（Blocked port）



![image-20210909174507722](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210909174507722.png)



![image-20210909174854564](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210909174854564.png)



![image-20210909174913002](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210909174913002.png)