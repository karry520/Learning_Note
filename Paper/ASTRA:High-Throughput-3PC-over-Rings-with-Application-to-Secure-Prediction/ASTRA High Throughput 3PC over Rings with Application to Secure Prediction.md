## ASTRA: High Throughput 3PC over Rings with Application to Secure Prediction

Harsh Chaudhari, Arpita Patra, Ajith Suresh (CCSW'19)

### ABSTRACT 

本文提出一种考虑半诚实和恶意安全性的三方计算协议，采用offline-online的范式，能够容忍1方腐化。半诚实安全假设下，模$2^l$的环中，一个乘法门的在线计算阶段仅需通信2 ring elements，这在3PC设定中是首次实现的。在恶意安全假设下，乘法计算需要4 ring elements的通信量，通信效率高于SOTA的5 ring elements。与此同时，恶意安全设定下也实现了Fairness（恶意方接收到output，当且仅当诚实方接收到output），且不附带多余的通信量。

作者将安全3PC协议应用到ML安全预测任务上（linear regression, linear SVM regression, logistic regression, and linear SVM classification）。将计算外包给三台不合谋的服务器，其构造满足半诚实和恶意安全的设定，其性能表现是当前最优。

### INTRODUCTION

安全3PC计算在半诚实和恶意设定下得到了相当有关注，本文主要的目标是提升效率。基于SS的方法可以在linear门实现非交互，所以重点关注multiplication门的算法设计。一个提升效率的方向是采用offine-online两阶段的方法，offline阶段执行与input无关的操作，online阶段执行与input有关的且快速的操作。现有的3PC计算协议可以分为两类：1）支持在素域下的算术计算；2）支持半诚实下环上的算术电路计算。现有的方案各有优缺点。

#### Our Contribution

本文所提出的协议都拥有一个的特点是在线计算的通信比3 pair少，因此也产生了更好的在线计算性能。将贡献总结如下：

1. Secure 3PC：我们的协议在半诚实安全设定下，计算乘法时在线计算阶段通信仅需要2 elements。平均下来每方的通信量小于1 elements，这在3PC下是第一次做到的。对于恶意安全的设定下，我们的乘法计算在线阶段通信需要4 elements，是现在最好的通信效率。此外，恶意安全下也满足Fairness。

2. Secure ML prediction：我们考虑MLaaS的场景，使用辅助服务器完成ML的安全预测。我们与ABY$^3$的效率进行了比较，并提出一种新颖的应用在分类协议中的常数轮安全比较协议。

   <img src="/home/karrylee/Learning_Note/Paper/ASTRA:High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/pic/1.png" style="zoom:80%;" />

3. Implementation：对于3PC，协议基于$\mathbb{Z}_{2^l}$环实现。我们使用延迟（runtime）和在线阶段的throughput作为性能比较的benchmarks。实验表明，我们的方法效率更高。

**Shared Key Setup**：使用预分享随机key的方式用RPF实现3PC非交互的通信。这里的keys有

1. 每对Parties有一个公共的key。$k_{01},k_{02},k_{12}$分别表示$(P_0, P_1),(P_0, P_2),(P_1, P_2)$之间共有的key
2. 三方公有的两个key。$k_{P, 1},k_{P, 2}$

### SHARING SEMANTICS

定义本文使用的三种类型的秘密分享形式：

1. $[\cdot]$-sharing：将$v$以加法秘密分享的形式分发给$P_1$和$P_2$，满足$v = v_1 + v_2$

2. $[\![\cdot]\!]$-sharing：一个值$v$被称为$[\![\cdot]\!]$-sharing 在$P_0,P_1,P_2$之间，那么满足

   - 存在一个$\lambda_v,m_v$满足$v = m_v - \lambda_v$
   - $P_1,P_2$以明文的形式知道$m_v$，$\lambda_v$以$[\cdot]$-sharing的形式分享在它们之间。即$[\lambda_v]_{P_1} = \lambda_{v,1},[\lambda_v]_{P_2} = \lambda_{v,2}$
   - $P_0$以明文的方式知道$\lambda_{v,1},\lambda_{v,2}$

   我们将每一方的sharing表示为$[\![v]\!]_{P_0} = (\lambda_{v,1}, \lambda_{v,2}), [\![v]\!]_{P_1} = (m_{v}, \lambda_{v,1}), [\![v]\!]_{P_2} = (m_{v}, \lambda_{v,2})$。将$v$的$[\![\cdot]\!]$-sharing表示为$[\![v ]\!] = (m_v, [\lambda])$。

3. Linearity of the secret sharing scheme：

   - 给定$x,y\in \mathbb{Z}_{2^l}$的$[\cdot]$-sharing，以及常数$c_1,c_2\in \mathbb{Z}_{2^l}$，参与方本地计算 $[c_1x + c_2 y]$

   $$
   \begin{aligned}
   \  [c_1x + c_2y] &= (c_1x_1 + c_2 y_1, c_1 x_2 + c_2 y_2) \\
   &= c_1[x] + c_2[y]
   \end{aligned}
   $$

   - 同理，给定$x,y\in \mathbb{Z}_{2^l}$的-$[\![\cdot]\!]$sharing，以及常数$c_1,c_2\in \mathbb{Z}_{2^l}$，参与方本地计算 $[\![c_1x + c_2 y]\!]$
     $$
     \begin{aligned}
     \  [\![c_1x + c_2y]\!] &= (c_1m_x + c_2 m_2, c_1 [\lambda_x] + c_2 [\lambda_y]) \\
     &= c_1[\![x]\!] + c_2[\![y]\!]
     \end{aligned}
     $$

   这种线性性质可以使得参与方本地计算加法或者关于常数的乘法。

### OUR 3PC PROTOCOL

#### 3PC with semi-honest security

半诚实下的协议分为三个阶段-input-sharing, circuit-evaluation和output-reconstruction 

**Input-sharing**

协议$\Pi_{\mathrm{Sh}}^s(P_i, x)$使得指定方将$x$按$[\![\cdot]\!]$-sharing。比如$P_i,i\in {1,2}$需要分享自己的输入$x$。离线阶段，$P_0,P_i$随机采样得到$\lambda_{x,i}$，所有方随机采样得到$\lambda_{x,3-i}$。这里$P_0,P_i$是可以得到$\lambda_x = \lambda_{x,i} + \lambda_{x,3-i}$的；在线阶段，$P_i$发送$m_x = x + \lambda_x$给$P_{j},j\in {1,2}$，然后$P_j$令$[\![x]\!]_{P_j} = (m_x, \lambda_{x,j})$。可以看到$P_i$没有泄露自己的隐私输入。

<img src="/home/karrylee/Learning_Note/Paper/ASTRA:High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/pic/3.png" style="zoom:80%;" />

**Circuit-evaluation**

参与方按照电路拓扑顺序依次计算，对于加法门$g:(w_x,w_y,w_z)$，那么可以按照Linearity of the secret sharing scheme本地计算，见协议$\Pi_{\mathrm{Add}}(w_x,w_y,w_z)$

<img src="/home/karrylee/Learning_Note/Paper/ASTRA:High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/pic/4.png" style="zoom:80%;" />

对于乘法门$g:(w_x,w_y,w_z)$，使用乘法协议$\Pi_{\mathrm{Mul}}^s(w_x,w_y,w_z)$计算$[\![x]\!]\times [\![y]\!] = [\![z]\!]$。基本思想与ABY2.0相同，不同的是这是三方的设置。

<img src="/home/karrylee/Learning_Note/Paper/ASTRA:High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/pic/5.png" style="zoom:80%;" />

**Output-reconstruction**

重构比较简单，可以看到，每两方就可以还原输出$y = m_y - \lambda_{y,1} - \lambda_{y, 2}$，将重构协议命名为$\Pi_{\mathrm{Res}}^s$。 将前面三个阶段的协议整合起来，

<img src="/home/karrylee/Learning_Note/Paper/ASTRA:High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/pic/6.png" style="zoom:67%;" />

#### 3PC with malicious security

#### Achieving Fairness

