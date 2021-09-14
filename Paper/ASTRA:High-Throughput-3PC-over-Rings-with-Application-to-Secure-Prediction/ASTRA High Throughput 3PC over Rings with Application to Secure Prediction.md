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

Correctness：令$(m_x = x + \lambda_x),(m_y = y + \lambda_y)$，有$\gamma_{xy} = \lambda_x\lambda_y$证明$\Pi_{\mathrm{Mul}}^s(w_x,w_y,w_z)$是正确的。
$$
\begin{aligned}
m_z &= m_xm_y - m_x\lambda_y - m_y\lambda_x + \lambda_z + \gamma_{xy} \\
&=(m_x - \lambda_x)(m_y - \lambda_y) + \lambda_z \\
&=xy + \lambda_z
\end{aligned}
$$
**Output-reconstruction**

重构比较简单，可以看到，每两方就可以还原输出$y = m_y - \lambda_{y,1} - \lambda_{y, 2}$，将重构协议命名为$\Pi_{\mathrm{Res}}^s$。

最后将前面三个阶段的协议整合起来，命令为$\Pi_{\mathrm{3pc}}^s$

<img src="/home/karrylee/Learning_Note/Paper/ASTRA:High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/pic/6.png" style="zoom:67%;" />

**Theorem 1： $\Pi_{\mathrm{3pc}}^s$**离线阶段需要一轮通信共M个ring elements，在线 Input-sharing 阶段需要通信一轮，共2I个ring elements；电路计算阶段，D轮通信共2M ring elements；output-reconstruction 阶段一轮共3O的通信量。

#### 3PC with malicious security

3PC恶意安全的方案跟半诚实的方案类似，分为三个阶段。

**Input Sharing and Output Reconstruction Stages**：

在协议$\Pi_{\mathrm{Sh}}^s$中$\lambda$-shares是consistent的，因为采样的时候非交互。但是，如果$P_0$拥有一个数$x$并且想创建一个inconsistent的$[\![\cdot]\!]$-sharing，那么它会发送不一致的$m_x$给$P_1,P_2$，我们只需要让$P_1,P_2$交换验证$H(m_x)$即可，如果不匹配则输出Abort。记Input Sharing为$\Pi_{\mathrm{Sh}}^m$。

对于输出也是同样采用验证的方式，不匹配则输出Abort。记Reconstruction为$\Pi_{\mathrm{Rec}}^m([\![y]\!], P)$

<img src="/home/karrylee/Learning_Note/Paper/ASTRA:High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/pic/7.png" style="zoom: 67%;" />

**Circuit Evaluation Stage**：

$\Pi_{\mathrm{Add}}$是本地计算所以恶意者存在的情况下是不受影响的，所以重点考虑$\Pi_{\mathrm{Mul}}^s$遭遇一个恶意者存在的情况。此时面临着两个问题：1）$P_0$作为恶意者可以分享不正确的$\gamma_{xy} \ne \lambda_x\lambda_y$；2）corrupt evaluator（$P_1\ or\ P_2$）可以发送不一致的$m_z$，所以讨论$\Pi_{\mathrm{Mul}}^s$的恶意安全性需要分为$\{P_0\}$和$\{P_1,P_2\}$两个集合来讨论，但我们的方法可以将两种情况统一起来解决。主要的思想是：检察三元组的$[\![\cdot]\!]$-sharing的product-relation。具体地，

假设$P_1$的$m_z$被不正确的构造，需要检察它的正确性，可以借助$P_0$：$P_1$可以将$m_x,m_y$发送给$P_0$，因为$P_0$在离线阶段知道$\lambda_x,\lambda_y,\lambda_{z}$，所以它能计算出$m_z$并发送给$P_1$进行校验，但这样会泄露$m_x,m_y$明文给$P_0$。于是考虑加一个掩码$m_x^* = m_x + \delta_x, m_y^* = m_y + \delta_y$。$P_0$计算
$$
\begin{aligned}
m_z^* &= -m_x^* \lambda_y - m_y^* \lambda_x + \lambda_z + 2\gamma_{xy}\\
&=-(m_x + \delta_x)\lambda_y - (m_y + \delta_y)\lambda_x + \lambda_z + 2\gamma_{xy}\\
&=(m_z - m_zm_y) - (\delta_x\lambda_y + \delta_y\lambda_x - \gamma_{xy})\\
&=(m_z - m_xm_y) - \chi
\end{aligned}
$$
这里把$m_z = m_xm_y - m_x\lambda_y - m_y\lambda_x + \lambda_z + \gamma_{xy}$代入即可得到这个结论。如果$P_0$知道$\chi$那么就可以计算$m_z^*$发送给$P_1,P_2$进行判断了。下面继续推导如果使$P_0$得到$\chi$。我们发现如果向$P_0$公开$\chi$，那么$P_0$根据自己拥有的$\lambda_x,\lambda_y,\gamma_{xy}$，以及$m_x + \delta_x,  m_y + \delta_y$可以得到$m_x,m_y$。所以我们要对$\chi$加一个掩码，即$\delta_x\lambda_y + \delta_y\lambda_x + \delta_z - \gamma_{xy}$，这里$\delta_z$是一个随机数。所以可以让$P_1,P_2$随机采样$\delta_x,\delta_y,\delta_z$然后做$\chi$的$[\cdot]$-sharing并发送给$P_0$，$P_0$收到后做个Add就可以得到$\chi$。这里引入了一个新的问题，如果$P_1,P_2$中的一个是恶意者，那么要保证发送正确的$\chi$的share。

