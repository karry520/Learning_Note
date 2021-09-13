## ABY2.0: Improved Mixed-Protocol Secure Two-Party Computation

Arpita Patra, Thomas Schneider, Ajith Suresh, Hossein Yalame (USENIX Security'21)

### ABSTRACT

本文主要包括以下几个方面：

1. 提出一种目前最高效的2PC混合协议框架，该框架扩展了乘法计算，对于多个数相乘，在线通信也仅需要一轮
2. 同时构建了一些原语模块（离散内积、矩阵乘法、比较、maxpool和相等判断...），其最大的改进在于在线计算方面，对于离散内积计算的在线阶段只需要1轮，且通信量与向量维度无关这是第一次做到的
3. 演示了ABY2.0的四种应用：1）AES S-box; 2）Circuit-based PSI; 3）Biometric Matching; 以及4）PPML

### INTRODUCTION

现有MPC协议可以大致分为两类：1）低延迟（通常基于GC实现）；2）高通量（通常基于SS实现）。为实现性能的折中或者满足实际的需要，通常采用基于：1）Arithmetic，2）Boolean，3）Yao混合的方式来完成一个任务。为满足实时性要求，一些工作考虑将一部分工作独立地放在预计算阶段来做（比如生成Beaver乘法元组），进而在线阶段就能实时地处理参与方的安全计算任务。本文的目标是：**基于2PC实现对在线阶段的通信和交互轮数的优化**。

#### Contributions

1. 提出高效的2PC安全计算协议，其中乘法计算在线阶段仅需要$2l$-bit通信和1轮交互（多个数相乘也是这样）

   <img src="C:\Users\86188\Desktop\工作相关\job\Article\ABY2.0\aby2.0-1.PNG" style="zoom:70%;" />

2. 提出新的混合转换协议，对比ABY，ABY2.0在转化协议中由于没有在线阶段的OT操作，因此可以将通信轮数从2次减少到1次

   <img src="C:\Users\86188\Desktop\工作相关\job\Article\ABY2.0\aby2.0-2.PNG" style="zoom:70%;" />

3. 构建了安全协议模块

   - 离散内积：其通信量与向量的维度无关，这在2PC中第一次做到
   - 矩阵乘法：扩展了乘法协议来计算矩阵乘法
   - 电路深度优化：Parallel Prefix Adder(并行前缀加法器，PPA)被用来优化2个输入的AND门深度，本文提出2，3，4个输入的AND门计算
   - 提出优化的比较运算模块
   - 3元素中求最大
   - 相等判断

4. 演示了四种场景的应用

### 2PC in Arithmetic, Boolean and Yao's World

#### 2PC in Arithmetic World

##### High-level Overview of Our 2PC over Ring 

从Beaver元组的乘法中进行改进，首先介绍Beaver乘法，令$\Delta_a = a + \delta_a, \Delta_b= b + \delta_b, \Delta c = c + \delta_c$，其中$\delta_c = \delta_a \delta_b$

$$ \begin{aligned}a\cdot b &= ((a + \delta_a) - \delta_a)((b + \delta_b) - \delta) \\ &= (a + \delta_a)(b + \delta_b) - (a + \delta_a)\delta_b - (b + \delta_b)\delta_a +\delta_a\delta_b \end{aligned}$$

<img src="C:\Users\86188\Desktop\工作相关\job\Article\ABY2.0\aby2.0-3.PNG" style="zoom:80%;" />

可以看到，Beaver's方法分享的是$(a_i,[\delta_a]_i)$需要先还原$\Delta_a,\Delta_b$然后再计算。而ABY2.0采用的是分享$(\Delta_a,[\delta a]_i)$，因此就少了还原$\Delta_a,\Delta_b$这一步，进而少了一轮通信，最后为保证形式的统一，需要对$[\Delta_c]i$进行交换得到$\Delta_c, [\delta_c]_i$。可以看到，在线阶段仅需要交换$[\Delta_c]_i$，所以只需要一轮交互，通信量为$2l$-bit

##### Sharing Semantics

- $[\cdot]$-sharing：定义为将$v\in \mathbb{Z}_{2^l}$的分享在Party $P_i$，$i\in \{0,1\}$，使其满足$v = [v]_0 + [v]_1$
- $\langle\cdot \rangle$-sharing：定义为将$v\in \mathbb{Z}_{2^l}$的分享在Party $P_i$，$i\in \{0,1\}$，如果存在$\delta_v,\Delta_v\in \mathbb{Z}_{2^l}$，使其满足：1）$\delta_v$按$[\cdot]$-sharing在$P_i$之间；2）$\Delta_v = v+\delta v$； 3）$\Delta_v$在$P_0,P_1$之间公开

##### Protocols

- Sharing Protocol（SHARE）：预计算阶段$P_i$随机采样$[\delta_v]_i$，然后与$P_{1-i}$一起随机采样$[\delta_v]_{1-i}$，这样$P_i$知道$\delta_v = [\delta_v]_0+[\delta_v]_1$，计算$\Delta_v = v + \delta_v$发送给$P_{1-i}$。（$P_i$分享$v$的时候，$P_{1-i}$不知道$\delta_i$，故无法知道$v$的值）

- Reconstruction Protocol（REC）：直接交换缺失的$[\delta_v]_i$给$P_{1-i},i\in {0,1}$，计算$v = \Delta_v - \delta_v$即可

- Multiplication Protocol：

  $$\begin{aligned}\Delta_y &= y + \delta_y = (\Delta_a - \delta_a)(\Delta_b - \delta_b) + \delta_y \\ &= \Delta_a\Delta_b - \Delta_a\delta_b - \Delta_b\delta_a + \delta_a\delta_b + \delta_y \end{aligned}$$

<img src="C:\Users\86188\Desktop\工作相关\job\Article\ABY2.0\aby2.0-4.PNG" style="zoom:80%;" />

1. 预计算阶段，首先计算$\delta_a\delta_b$的$[\cdot]$-sharing（记作setupMULT）。输入$[\delta_a]_0,[\delta_a]_1,[\delta_b]_0,[\delta_b]_1$，输出$[\delta_a\delta_b]_0,[\delta_a\delta_b]_1$。有两种方式

   $$\delta_a\delta_b = ([\delta_a]_0+[\delta_a]_1)([\delta_b]_0+[\delta_b]_1)=[\delta_a]_0[\delta_b]_0+[\delta_a]_0[\delta_b]_1+[\delta_a]_1[\delta_b]_0+[\delta_a]_1[\delta_b]_1$$

   **1）OT based**（仅展示如何计算交叉项$[\delta_a]_0[\delta_b]_1$）

   - $j\in \{0, \dots, l-1\}$，使用cOT
   - $P_0$输入关联随机函数$f_j(x) = x + 2^j[\delta_a]_0$，获得$(m_{j,0}=r_j,m_{j,1}=r_j+2^j[\delta_a]_0)$
   - $P_1$取$[\delta_b]_1$的第$j$位作为选择位$b_j$，得到$m_{j,b_j}$
   - $[\cdot]$-sharing定义为$[([\delta_a]_0[\delta_b]_1)]_0 = \sum_{j=0}^{l-1}(-r_j),[([\delta_a]_0[\delta_b]_1)]_1 = \sum_{j=0}^{l-1}m_{j,b_j}$

   **2）HE-based**

   - $P_0$生成公钥$pk_0$，加密$[\delta_a]_0, [\delta_b]_0$发送给$P_1$
   - $P_1$使用$pk_0$加密$[\delta_a]_1, [\delta_b]_1$，并生成一个随机数$r\in \mathbb{Z}_{2^l}$，然后同态计算$v = [\delta_a]_0[\delta_b]_1+[\delta_a]_1[\delta_b]_0 - r$，发送结果给$P_0$
   - $P_0$解密$v$

2. 在线阶段，交换$[\Delta_y]_i$给$P_{1-i}$，双方得到$\Delta_y$，进而形成统一的形式$(\Delta_y,[\delta_y]_i)$

##### Multi-Input Multiplication Gates

本文扩展了乘法计算（以3-Input为例），优化了在线阶段的通信和通信轮数

$$\begin{aligned}\Delta_y &= abc + \delta_y = (\Delta_a - \delta_a)(\Delta_b - \delta_b)(\Delta_c - \delta_c)\\&=\Delta_{abc}-\Delta_{ab}\delta_c - \Delta_{bc}\delta_a - \Delta_{ac}\delta_b +\Delta_{a}\delta_{bc} +\Delta_{b}\delta_{ac} +\Delta_{c}\delta_{ab} - \delta_{abc} + \delta_y\end{aligned}$$

只需要在预计算阶段计算$\delta_{ab},\delta_{bc},\delta_{ac},\delta_{abc}$，因为带$\Delta$的项都是公开的，所以仍然只需要$2l$-bit通信量和1轮通信

#### 2PC in Boolean World

Boolean 运算与Arithmetic运算基本一致，相当于$\mathbb{Z}_{2^1}$上的秘密分享，只需要把加法替换为XOR，乘法替换为AND运算

##### Negation Protocol 

对1bit 的$u$进行B-sharing为$\langle u\rangle^B = ([\delta_u, \Delta_u])$，NOT协议生成$u$的否运算$\bar{u}$，直接在本地令$\Delta_{\bar{u}}=1\oplus \Delta_u,[\delta_{\bar{u}}]=[\delta_u]$

#### 2PC in Yao World

ABY2.0中的Yao's sharing跟ABY下的Yao's sharing一样，对于$v$在线路$u$上的分享值可以定义为：

- $P_0$扮演garbler生成zero-key：$K_u^0$，使用到Free-XOR和point-and-permute优化（$K_u^0[0]=1$），所以$K_u^1=K_u^0\oplus R\{0,1\}^k$
- $P_1$扮演evaluator，并使用$\mathrm{cOT}_k^1$从$P_0$处取得$K_u^v$

$\langle \cdot \rangle$-sharing一个比特$v$，被表示为SHARE($P_i,v$)

- 如果$P_i=P_0$，那么$P_0$直接将$k_u^v$发送给$P_1$
- 如果$P_i=P_1$，那么$P_1$使用$\mathrm{cOT}_k^1$从$P_0$处取得$K_u^v$

### Mixed Protocol Conversions

#### Standard Conversions

ABY的转化协议在线阶段需要使用到OT，ABY2.0则不需要这是一个区别。

##### 1. Y2B: 给定bit $u\in \{0,1\}$的$\langle \cdot \rangle^Y$-sharing，目标是转化成等价的Boolean sharing

观察到Yao's sharing的最低有效位LSB满足Boolean sharing的定义，所以可以用LSB$(\langle u\rangle_i^Y)$来表示，从而实现了Y2B的转化

##### 2. B2Y: 给定bit $u\in \{0,1\}$的$\langle \cdot \rangle^B$-sharing，目标是转化成等价的Yao's sharing

- $P_i$首先本地计算$u_i=(1-i)\Delta_u \oplus [\delta_u]_i$，容易验证$u = u_0\oplus u_1$
- $P_i$再调用SHARE($P_i,u_i$)生成$\langle u_i\rangle^Y$
- 使用Free-XOR技术，本地计算$\langle u\rangle^Y = \langle u_0\rangle^Y\oplus \langle u_1\rangle^Y$

##### 3. A2Y: 给定$\langle v \rangle^A$，目标是转化成等价的Yao's sharing

类似于B2Y，先还原再SHARE($P_i,v_i$)

- $P_i$首先本地计算$v_i=(1-i)\Delta_v \oplus [\delta_v]_i$，容易验证$v = v_0+ v_1$
- 然后$P_0$构建一个混乱加法电路，通过混乱电路计算$v$，最后的结果即是$v$的Yao's sharing

##### 4. Y2A: 给定$\langle v \rangle^Y$，目标是转化成等价的Arithmetic  sharing

同ABY中的Y2A方法

- 预计算阶段，$P_0$随机采样$r\in \mathbb{Z}_{2^l}$，然后对$r$进行SHARE$^Y(P_0, r)$和SHARE$^A(P_0,r)$生成$\langle r\rangle^Y,\langle r\rangle^A$，并行地，$P_0$生成一个混乱加法电路，发送给$P_1$
- 在线阶段，$P_1$对$\langle r\rangle^Y,\langle v\rangle^Y$作为输入，使用混乱电路计算$\langle v+r\rangle^Y$。解码后$P_1$获得$v+r$明文，再对其进行SHARE$^A(P_1,v+r)$得到$\langle v+r\rangle^A$，与$P_0$的$\langle r\rangle^A$构成算术分享

##### 5. A2B: 给定$\langle v \rangle^A$，目标是转化成等价的Boolean sharing

同Y2A方式，区别在于构建的不是混乱加法电路而是Boolean 加法电路，其它过程是一样的

##### 6. Bit2A: 对于1bit的$v\in \{0,1\}$，给定它的Boolean sharing $\langle v\rangle^B$目标是转化成等价的Arithmetic  sharing

令$v^a$被看作$v$在$l$-bit环上的表示。那么$v=v_0\oplus v_1$可以写为$v^a=v_0^a+v_1^a-2v_0^av_1^a$，注意到$v^a=(\Delta_v\oplus \delta_v)^a=\Delta_v^a+\delta_v^a-2\Delta_v^a\delta_v^a$

- 预计算阶段交互生成$\delta_v^a$的$[\cdot]$-sharing
- 在线阶段，$P_i,i\in\{0，1\}$本地计算$[v^a]_i=i\cdot \Delta_v^a + (1 - 2\Delta_v^a)\cdot [\delta_v^a]_i$，并且执行SHARE$^A(P_i,[v^a]_i)$生成$\langle v^a\rangle^A$

其中$[\delta_v^a]$的生成同ABY中B2A的方法一样，区别在于放在了预计算阶段来做了

##### 7. B2A: 将 $v\in \mathcal{Z}_{2^l}$从$\langle v\rangle^B$-sharing转化成等价的Arithmetic  sharing

$l$次调用Bit2A协议即可

#### Special Conversions

有了Bit2A协议可以定制一些特殊的转化协议

<img src="C:\Users\86188\Desktop\工作相关\job\Article\ABY2.0\aby2.0-5.PNG" style="zoom:80%;" />

### Building Blocks for Applications

