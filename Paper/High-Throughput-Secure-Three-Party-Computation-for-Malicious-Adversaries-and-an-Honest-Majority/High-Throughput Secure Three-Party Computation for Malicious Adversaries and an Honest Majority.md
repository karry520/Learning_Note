## High-Throughput Secure Three-Party Computation for Malicious Adversaries and an Honest Majority

Jun Furukawa, Yehuda Lindel, Ariel Nof, and Or Weinstein （EUROCRYPT'17）

### ABSTRACT

本文提出一个新的恶意安全下诚实多数的关于任意函数的安全三方计算协议。该工作基于Araki 等人（ACM  CCS 2016）提出的半诚实协议，改进为恶意安全下的三方计算协议。主要是通过构造Beaver乘法元组来对门电路进行验证，从而达到计算正确的目的。不同于TinyOT和SPDZ，我们基于cut-and-choose范式来保证元组被正确构造。同时提出新的可被用在相关cut-and-choose的组合分析方法。

### INTRODUCTION

安全计算的设定中，要求一组互不信任的多方联合计算一个约定的函数得到输出，而不泄漏关于输入的任意信息。安全计算协议要求保证1）Privacy；2）Correctness。为刻画安全性，对敌手的行为分为半诚实和恶意两种，对敌手的数量分为诚实多数和恶意多数两种。尽管前人已有工作做到使协议满足计算安全性或者统计安全性。但是这些在半诚实或者恶意的设定下要做到统计安全性要求诚实者的数量不少于2/3。

通常有两种方法来构造通用安全计算协议的方法：

1）基于Secret-Sharing，优点是适用于低带宽，通信量少，但是交互多。

2）基于Garbled-Circuit的方法，优点是常数轮交互，适用于高延迟网络的场景。

本文关注在恶意安全下诚实多数的三方计算，期望在一个高速的网络环境下实现高通量（适用于一个网络设备较好的场景下，计算效率高，吞吐量大）。Araki 等人提出的半诚实协议能实现每秒70亿的AND门计算。我们的方法满足恶意安全性，在同样的实验设定下性能上可以达到50亿的门电路计算。

#### Outline of Solution

1. 首先提出2-out-of-3的秘密分享方案以及一些重要的性质。
2. 描述半诚实下的乘法（AND）协议，并且证明本文依赖的重要性质：对于任意恶意敌手，乘法协议计算后，诚实方总是有一个有效的秘密份额。这个值要么等于输入的AND（半诚实），要么等于它的补（恶意）。
3. 介绍如何生成关联随机性（correlated randomness，$\mathcal{F}_{\mathrm{cr}}$）。
4. 使用$\mathcal{F}_{\mathrm{cr}}$以安全的方式生成$\mathcal{F}_{\mathrm{rand}}$，它给所以参与方提供随机份额。一个非常重要的性质是$\mathcal{F}_{\mathrm{cr}}$是非交互的，所以$\mathcal{F}_{\mathrm{rand}}$也是非交互的。这意味着可以借助$\mathcal{F}_{\mathrm{rand}}$以满足恶意安全的方式生成乘法元组中的$a,b$随机分享。
5. 接着可以利用$\mathcal{F}_{\mathrm{rand}}$生成安全抛硬币（互不信任的多方生成一个随机bit的方法）。其实用$\mathcal{F}_{\mathrm{rand}}$生成随机数后，我们需要交互才能公开一个数，如何保证交互的过程中保证恶意安全性？这一节将讲这一个问题。
6. 接下的部分会讲额外的Functionalities。做cut-and-choose的时候需要用于随机排列（random permutation，$\mathcal{F}_{\mathrm{perm}}$）。另外我们需要定义恶意安全下的shares（$\mathcal{F}_{\mathrm{share}}$）和open shares（$\mathcal{F}_{\mathrm{reconst}}$）。

接下来介绍这些子协议之间的关系，

<img src="/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/dependencies.png" style="zoom: 67%;" />

特别说明：本文工作仅适用于3方的情况，且仅仅是恶意安全下诚实多数情况的Secure with Abort。没有实现Fairness.

### BUILDING BLOCKS AND SUBPROTOCOLS

#### The Secret Sharing Scheme

**定义2-out-of-3分享**：为了分享一个比特$v = s_1 \oplus s_2 \oplus s_3$ 那么约定

