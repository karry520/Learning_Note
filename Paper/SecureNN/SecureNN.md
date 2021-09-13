## SecureNN: 3-Party Secure Computation for Neural Network Training 

Sameer Wagh, Divya Gupta, and Nishanth Chandran (PPET'19)

### ABSTRACT

本文构建了新的基于三方安全计算协议NN计算模块，包括矩阵乘法、卷积、ReLU、Maxpool、正则化等。本文的工作在以下三个方面优于其它的隐私保护NN方法：

1. Scalability：是第一个用于安全计算卷积神经网络训练的研究
2. Performance：在安全预测中效率优于（SecureML、MiniONN、Chameleon、Gazelle等）$6\times-113\times $。总的时间快$2-4\times$。在安全训练任务中，分别比2-Party、3-Party的SecureML快$79\times,3\times$​​。
3. Security：前人的相关工作只提供了半诚实的安全性保证，本文是第一个在NN的训练和预测任务中考虑恶意敌手存在的安全的工作

### TECHNICAL OVERVIEW

神经网络算法中包含着线性运算（如，矩阵乘法、卷积...）和布尔运算（如，ReLU、Maxpool和它的导数...），为了使这些计算兼容因此会引入一些转换协议，但基于GC的布尔运算其通信开销特别大，是安全参数长度的许多倍。本文整个方案只使用了算术加法秘密分享，没有涉及到布尔电路计算，因此效率得到了有效的提升。

#### Protocols Structure 

![](/home/karrylee/Desktop/工作相关/╣ñ╫≈╧α╣╪/job/Article/SecureNN/SecureNN-2.PNG)

#### Supporting Protocols

##### Matrix Multiplication 

乘法采用Beaver三元组的方式，由P2辅助生成三元组，以秘密分享的形式分发给$P_0$和$P_1$。（为减少通信，$P_2$采用分发随机数种子的方式，而不是直接传随机数，最后计算好$[c_1]_i$后给$P_1$）

<img src="C:\Users\86188\Desktop\工作相关\job\Article\SecureNN\SecureNN-4.PNG" style="zoom:80%;" />

##### Select Share

Select Share协议的基本要求是，用一个比特$a$​来选择分享份额，即假设$P_0,P_1$​持有$x,y$​的秘密分享份额，当$a=0$​时选择$x$​，反之$y$​​。

<img src="C:\Users\86188\Desktop\工作相关\job\Article\SecureNN\SecureNN-5.PNG" style="zoom:80%;" />

主要思路是：$x,y$中的选择可以表示为$(1-a)\cdot x + a \cdot y = x + a \cdot (y-x)$，那么协议可以计算$\langle z\rangle _j^L = \langle x\rangle _j^L +  \langle a\rangle_j^L\cdot (\langle y\rangle _j^L - \langle x\rangle _j^L)$​，先本地计算减法然后调用一 次乘法协议即可​，最后增加盲化值$u_j$。

##### Private Compare 

整个方案的核心在非线性算子的计算。以激活函数为突破口。$\mathrm{ReLU}(x)$​​​的计算关键在于判断$x$​​​的正负，也就是看$\mathrm{MSB}(x)$​​​。而$\mathrm{MSB}(x)$​​​需要做比特提取（bit extraction），它没有求$\mathrm{LSB}(x)$​​​效率高，因此需要先将问题做一个转化。

原始问题是$\mathrm{MSB}(x)$​​​一般的做法是Boolean加法然后做一次比较。现在希望不在Boolean 电路上做加法和比较。其方法是先观察到在奇数环里有$\mathrm{MSB}(a)\leftrightarrow \mathrm{LSB}(2a)$​​​，然后最低有效位根据环的定义也需要看原数有没有wrap around，即$\langle y \rangle_0^{L-1}+\langle y \rangle_1^{L-1} \ge L-1$​​​。这样问题又回去了。为避免做bit extraction，通过设定$r=y+x\ \mathrm{mod}\ L-1$​​​，把原问题转化为$y[0] = x[0]\oplus r[0] \oplus (x>r)$​​​，其核心在$x>r$​​​的判定，现在可以在算术电路上做比较了，相比于$y>L-1$​​，$r$​​可以公开而不会泄露$y$​​的信息（因为它$r$​是由$x$​给$y$​​加掩码​​计算得到的，如果不这样做的话，直接利用算术电路进行比较会泄露$y$​的信息）

- 对于奇数环$\mathbb{Z}_{L-1}$里，有$\mathrm{MSB}(a)\leftrightarrow \mathrm{LSB}(2a)$

  - 在模n的环里，$\mathrm{MSB}(a)=1 \leftrightarrow a > n/2 \leftrightarrow n$​​​ > $2a-n > 0$​​​。 因为$n$​​​是奇数，那么$2a-n$​​​是奇数。因此$\mathrm{LSB}(2a-n)=1$​​​。 又$2a-n = 2a\ \mathrm{mod}\ n$​​​， 因此 $\mathrm{LSB}(2a) = 1$​​​。同理，当$\mathrm{MSB}(a)=0$​​​， 有$\mathrm{LSB}(2a)=0$​​​。

- 假设现在的计算都已经在奇数环$\mathbb{Z}_{L-1}$​内，计算得到$y = 2a\ \mathrm{mod}\ L-1$​。那问题的关键在求$y[0]$。

  $y[0] = \langle y \rangle_0^{L-1}[0] \oplus \langle y \rangle_1^{L-1}[0] \oplus \underbrace{(\langle y \rangle_0^{L-1}+\langle y \rangle_1^{L-1} \ge L-1)\cdot 1}_{\text{根据环的定义，如果wrap around后减$L-1$}} $​​​

  - 现在需要计算$y \ge L-1$，为避免回到基于GSW或者GC的思路，因此继续设计
  - 对于$r=y+x\ \mathrm{mod}\ L-1$​​​​，如果$x>r$​​​​那么必然有$r = y+x-(L-1)$​​​​ ，说明wrap around了。因此 $y[0] = x[0]\oplus r[0] \oplus (x>r)$​​​​。 我们可以利用$P_2$​​​生成$x$​​​并以秘密分享的形式给$P_0,P_1$​​​，把$r$​​​（由$P_0,P_1$​​​交换信息算出来的）公开，那么计算$x>r$​​​将容易许多。——**这里也可以由$P_0,P_1$​​​生成各自的$\langle x\rangle_j^L$​​​​，再通过交换信息计算出$r$​​​​（好像可以提高效率，也没有泄露信息）**。


因此，首先给出计算$x>r$的协议

![](C:\Users\86188\Desktop\工作相关\job\Article\SecureNN\SecureNN-3.PNG)

这个方法适用于三方的情况，其主要的思路如下：

1. $P_0,P_1$​​分别持有$\{\langle x\rangle_0^p\}_{i\in [l]},\{\langle x\rangle_1^p\}_{i\in [l]}$​​。以及一个公共的$l$​​-bit的$r$​​和$1$​​-bit的$\beta$​​。$P_2$​​计算得到$\beta^{'}=\beta \oplus (x>r)$​​（$\beta=0$​​计算的是$\beta^{'} = (x>r)$​​，反之$\beta^{'} = (x\le r)$​​）。

2. 对$r$​​可以分为$r=2^l-1$​​和$r\ne 2^l -1 $​​两种情况讨论，因为当$r=2^l -1$​​时，总有$x\le r$​​成立，进而能简化计算。看代码11行表示的是$\beta=1$​​ AND $r=2^l -1$​​ 的情况，$P_2$​​中$\mathrm{Reconst(\langle d_i\rangle_0^p, \langle d_i\rangle_1^p)}$​​值为$111\dots0$​​，故$\beta^{'}=1$​​​，它表示$1 = 1 \oplus (x>r)\to (x>r)=0\to (x\le r)$​​​ 。

3. 对于$\beta = 0$的情况，$\beta^{'}=1$ 当且仅当$x>r$​成立。换句话说，最高非相等的位上有$x[i]\ne r[i]$且$x[i]=1$​，代码5-6行​表述的就是这个逻辑：

   - $w_i=x[i] \oplus r[i]=x[i] + r[i]- 2x[i]r[i]$​，$c_i = r[i]-x[x] + 1 + \sum_{k=i+1}^lw_k$
     - 当$w_{i+1} = 0,\dots,w_l = 0$​表示$x,r$的前$l-i$比特相等，如果$x[i] = r[i]$可以推出$w_i=0\to c_i = 1$，此时$c_i=1,\dots,c_l=1$
     - 当$w_{i+1} = 0,\dots,w_l = 0$​​即前$l-i$​​比特相等，如果$x[i] \ne r[i]$此时$w_i = 1$，对于$c_i$分两种情况，1）$x[i]=1$，可以推出$c_i=0$；2）$x[i]=0$，可以推出$c_i = 2$
     - 那么对于后面的$l-i$比特的比较而言，均有$c_i\ge 1$

   最后可以通过查询$c_i$是否等于0来判定是否有$x>r$。不过出于安全性考虑，$P_0,P_1$将$\langle c_i\rangle_j^p$传给$P_2$的时候需要通过非零的$s_i$和一个随机排列$\pi$处理后再发送。

4. 对于$\beta = 1$的情况逻辑同3

##### Share Convert

能够方便的在奇数环里进行比较运算了，现在就要设计一个转换协议$\mathbb{Z}_L\to \mathbb{Z}_{L-1}$，目标是$\mathrm{Reconst}^{L-1}(\langle y\rangle _0^{L-1} + \langle y\rangle _1^{L-1}) = \mathrm{Reconst}^{L}(\langle a\rangle _0^L + \langle a\rangle _1^L) = a$。想法是：

- 当$a<L\to\ a\ \mathrm{mod}\ L = a \ \mathrm{mod}\ L-1 $
- 当$a\ge L\to\ a\ \mathrm{mod}\ L = a \ \mathrm{mod}\ (L- 1 + 1)\to = a - (L-1) - 1 $
- 统一形式，即$\theta = (a > L): a \ \mathrm{mod}\ L-1\ = a - L - 1 - \underbrace{(a>L)}_{\theta}$

令$\theta = \mathrm{wrap}(\langle a\rangle _0^L, \langle a\rangle _1^L, L)$。现在需要利用前面的比较方法，来进行设计：

1. $r = \langle r\rangle _0^L + \langle r\rangle _1^L - \alpha L$

2. $\langle \tilde{a}\rangle_j^L = \langle a\rangle _j^L + \langle r\rangle _j^L - \beta_j L$

3. $x = \langle \tilde{a}\rangle _0^L + \langle \tilde{a}\rangle _1^L - \delta L$

4. $x = a+r-(1-\eta) L$

5. 令$\theta$满足$a=\langle a\rangle _0^L+\langle a\rangle _1^L-\theta L$

(1)-(2)-(3)+(4)+(5)可以解出来$\theta = \beta_0+\beta_1-\alpha+\delta+\eta -1$​

![](C:\Users\86188\Desktop\工作相关\job\Article\SecureNN\SecureNN-6.PNG)

##### Compute MSB

首先在odd环上，$\mathrm{MSB}(a) = \mathrm{LSB}(y),\ where\ y=2a$​，然后$P_2$​生成$x$​，$P_0,P_1$​计算​​得到$r = x + y\ \mathrm{mod}\ L-1$，再通过$x>r$的判定得到$\mathrm{wrap}(y, x, L-1)$​，这样就得到了最终结果 

![](C:\Users\86188\Desktop\工作相关\job\Article\SecureNN\SecureNN-7.PNG)

##### Overheads of supporting protocols

![](C:\Users\86188\Desktop\工作相关\job\Article\SecureNN\SecureNN-8.PNG)

#### Main Protocols

##### Linear and Convolutional Layer

线性运算实质是一个矩阵乘法计算

##### Derivative of ReLU



##### ReLU(x)