梳理一下，通过前面的描述明晰了当evaluator中的一个是恶意者的情况——检测不一致$m_z$的问题。并遗留下来两个问题：1）当$P_0$作恶时，$\gamma_{xy} \ne \lambda_x\lambda_y$；2）当$P_1$或者$P_2$作恶时，对$\chi$的不正确发送。这两个问题通过离线阶段的三元组来解决。一旦$P_0$接收到$\chi$后，各方本地计算$a = \delta_x - \lambda_x, b = \delta_y - \lambda_y$以及$ c = (\delta_z+\delta_x\delta_y) - \chi$的$[\![\cdot]\!]$-sharing。

![](/home/karrylee/Learning_Note/Paper/ASTRA:High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/pic/8.png)

观察到（$a,b,c$）构成一个乘法三元组
$$
\begin{aligned}
ab & = (\delta_x - \lambda_x)(\delta_y - \lambda_y) =\delta_x\delta_y + \lambda_x\lambda_y - \delta_x \lambda_y - \delta_y\lambda_y \\
&=(\delta_x\delta_y +\delta_z) - (\delta_x\lambda_y + \delta_y\lambda_x + \delta_z - \gamma_{xy})\\
&=(\delta_z+\delta_x\delta_y) - \chi = c
\end{aligned}
$$
这里$\gamma_{xy} = \lambda_x\lambda_y$。通过验证这个乘法元组（它的检验方法同《High-Throughput Secure Three-Party Computation for Malicious Adversaries and an Honest Majority》(EUROCRYPT'17)）可以解决提到的这两个遗留问题 ：

（1）、如果$P_0$作恶，$\gamma_{xy} \ne \lambda_x\lambda_y$，不妨令$\gamma_{xy} = \lambda_x\lambda_y + \Delta$，那么
$$
\begin{aligned}
c & = (\delta_x\delta_y +\delta_z) - (\delta_x\lambda_y + \delta_y\lambda_x + \delta_z - (\gamma_{xy} - \Delta))\\
&=(\delta_x - \lambda_x)(\delta_y - \lambda_y) - \Delta = ab - \Delta \ne ab
\end{aligned}
$$
（2）、考虑evaluators中的一个作恶，不妨假设是$P_1$作恶。在发送$\chi_1$给$P_0$的时候发送一个错误值$\chi_1 + \Delta$，那么$P_0$恢复的值为$\chi^{'}= \chi + \Delta$
$$
\begin{aligned}
c & = (\delta_x\delta_y +\delta_z) - \chi^{'} = (\delta_x\delta_y +\delta_z) - (\chi+\Delta)\\
&= (\delta_x\delta_y +\delta_z) - (\delta_x\lambda_y + \delta_y\lambda_x + \delta_z - \gamma_{xy}) - \Delta\\
&= (\delta_x - \lambda_x)(\delta_y - \lambda_y) - \Delta = ab - \Delta \ne ab
\end{aligned}
$$
（3）、加上前面讨论的一个问题：在线阶段$m_z$不一致，不妨假设$P_1$发送的$[m_z]_{P_1} + \Delta$给$P_2$。那么诚实方$P_2$计算得到的是$m_z + \Delta$。这种情况下，诚实方$P_0$在离线阶段正确计算了$\chi = \delta_x\lambda_y + \delta_y\lambda_x + \delta_z - \gamma_{xy}$。由于在线计算阶段$P_0$得到了诚实的$m_x + \delta_x,  m_y + \delta_y$以及有$\gamma_{xy} = \lambda_x\lambda_y$成立。如果它将这个结果发送给$P_1,P_2$，那么$P_2$接收到的$m_z^*$与$m_z + \Delta - m_xm_y +\delta_z$不同，所以会输出Abort。

下面是完整的恶意安全下的乘法计算协议

![](/home/karrylee/Learning_Note/Paper/ASTRA:High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/pic/9.png)

#### Achieving Fairness

我们通过一个公平的Reconstruct协议将$\Pi_{\mathrm{3pc}}^m$的安全性从可中止提升到Fairness。为了fairly reconstruct $[\![y]\!]$，可以使用commitment。在离线阶段，$\{P_0, P_1\}$承诺它们的公共share $\lambda_{y,1}$ 发送给$P_2$；$\{P_0,P_2\}$同样承诺公共的share $\lambda_{y,2}$ 发送给$P_1$；在线阶段$\{P_1,P_2\}$承诺它们的公共值$m_y$发送给$P_0$。由诚实多数的假设下，$(P_0, P_1), (P_0, P_2), (P_1, P_2)$中至少有1对是诚实的。当承诺不匹配的时候，向其它方广播Abort。如果没有Abort的信号出现，则按照Reconstruct的规则打开$y$。但是这里会有一个问题，我们缺乏一个可信任的广播信道（有可能恶意方$P_0$向$P_1$发送Abort，向$P_2$发送contine的信号，最终结果就会出现不一致）。为解决这个问题，可以在离线阶段$(P_0, P_1), (P_0, P_2)$分别将相应的随机数$r_1,r_2$的承诺发送给对方，在线阶段收到Abort的时候，会附带一个信息用来证明另一方收到的也是Abort。

![](/home/karrylee/Learning_Note/Paper/ASTRA:High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/pic/10.png)