- $P_1$的分享是$(t_1, s_1)$，这里$t_1 = s_3 \oplus s_1$
- $P_2$的分享是$(t_2, s_2)$，这里$t_2 = s_1 \oplus s_2$
- $P_3$的分享是$(t_3, s_3)$，这里$t_3 = s_2 \oplus s_3$

可以看到任意的一方是无法知道原始值$v$的，且任意的两方是可以还原$v$的，比如给定$(t_1, s_1), (t_2, s_2)$，$v = t_1 \oplus s_2$。

**Claim 1：当一方的秘密份额确定了，那么另外两方的份额也被确定！**

**定义Opening shares**：$v$的秘密分享为$\{(t_i, s_i)\}_{i=1}^{i=3}$，那么每一方$P_i$发送$t_i$给$P_{i+1}$，并且每一方$P_i$计算$v = s_i \oplus t_{i-1}$。表示为$\mathrm{open}([v])$

#### **Local operators for shares**：

- Addition $[v_1] \oplus [v_2]$：每一方$P_i$，假定$(t_i^1, s_i^1)$是$v_1$的秘密分享，$(t_i^2, s_i^2)$是$v_2$的秘密分享。那么每一方$P_i$对应计算$(t_i^1 \oplus t_i^2, s_i^1 \oplus s_i^2)$
- Multiplication by a scalar $\sigma \cdot [v]$：每一方$P_i$计算$(t_i \oplus \sigma, s_i \oplus \sigma)$
- Addition of a scalar $[v] \oplus \sigma$：每一方$P_i$计算$(t_i, s_i \oplus \sigma)$
- Complement $\bar{[v]}$ ：每一方$P_i$计算$(t_i, \bar{s_i})$

**Claim 2：令$[v_1],[v_2]$表示$v_1,v_2$的秘密分享，令$\sigma\in [0,1]$表示一个离散值。那么有下面的式子成立：**

1. $[v_1]\oplus [v_2] = [v_1 \oplus v_2]$
2. $\sigma\cdot [v_1] = [\sigma \cdot v_1]$
3. $[v_1]\oplus \sigma = [v_1 \oplus \sigma]$
4. $\bar{[v_1]} = [\bar{v_1}]$

**一致性（Consistency）**：一致性相对于诚实方来说的，在恶意者存在的情况下，在交互中它的行为是任意的，可以发送任意的数字使得诚实方得不到任意值的正确秘密分享。假设$P_i$是恶意方，当在Opening shares的时候，对于$P_{i+1}$有$v = t_i \oplus s_{i+1}$，对于对于$P_{i+2}$有$v = t_{i+1} \oplus s_{i+2}$。由于$P_i$是恶意的，则$P_{i+1}$中的$t_i$有可能是被篡改过的。另一方面，如果$t_i$是正确的，那么有$t_i = t_{i+1}\oplus t_{i+2}$。所以做一个代换：
$$
\begin{aligned}
s_{i+1} \oplus \underbrace{t_{i+1}\oplus t_{i+2}}_{t_i} &= t_{i+1} \oplus s_{i+2} \\
\Rightarrow s_{i+1} &= s_{i+2} \oplus t_{i+2}
\end{aligned}
$$
在诚实多数的约定下，$P_{i+1}$和$P_{i+2}$是诚实的，下面定义consistency

**Definition 1：令$P_1,P_2,P_3$分别持有$(t_1, s_1), (t_2, s_2), (t_3, s_3)$。令$P_i$为恶意方，如果$s_{i+1}=s_{i_2}\oplus t_{i+2}$成立，那么我们说shares是consistent的。** 

#### Computing AND Gates -- One Semi-Honest Corrupted Party

乘法（AND）的计算分为两阶段，假设$(t_1, s_1), (t_2, s_2), (t_3, s_3)$是$v_1$的分享，$(u_1, w_1), (u_2, w_2), (u_3, w_3)$是$v_2$的分享，其中$\alpha_1 \oplus \alpha_2 \oplus \alpha_3 = 0$。计算$v_1 \and v_2$

- Step1 -- 计算$\begin{pmatrix} 3 \\ 3  \end{pmatrix}$-sharing
  - $P_1$的计算$r_1 = t_1u_1 \oplus s_1w_1 \oplus \alpha_1$，发送$r_1$给$P_2$
  - $P_2$的计算$r_2 = t_2u_2 \oplus s_2w_2\oplus \alpha_2$，发送$r_2$给$P_3$
  - $P_3$的计算$r_3 = t_3u_3 \oplus s_3w_3 \oplus \alpha_3$，发送$r_3$给$P_1$
