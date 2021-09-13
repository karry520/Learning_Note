## ABY$^3$: A Mixed Protocol Framework for Machine Learning 

 Payman Mohassel and Peter Rindal (CCS'18)

### ABSTRACT

本文主要包括以下几个方面：

- 设计实现了一个通用的基于三方服务器的隐私保护机器学习框架，基于此，设计实现了新的线性回归、逻辑回归、神经网络模型
- 提出了新的完整的算术电路、布尔电路和Yao电路之间的转化协议
- 提出了新的三方秘密分享下的定点十进制小数乘法、并设计了计算分断线性多项式函数的协议
- 本文考虑了semi-honest和malicious两种情况

### INTRODUCTION

本文探索基于三方服务器的隐私保护机器学习框架，改进前人的工作所遇到的挑战有以下几个方面

1. 现有的3PC技术只能适应$\mathbb{Z}_{2^k}$环上的运算，而ML中的计算或者中间参数都是以小数的形式存在，它不能直接应用模运算的代数系统上，现有的解决方案有：
   - 将小数乘上一个大因子使其成为整数，同时使用一个大的模数来避免wrap around，这样的弊端就是当计算大量浮点乘法的时候将不满足结果的正确性，同时采用了更大的模数增加了计算和能信的复杂度
   - 使用基于布尔电路和混乱电路的定点小数乘法方法，但是其效率将又大大增加
2. ML中的涉及到大量的算术计算与布尔运算，往往使用混合协议的方式比使用单一协议的方法要更高效，但是现有的转换协议已成为主要的性能瓶颈

#### Contributions

1. 设计实现了一种新的秘密分享下的定点数乘法协议，其吞吐量和延迟比优化的布尔电路上的实现方法分别提升了$50\times$和$24\times$。此外该协议不仅在malicious下安全而且能扩展到任意多方计算
2. 提出新的基于三方的算术、布尔与Yao之间的转化协议，该协议能够抵抗malicious的情况
3. 提出了delayed re-share技术优化技术，在很大程度上减少向量操作中的通信复杂度，并且提出基于三方的通用扩展OT协议来计算分断多项式函数的方法
4. 构建了semi-honest和malicious两种设定下的协议模块
5. 实验表明比对于神经网络任务比secureML快$55000\times$，预测任务比Chameleon快270倍......

### OUR FRAMEWORK

#### Secret Sharing

##### Arithmetic sharing 

- 2-out-of-3 秘密分享

  1. $P_0:(x_0, x_1), (y_0, y_1)$
  2. $P_1:(x_1, x_2), (y_1, y_2)$
  3. $P_2:(x_2, x_0), (y_2, y_0)$

  常见类型的计算

  - $\langle x + y\rangle  := (x_0 + y_0, x_1 + y_1, x_2 + y_2)$

  - $\langle x\rangle \pm c:= (x_0\pm c, x_1, x_2)$

  - $\langle cx\rangle:=(cx_0, cx_1, cx_2)$

  - 处理三方分享下的乘法先本地计算交叉项后，为保持形式的统一需要进行re-sharing的过程（将$z_i$发送给$P_{i+1}$）

    $$\begin{aligned}\langle xy\rangle &=(x_0+x_1+x_2)(y_0+y_1+y_2)\\&=(x_0y_0+x_0y_1+x_1y_0):P_0\to z_0\\&+(x_1y_1+x_1y_2+x_2y_1):P_1\to z_1\\&+(x_2y_2+x_2y_0+x_0y_2):P_2\to z_2\end{aligned}$$

- 优化：令$0 = a_0+a_1+a_2, 0 = b_0+b_1+b_2$，那么可以定义sharing为$\langle x\rangle:= (x_0 + a_0, a_1, a_2)$

- 如何生成$0=a_0+a_1+a_2$

  - $P_0,P_1$共有一个随机数种子$k_0$，$P_1,P_2$共有一个随机数种子$k_1$，$P_2,P_0$共有一个随机数种子$k_2$
  - 给定一个nonce和伪随机函数 $PRG$
    - $P_0\to a_0 = PRG_{k_2}(nonce)-PRG_{k_0}(nonce)$
    - $\dots$
  - $k_i$可以在初始化阶段生成，$k_{i+1}=PRG(k_i)$，所以不需要通信

##### Binary sharing 



##### Yao's sharing



#### Fixed-point Arithmetic

假设$\langle x^{'}\rangle = \langle y\rangle \langle z\rangle \in \mathbb{Z}_{2^k}$表示是$y,z$在秘文下的乘法，最后将结果以秘密分享的形式表示为$\langle x^{'}\rangle = (x^{'} + r^{'}, -r^{'})$，都是定点数表示，其正确结果需要除以$2^d$（小数部分用$d$-bit表示），即是说正确的结果应是$\langle x\rangle = \langle x^{'} / 2^d\rangle$。现有的定点截断方法采用的是将$\langle \tilde{x}\rangle := (\frac{x^{'} + r^{'}}{2^d}, -\frac{r^{'}}{2^d})$，也就是说直接在秘文下进行截断，可以证明还原后其值以可忽略的概率使得误差只有最后1-bit。

##### 这种方式存在两个问题

1. 精确性问题：1）截断直接抹掉了第d位处的进位；2）如果$\langle \tilde{x}\rangle$的分享值最高符号位与原数的最高符号位不一致（比如一个负数其秘密分享后最高符号位可能都是0，那么截断后前面$d+1$位均是0）那么会导致还原错误
2. 针对问题1，可以限制$|x^{'}|< 2^l \ll 2^k$，但是不能扩展到3-PC的情况

##### 基于三方的截断方法一$\Pi_{trunc1}$

令每方有$\langle x^{'}\rangle=\langle y\rangle \langle z\rangle$的2-out-of-3分享份额，现在的目标是计算截断$\langle x\rangle = \langle x^{'} /2^d\rangle $（转为两方的设置下来做）

- 定义在$P_1,P_2$、之间的2-out-of-2秘密分享$(x_1^{'}, x_2^{'}+x_3^{'})$，然后本地截断$(x_1^{'} / 2^d, (x_2^{'}+x_3^{'})/ 2^d)$， 最后的结果表示为$\langle x \rangle := (x_1^{'} / 2^d, (x_2^{'}+x_3^{'})/ 2^d - r, r)$，然后做一次re-sharing的步骤

##### 基于三方的截断方法二$\Pi_{trunc2}$

回顾一下三方秘密分享的步骤为：1）计算$\langle x^{'}\rangle$的3-out-of-3秘密分享；然后，2）进行2-out-of-3秘密分享。$\Pi_{trunc2}$合并计算过程，从而只需要一轮通信，首先三方使用bool 电路上的减法生成$\langle r\rangle = \langle r^{'} / 2^d\rangle$

- Step1秘密分享$\langle x^{'} - r^{'} \rangle$

- Step2定义$\langle x\rangle := (x^{'} - r^{'}) / 2^d + \langle r\rangle$，因为这里$\langle r\rangle = \langle r^{'} / 2^d\rangle$

  <img src="C:\Users\86188\Desktop\工作相关\job\Article\ABY3\trunc2.PNG" style="zoom:60%;" />

#### Vectorized Multiplication

对于内积计算$\overrightarrow{x}\cdot \overrightarrow{y}:=\sum_{i=1}^nx_iy_i$，按照原始的做法将需要调用$n$次安全乘法协议，需要$O(n)$次通信。本文做了优化，只需要$O(1)$次通信和一次trunctation-pair $(\langle r^{'}\rangle, \langle r\rangle)$。

- $$\langle \overrightarrow{x}\rangle \cdot \langle \overrightarrow{y}\rangle :=\mathrm{reveal}((\sum_{i=1}^n\langle x_i\rangle \langle y_i\rangle) + \langle r^{'}\rangle ) / 2^d - \langle r\rangle $$

​	也就是，首先每方本地计算3-out-of-3的$\langle x_i\rangle \langle y_i\rangle$，然后求和，求掩码，截断，最后2-out-of-3做reshared

#### Share Conversions

##### Bit Decomposition $\langle x\rangle^A \to \langle \overrightarrow{x}\rangle ^B$

- 基本思路是将 $\langle x\rangle^A = (x_1, x_2, x_3)$作为3PC boolean circuit的输入，进行布尔加法运算，最后的结果就是 $\langle x\rangle^A$的布尔分享即$\langle \overrightarrow{x}\rangle ^B$，本文做了一些电路的优化，仅需要$1 + \log k$轮通信

##### Bit Extraction  $\langle x\rangle^A \to \langle \overrightarrow{x}[i]\rangle ^B$

- 该协议计算的是 $\langle x\rangle^A $将算术分享中的第$i$个位置的值转化为boolean 分享$\langle \overrightarrow{x}[i]\rangle ^B$
- 基本思路同上（相当于只需要剩下的$i$-bit的分享值就可以计算出来），不过只需要$O(i)$个AND门电路和$O(\log i)$通信轮数

##### Bit Composition 

- 使用Bit Decomposition同样的布尔电路，输入是$\langle -x_2\rangle^B,\langle -x_3\rangle^B$，计算$\langle x_1\rangle^B = \langle x\rangle^B + \langle -x_2-x_3\rangle^B$，再把二进制表示的布尔值表示成算术值，因此$\langle x\rangle^A = (x_1, x_2, x_3)$

##### Yao to Binary

- 使用Yao's share中key的$p_x = k_x^0[0]$（permutation bit），符合Binary share的定义，因此取$\langle x\rangle^B = (x\oplus p_x\oplus r, r, p_x)$

##### Binary to Yao 

- 使用Joint Yao Input直接对$\langle x\rangle^B$进行Yao's 秘密分享

##### Yao to Arithmetic  

- 通过Yao's circuit 3PC来计算一个加法计算出$\langle x\rangle^Y$，首先Parties 1,2采样$x_2\leftarrow \mathbb{Z}_{2^k}$，Parties 2,3采样$x_3\leftarrow \mathbb{Z}_{2^k}$，然后使用Yao's circuit来计算$\langle x_1\rangle^Y = \langle x\rangle^Y - \langle x_2\rangle^Y - \langle x_3\rangle^Y$。最后就得到了$\langle x\rangle^A = (x_1, x_2, x_3)$

##### Arithmetic to Yao 

- 直接将$\langle x_i\rangle^Y$使用jointly input 转化为$\langle x_i\rangle^Y$

#### Computing $\langle a\rangle^A\langle b\rangle^B = \langle ab\rangle^A$

现在已经可以在任意的share上进行转化了，不过对于特定的计算任务仍然可以继续优化，本文提出了一种$\langle a\rangle^A\langle b\rangle^B = \langle ab\rangle^A$混合计算协议，被应用在分断线性或者多项式函数来近似非线性激活函数的计算中

- 首先提出了Three-Party OT，被形式化定义为$((m_0, m_1), c, c) \to (\perp, m_c, \perp)$，分三个角色：Sender、Receiver、Helper。

  - Sender和Helper随机生成$w_0, w_1\leftarrow \{0,1\}^k$
  - Sender发送$m_0\oplus w_0,m_1\oplus w_1$给Receiver
  - Helper发送$w_c$给Receiver，那么Receiver就可以得到$m_c = m_c\oplus w_c\oplus w_c$

- 基于此，首先来计算$ a\langle b\rangle^B = \langle ab\rangle^A$，其中$a\in \mathbb{Z}_{2^k}$被公开，$b\in {0,1}$被Boolean分享

  - $P_3$（Sender）采样$r\leftarrow \mathbb{Z}_{2^k}$，定义$m_i = (i\oplus b_1\oplus b_3)a - r, i\in {0,1}$
  - $P_2$（Receiver）取到$m_{b_2} = (b_2\oplus b_1\oplus b_3)a - r = ba-r$
  - $P_1$（Helper）已知$b_2$按照3PC-OT的流程将掩码发给$P_2$

  此时，使用zero sharing 得到$(s_1, s_2, s_3)$，最一次resharing 后得到$\langle c\rangle^A = \langle ab\rangle^A = (s_1 + r, ab - r + s_3, s_3)$

- 发现上述$a$也可以是算术分享的形式，因此可以计算$\langle a\rangle^A\langle b\rangle^B = \langle ab\rangle^A$

  - $\langle a\rangle^A\langle b\rangle^B = a_1\langle b\rangle^B + (a_2+a_3)\langle b\rangle^B$
  - $P_1$扮演Sender计算第1项，$P_3$扮演Sender计算第2项即可

- 总的每方需要1轮共计4k-bit的通信

#### Polynomial Piecewise Functions

令$f_1,\dots,f_m$表示$f(x)$的分段函数，$f(x) = f_i(x)$当$c_{i-1}<x \le c_i$时成立。因此首先判定$x$落在什么区间（即计算$\langle x\rangle < c$），得到$b_1,\dots,b_m\in\{0,1\}$满足$b_i=1\leftrightarrow c_{i-1} < x \le c_i$，那么$f(x) = \sum_ib_if_i(x)$。