- Step2 -- 计算$\begin{pmatrix} 3 \\ 2  \end{pmatrix}$-sharing。这一步是为了保持形式的一致，另外计算都发生在本地
  - $P_1$存储$(e_1, f_1)$，其中$e_1 = r_3 \oplus r_1, f_1 = r_1$
  - $P_2$存储$(e_2, f_2)$，其中$e_2 = r_1 \oplus r_2, f_2 = r_2$
  - $P_3$存储$(e_3, f_3)$，其中$e_3 = r_2 \oplus r_3, f_3 = r_3$

**Lemma 1：如果$[v_1],[v_2]$是consistent的，并且$[v_3]$是上述乘法协议在一个恶意者存在的情况下关于$[v_1],[v_2]$的结果，那么$[v_3]$要么是$v_1v_2$的一致性分享，要么是$v_1v_2\oplus 1$的一致性分享。**

Proof：如果恶意方遵从协议的执行，那么$[v_3]$是$v_1v_2$的一致性分享。不失一般性，假设$P_1$为恶意者，它唯一可以偏离协议规则的方式是将原本发送的$r_1$改为发送$r_1\oplus 1$。$P_2$的$(e_2, f_2) = (r_1 \oplus r_2 \oplus 1, r_2)$。$P_3$不会受到影响，根据Consistent的定义，$f_3 \oplus e_3 = r_3 \oplus (r_3\oplus r_2) = r_2$。此外，它也是$v_1v_2\oplus 1$的分享，因为$f_3 \oplus e_2 = r_3 \oplus (r_1 \oplus r_2 \oplus 1) = v_1v_2 \oplus 1$。

#### Generating Correlated Randomness -- $\mathcal{F}_{\mathrm{cr}}^1 / \mathcal{F}_{\mathrm{cr}}^2 $

协议强依赖于随机有关联的bits。因此定义两种类型的correlated randomness。

- Type 1：考虑一个理想的方法$\mathcal{F}_{\mathrm{cr}}^1$，它可以随机的选择$\alpha_1,\alpha_2,\alpha_3\in \{0,1\}$满足$\alpha_1 \oplus \alpha_2 \oplus \alpha_3 = 0$，并且发送$\alpha_i$给$P_i$。
- Type 2：考虑一个理想的方法$\mathcal{F}_{\mathrm{cr}}^2$，同样可以随机的选择$\alpha_1,\alpha_2,\alpha_3\in \{0,1\}$，然后将$(\alpha_1,\alpha_2)$发送给$P_1$，$P_2,P_3$类推。

<img src="/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/2.5.png" style="zoom:80%;" />

**Proposition 1：如果$\mathcal{F}$是伪随机函数，那么协议2.5满足诚实多数下的恶意安全**

#### Generating Shares of a Random Value -- $\mathcal{F}_{\mathrm{rand}}$

描述如何生成一个随机数$v$的sharing，满足各个参与方对其真实值未知。

<img src="/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/2.9.png" style="zoom:80%;" />

观察到$t_1 \oplus t_2 \oplus t_3 = 0$。可以定义$v = s_1\oplus t_3 = s_2\oplus t_1 = s_3\oplus t_2 = r_1 \oplus r_2 \oplus r_3$

**Proposition 2： 协议2.9满足诚实多数下的恶意安全**

#### Coin Tossing -- $\mathcal{F}_{\mathrm{coin}}$

现在提出一种高效的三方抛硬币协议，它在存在一个恶意者的情况下也是安全的。想法是调用$\mathcal{F}_{\mathrm{rand}}$生成随机数的秘密分享，然后open这个结果得到随机数$v$。但是这是有问题的，因为在恶意者而在的情况下，有可能在open的时候会发送不正确的值导致无法成功还原$v$。一个解决办法就是各个参与方将还原的$v$互相比较。如果一致则接受它，如果不一致则输出Abort。

<img src="/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/2.11.png" style="zoom:80%;" />

**Proposition 3： 协议2.11满足诚实多数下的恶意安全**

#### Random Shuffle -- $\mathcal{F}_{\mathrm{perm}}$

作为一个辅助协议，需要计算一个array的随机排列

<img src="/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/2.13.png" style="zoom:80%;" />

**Proposition 4： 协议2.13满足诚实多数下的恶意安全**

#### Reconstruct a Secret to One of the Parties -- $\mathcal{F}_{\mathrm{reconst}}$

下面展示以一种安全的方式打开一致性$[v]$为$v$。区别于前面提到的 open procedure有两点不同：1）仅仅是一方得到输出；2）这个open procedure不保证正确性（correctness）。这里的Reconstruct保证一方要么得到正确的输出要么得到Abort。

<img src="/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/2.16.png" style="zoom:80%;" />

#### Robust Sharing of a Secret -- $\mathcal{F}_{\mathrm{share}}$

$\mathcal{F}_{\mathrm{share}}$作为一个子协议被用来将参与者的输入进行秘密分享。

<img src="/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/2.19.png" style="zoom:80%;" />

#### Triple Verification With Opening

在主协议中，参与方采用两阶段的方法生成乘法元组：

- Step 1：参与方两次调用$\mathcal{F}_{\mathrm{rand}}$生成随机数$[a],[b]$的秘密分享
- Step 2：参与方调用半诚实乘法协议计算$[c] = [a]\cdot [b]$

元组使用打开验证正确性基于一个事实：如果shares是一致性的，并且$c\ne ab$，那么诚实方之一一定能检测出来的。这个协议之所以with opening是因为$a,b,c$被揭露了。

<img src="/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/2.22.png" style="zoom:80%;" />

#### Triple Verification Using Another (Without Opening)

通过打开验证的方式会使得$a,b,c$被披露导致不再可用。这一节介绍不使用Opening的方式验证乘法的正确性。主要的想法是。给定$x,y,z$和$a,b,c$。参与者打开$\rho = x\oplus a$和$\sigma = y\oplus b$（没有泄露$x,y$的值，因为$a,b$是随机的）下面会证明，如果$x,y,z$，$a,b,c$中之一是正确的，另一个不是正确的（比如，$z\neq x\cdot y$但是$c = a\cdot b$），那么有$z+c+\sigma \cdot a + \rho \cdot b + \rho\cdot \sigma = 1$。因此可以通过披露这个值来进行验证。

<img src="/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/2.24.png" style="zoom:80%;" />

**Lemma 1：如果$([a], [b], [c])$是一个正确的乘法元组并且 $([x], [y], [z])$是一致性的分享，但是$([x], [y], [z])$不是乘法元组。那么协议2.24中所有的诚实方会输出Abort。**

Proof：令$P_i$是恶意方，有$z\neq x\cdot y$但是$c = a\cdot b$。如果$z\neq x\cdot y$那么有$z=xy\oplus 1$。根据$\rho = x\oplus a$和$\sigma = y\oplus b$的定义有
$$
\begin{aligned}
[z] \oplus[c] & \oplus \sigma \cdot [a] \oplus \rho \cdot [b] \oplus \rho\cdot \sigma \\
&=[xy\oplus 1] \oplus [c] \oplus (y \oplus b)[a] \oplus (x \oplus a)[b] \oplus (x \oplus a)(y \oplus b) \\
&=[xy\oplus 1] \oplus [c] \oplus [(y \oplus b)a] \oplus [(x \oplus a)b] \oplus (xy \oplus ay \oplus xb \oplus ab) \\
&=[xy\oplus 1 \oplus [c] \oplus (y \oplus b)a \oplus (x \oplus a)b \oplus xy \oplus ay \oplus xb \oplus ab]\\
&=[1]
\end{aligned}
$$

### SECURE GENERATION OF MULTIPLICATION TRIPLES -- $\mathcal{F}_{\mathrm{triples}}$

我们使用三方协议生成正确的乘法元组。主要想法是参与方使用$\mathcal{F}_{\mathrm{rand}}$生成许多$[a_i],[b_i]$，然后使用半诚实的乘法协议计算$c_i = a_i\cdot b_i$。为了满足恶意安全性，我们使用cut-and-choose来检测这些元组的正确性。

<img src="/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/3.2.png" style="zoom:80%;" />

### SECURE COMPUTATION OF ANY FUNCTIONALITY

这一小节说明如何利用前面介绍的子协议计算任意函数$f$。

![](/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/4.2.png)

### EFFICIENCY AND COMPARISON

本文与P. Mohassel et al.提出的方法（CCS‘15）进行了比较

![](/home/karrylee/Desktop/工作相关/Paper/High-Throughput/Pic/t4.png)

