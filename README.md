## 论文翻译: Learning values across many orders of magnitude

Hado van Hasselt, Arthur Guez, Matteo Hessel, Volodymyr Mnih, David Silver. "Learning values across many orders of magnitude." In *Advances in Neural Information Processing Systems*, pp. 4287-4295. 2016. [[pdf](https://arxiv.org/pdf/1602.07714)]

### 符号系统

| 符号 	| 含义 | 符号 | 含义 |
| :---: | :--- | :---: | :--- |
| <img alt="$Y_t$" src="svgs/39e731509d35a821a251fe62866ee4a4.svg" align="middle" width="14.509275000000004pt" height="22.46574pt"/> | 目标 (target) | <img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/>  | 标准化层权重 | 
| <img alt="$\tilde{Y}_t$" src="svgs/6dd56c2803c2d23d14a979df7224a25d.svg" align="middle" width="14.509275000000004pt" height="30.267599999999987pt"/> | 标准化后的目标 | <img alt="${\boldsymbol b}$" src="svgs/d86c2b3d62d848f120253b393984a5bb.svg" align="middle" width="8.561685000000002pt" height="22.831379999999992pt"/> | 标准化层bias |
| <img alt="${\boldsymbol \Sigma}_t$" src="svgs/7f546d8bf5152d18c9ab0724628d1a5e.svg" align="middle" width="18.618765000000003pt" height="22.557149999999986pt"/>, <img alt="${\boldsymbol \mu}_t$" src="svgs/84d9637961b880dea8afc1447f1cc6ca.svg" align="middle" width="16.601970000000005pt" height="14.61206999999998pt"/> | 学习到的*scale*和*shift* | <img alt="$h_{\boldsymbol \theta}$" src="svgs/b021405126b65c0e04c15d944d1f134f.svg" align="middle" width="17.113965000000004pt" height="22.831379999999992pt"/> | 模型参数 | 
| <img alt="$g(\cdot)$" src="svgs/a0377869f11bf54f0a0e8ac338ecea79.svg" align="middle" width="25.782075000000003pt" height="24.65759999999998pt"/> | 标准化函数 | <img alt="$f(\cdot)$" src="svgs/8824595f458085fe3bf467c4228300fc.svg" align="middle" width="27.169065pt" height="24.65759999999998pt"/> | 原函数    |
 
### 第二章 Adaptive normalization with Pop-Art

> We propose to normalize the targets <img alt="$Y_t$" src="svgs/39e731509d35a821a251fe62866ee4a4.svg" align="middle" width="14.509275000000004pt" height="22.46574pt"/>, where the normalization is learned **separately** from the approximating function. We consider an affine transformation of the targets

<p align="center"><img alt="$$&#10;\tilde{Y}_ t = {\boldsymbol \Sigma}_ t^{-1} (Y_ t - {\boldsymbol \mu}_ t),&#10;$$" src="svgs/22fc937d416e5d5eba720ba16636ce1f.svg" align="middle" width="138.74784pt" height="19.24329pt"/></p>

本文的核心思路是将学习**标准化**与**函数拟合**两个过程分离开, 具体分离的方式是在原有的函数拟合网络下接一层**affine transformation**(线性变换)。原有的函数负责学习函数拟合, 而下接的一层线性变换层负责"跟踪"|学习标准化的参数。

> We can then define a loss on a normalized function <img alt="$g(X_t)$" src="svgs/b76f93b526e06c1b9208513d17d87503.svg" align="middle" width="40.62234pt" height="24.65759999999998pt"/> and the normalized target <img alt="$\tilde{Y}_ T$" src="svgs/73ac43eb7799407056d01ec72c6c80e7.svg" align="middle" width="19.077135000000002pt" height="30.267599999999987pt"/>. THe unnormalized approximation for any input <img alt="$x$" src="svgs/332cc365a4987aacce0ead01b8bdcc0b.svg" align="middle" width="9.395100000000005pt" height="14.155350000000013pt"/> is then given by <img alt="$f(x) = {\boldsymbol \Sigma} g(x) + {\boldsymbol \mu}$" src="svgs/1d9424936f715c8f631977f52b3f57a2.svg" align="middle" width="129.906645pt" height="24.65759999999998pt"/>, where <img alt="$g$" src="svgs/3cf4fbd05970446973fc3d9fa3fe3c41.svg" align="middle" width="8.430510000000004pt" height="14.155350000000013pt"/> is the *normalized function* and <img alt="$f$" src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg" align="middle" width="9.817500000000004pt" height="22.831379999999992pt"/> is the *unnormalized function*.

> Thereby, we decompose the problem of learning an appropriate normalization from learning the specific shape of the function. The two properties that we want to simultaneosly achieve are

> (**ART**) to update scale <img alt="${\boldsymbol \Sigma}$" src="svgs/121d8877038c55b6166a6e22acc96149.svg" align="middle" width="13.652925000000005pt" height="22.557149999999986pt"/> and shift <img alt="${\boldsymbol \mu}$" src="svgs/4e52c06a410e541f12a7a783da1aa80d.svg" align="middle" width="11.636295000000004pt" height="14.61206999999998pt"/> such that <img alt="${\boldsymbol \Sigma}^{-1} (Y - {\boldsymbol \mu})$" src="svgs/ed3b22e3ba194b711ab8bab28a300df6.svg" align="middle" width="89.01057pt" height="29.260439999999992pt"/> is approproately normalized, and  
> (**POP**) to preserve the outputs of the unnormalized function when we change the scale and shift.

其中**ART**负责学习标准化参数, 而**POP**负责保障任何的标准化参数下总能从标准化后的结果复原原始的输出。有了两点的结合就能实现针对任意输出情况下用合适的标准化参数进行normalization, 与此同时保障不破坏已经学习到函数拟合结果。

#### 2.1 POP: Preserving outputs precisely

> Unless care is taken, repeated updates to the normalization might make learning harder rather than easier because the normalized targets become non-stationary. More importantly, whenever we adapt the normalization based on a certain target, this would simultaneously change the output of the unnormalized function of all inputs.

本段解释了提出**POP**的motivation。可想而知, 如果无法保障原始（未标准化）的target不能被复原的话, 那么一旦在学习时采用了不同的标准化参数, 原有学习到的模型将受到影响（破坏）。举例而言, 假设首先学习到的模型是通过原始target范围在<img alt="$[0, 1]$" src="svgs/e88c070a4a52572ef1d5792a341c0900.svg" align="middle" width="32.87674500000001pt" height="24.65759999999998pt"/>内, 而后出现的样本target范围在<img alt="$[10, 100]$" src="svgs/a343cf36f4495c31070af031022eac92.svg" align="middle" width="57.534510000000004pt" height="24.65759999999998pt"/>, 将其标准化至<img alt="$[0, 1]$" src="svgs/e88c070a4a52572ef1d5792a341c0900.svg" align="middle" width="32.87674500000001pt" height="24.65759999999998pt"/>内后作为新的样本训练已有模型, 那么原来模型的所给出的<img alt="$[0, 1]$" src="svgs/e88c070a4a52572ef1d5792a341c0900.svg" align="middle" width="32.87674500000001pt" height="24.65759999999998pt"/>范围结果将受到"波及"。

> The **only way** to avoid changing all outputs of the unnormalized function whenever we update the scale and shift is by changing the normalized function <img alt="$g$" src="svgs/3cf4fbd05970446973fc3d9fa3fe3c41.svg" align="middle" width="8.430510000000004pt" height="14.155350000000013pt"/> itself simultaneously. The goal is the preserve the outputs from before the change of normalization, for all inputs. This prevents the normalization from affecting the approximation, which is appropriate because its objective is solely to make learning easier, and to leave solving the approximation itself to the optimization algorithm.

解决此问题的唯一办法就是在更新标准化参数的同时更新原有标准化前的输出。标准化的目的在于使得训练集在差不多的规模内从而使模型训练更为容易, 需要防止标准化影响模型训练本身。

> Without loss of generality the unnormalized function can be written as  
<p align="center"><img alt="$$&#10;f_ { {\boldsymbol \theta, \Sigma}, {\bf W}, {\boldsymbol b} } (x) \equiv {\boldsymbol \Sigma} g_ { {\boldsymbol \theta}, {\bf W}, {\boldsymbol b} } (x) + {\boldsymbol \mu} \equiv {\boldsymbol \Sigma} ( {\bf W} h_ { {\boldsymbol \theta} } (x) + {\boldsymbol b}) + {\boldsymbol \mu},&#10;$$" src="svgs/4c683ec967181adeab480d0cb8f72107.svg" align="middle" width="390.9444pt" height="17.031959999999998pt"/></p> 

其中<img alt="$g_ { {\boldsymbol \theta}, {\bf W}, {\boldsymbol b} } (x) = {\bf W} h_ { {\boldsymbol \theta} } (x) + {\boldsymbol b}$" src="svgs/951da717c5fac63397630ae2cb8d54f3.svg" align="middle" width="178.956855pt" height="24.65759999999998pt"/> 为标准化函数, 而<img alt="$h_ { {\boldsymbol \theta} } (x)$" src="svgs/6a1250dbd9366616ae954c3472309f2f.svg" align="middle" width="40.11612pt" height="24.65759999999998pt"/> 是函数拟合的网络(non-linear)。

> It is not uncommon for deep neural networks to end in a linear layer, and the <img alt="$h_ { {\boldsymbol \theta} }$" src="svgs/0c212e3dd00bcc0be7e55ad560026ac6.svg" align="middle" width="17.113965000000004pt" height="22.831379999999992pt"/> can be the output of the last (hidden) layer of non-linearities. Alternatively, we can always add a square linear layer to any non-linear function <img alt="$h_ { {\boldsymbol \theta} }$" src="svgs/0c212e3dd00bcc0be7e55ad560026ac6.svg" align="middle" width="17.113965000000004pt" height="22.831379999999992pt"/> to ensure this constraint, for instance initialized as <img alt="${\bf W}_ 0 = {\bf I}$" src="svgs/01268c569b6146a9845b6c3bef8f0841.svg" align="middle" width="56.266980000000004pt" height="22.557149999999986pt"/> and <img alt="${\boldsymbol b}_ 0 = {\boldsymbol 0}$" src="svgs/990ea5833bad1ce4e52b1647978ce072.svg" align="middle" width="47.305665000000005pt" height="22.831379999999992pt"/>.

本段解释了网络架构, 一般而言DNN的最后一层（输出层）均具备线性激活, 即满足以上的条件; 若不然, 则可以在原有网络的基础上加上一个线性层(本层参数规模为<img alt="$k\times k | k$" src="svgs/69b48152de429efd0c9094b4cab7b11d.svg" align="middle" width="51.88359pt" height="24.65759999999998pt"/>, <img alt="$k$" src="svgs/63bb9849783d01d91403bc9a5fea12a2.svg" align="middle" width="9.075495000000004pt" height="22.831379999999992pt"/>为输出的层的神经元数量), 不改变网络架构的同时满足了以上的条件。

> **Proposition 1.** *Consider a function <img alt="$f: \mathcal{R}^n \rightarrow \mathcal{R}^k$" src="svgs/d2662946a3ae77ef0029b52c0aa631a5.svg" align="middle" width="93.16362pt" height="27.91271999999999pt"/> as*
<p align="center"><img alt="$$&#10;f_ { {\boldsymbol \theta, \Sigma}, {\bf W}, {\boldsymbol b} } (x) \equiv {\boldsymbol \Sigma} ( {\bf W} h_ { {\boldsymbol \theta} } (x) + {\boldsymbol b}) + {\boldsymbol \mu},&#10;$$" src="svgs/a662b947928b8d220f6909ca7e151d3a.svg" align="middle" width="255.18405pt" height="17.031959999999998pt"/></p>  

> *where <img alt="$h_ {\boldsymbol \theta}: \mathcal{R}^n \rightarrow \mathcal{R}^m$" src="svgs/0f290327f70b5c997fc4b2496a20e507.svg" align="middle" width="105.680685pt" height="22.831379999999992pt"/> is any non-linear function of <img alt="$x\in \mathcal{R}^n$" src="svgs/004ab5a9f19ea49b06deb310e0407c1e.svg" align="middle" width="51.543855pt" height="22.46574pt"/>, <img alt="${\boldsymbol \Sigma}$" src="svgs/121d8877038c55b6166a6e22acc96149.svg" align="middle" width="13.652925000000005pt" height="22.557149999999986pt"/> is a <img alt="$k\times k$" src="svgs/649b7f492ffd7b6c6f42429f3fe29451.svg" align="middle" width="38.242050000000006pt" height="22.831379999999992pt"/> matrix, <img alt="${\boldsymbol \mu}$" src="svgs/4e52c06a410e541f12a7a783da1aa80d.svg" align="middle" width="11.636295000000004pt" height="14.61206999999998pt"/> and <img alt="${\boldsymbol b}$" src="svgs/d86c2b3d62d848f120253b393984a5bb.svg" align="middle" width="8.561685000000002pt" height="22.831379999999992pt"/> are <img alt="$k$" src="svgs/63bb9849783d01d91403bc9a5fea12a2.svg" align="middle" width="9.075495000000004pt" height="22.831379999999992pt"/>-element vectors, and <img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/> is a <img alt="$k\times m$" src="svgs/ef924289610252b6b10f485f9cc2a2d3.svg" align="middle" width="43.599765000000005pt" height="22.831379999999992pt"/> matrix. Consider any change of the scale and shift parameters from <img alt="${\boldsymbol \Sigma}$" src="svgs/121d8877038c55b6166a6e22acc96149.svg" align="middle" width="13.652925000000005pt" height="22.557149999999986pt"/> to <img alt="${\boldsymbol \Sigma}_ \text{new}$" src="svgs/626f14c3ae1babc3216165ecbfab765d.svg" align="middle" width="36.318645000000004pt" height="22.557149999999986pt"/> and from <img alt="${\boldsymbol \mu}$" src="svgs/4e52c06a410e541f12a7a783da1aa80d.svg" align="middle" width="11.636295000000004pt" height="14.61206999999998pt"/> to <img alt="${\boldsymbol \mu}_ \text{new}$" src="svgs/759b0ba96f2cc8322bbbf9dde93e366f.svg" align="middle" width="34.30185pt" height="14.61206999999998pt"/>, where <img alt="${\boldsymbol \Sigma}_ \text{new}$" src="svgs/626f14c3ae1babc3216165ecbfab765d.svg" align="middle" width="36.318645000000004pt" height="22.557149999999986pt"/> is non-singular. If we then additionally change the parameters <img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/> and <img alt="${\boldsymbol b}$" src="svgs/d86c2b3d62d848f120253b393984a5bb.svg" align="middle" width="8.561685000000002pt" height="22.831379999999992pt"/> to <img alt="${\bf W}_ \text{new}$" src="svgs/9f1325fa9976a0e80cb78e9afb4005c1.svg" align="middle" width="42.471495pt" height="22.557149999999986pt"/> and <img alt="${\boldsymbol b}_ \text{new}$" src="svgs/97dba530b5af389ba37d72b44c5b15be.svg" align="middle" width="31.227240000000002pt" height="22.831379999999992pt"/>, defined by*  

<p align="center"><img alt="$$ &#10;{\bf W}_ \text{new} = {\boldsymbol \Sigma}_ \text{new}^{-1} {\boldsymbol \Sigma} {\bf W} \quad \text{and} \quad {\boldsymbol b}_ \text{new} = {\boldsymbol \Sigma}_ \text{new}^{-1} \left( {\boldsymbol \Sigma b + \mu - \mu}_ \text{new} \right)&#10;$$" src="svgs/95f9bf2f5e8978a721f10fa8b2b81046.svg" align="middle" width="405.10634999999996pt" height="18.73971pt"/></p>  

> *then the outputs of the unnormalized function <img alt="$f$" src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg" align="middle" width="9.817500000000004pt" height="22.831379999999992pt"/> are preserved precisely in the sense that*  

<p align="center"><img alt="$$&#10;f_ { {\boldsymbol \theta, \Sigma}, {\bf W}, {\boldsymbol b} } (x) = f_ { {\boldsymbol \theta, \Sigma}_ \text{new}, {\bf W}_ \text{new}, {\boldsymbol b}_ \text{new} } (x), \quad \forall x.&#10;$$" src="svgs/5a763cc670ed583d67ce69be8bc88e51.svg" align="middle" width="292.12755pt" height="17.853824999999997pt"/></p>

以上的Proposition表明只要采用**适当的**标准化参数变换方案总能保证原函数拟合的结果保持不变。其中适当的变化方案即以上给出的<img alt="${\bf W}_ \text{new}$" src="svgs/9f1325fa9976a0e80cb78e9afb4005c1.svg" align="middle" width="42.471495pt" height="22.557149999999986pt"/>与<img alt="${\boldsymbol b}_ \text{new}$" src="svgs/97dba530b5af389ba37d72b44c5b15be.svg" align="middle" width="31.227240000000002pt" height="22.831379999999992pt"/>的更新方案。

> For the special case of scalar scale and shift, with <img alt="${\boldsymbol \Sigma} \equiv \sigma {\bf I}$" src="svgs/f3a8224dc5329fe285b71e5b41d4769b.svg" align="middle" width="52.72245pt" height="22.557149999999986pt"/> and <img alt="${\boldsymbol \mu} \equiv \mu {\boldsymbol 1}$" src="svgs/19071f7482fd4538727794dc3e7c0a19.svg" align="middle" width="52.91088pt" height="21.18732pt"/>, the updates to <img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/> and <img alt="${\boldsymbol b}$" src="svgs/d86c2b3d62d848f120253b393984a5bb.svg" align="middle" width="8.561685000000002pt" height="22.831379999999992pt"/> become <img alt="${\bf W}_ \text{new} = (\sigma/\sigma_\text{new}) {\bf W}$" src="svgs/ee1ce1a404b265b7ffdf0c587e2356ea.svg" align="middle" width="148.531185pt" height="24.65759999999998pt"/> and <img alt="${\boldsymbol b}_ \text{new} = (\sigma {\boldsymbol b} + {\boldsymbol \mu} - {\boldsymbol \mu}_ \text{new})/\sigma_ \text{new}$" src="svgs/194029dc8db2fb66141948b2e96974c2.svg" align="middle" width="212.16310499999997pt" height="24.65759999999998pt"/>. After updating the scale and shift we can update the output of the normalized function <img alt="$g_ { {\boldsymbol \theta}, {\bf W}, {\boldsymbol b} } (X_ t)$" src="svgs/d5d75d4c4c6d5d6e8a50089ee85f3f6b.svg" align="middle" width="78.475485pt" height="24.65759999999998pt"/> toward the normalized <img alt="$\tilde{Y}_ t$" src="svgs/a991a484bfbd0041e2b3cb9928640a20.svg" align="middle" width="14.509275000000004pt" height="30.267599999999987pt"/>, using any learning algorithm.

以上给出了特例, 即当scale和shift均为标量。按以上规则更新了标准化参数后, 根据**Proposition 1**可知原来的输出在新的标准化参数下并不会改变, 而新的训练数据通过新的标准化参数处理后即可用于对函数拟合模型(即<img alt="$h_ { {\boldsymbol \theta} }$" src="svgs/0c212e3dd00bcc0be7e55ad560026ac6.svg" align="middle" width="17.113965000000004pt" height="22.831379999999992pt"/>)的训练。

<p align="center">
<img src="https://user-images.githubusercontent.com/16682999/64672136-33788080-d49d-11e9-9771-bc07a48e99b6.png" alt="algorithm 1" width="800">
</p>

> Algorithm 1 is an example implementation of SGD with Pop-Art for a squared loss. It can be generalized easily to any other loss by changin the definition of <img alt="${\boldsymbol \delta}$" src="svgs/ef2db80fd6c60a9bf24092a1f40a1215.svg" align="middle" width="9.212280000000005pt" height="22.831379999999992pt"/>. Notice that <img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/> and <img alt="${\boldsymbol b}$" src="svgs/d86c2b3d62d848f120253b393984a5bb.svg" align="middle" width="8.561685000000002pt" height="22.831379999999992pt"/> are updated twice: first to adapt to the new scale and shift to preserve the outputs of the function, and then by SGD. The order of these updates is important because it allows us to use the new normalization immediately in the subsequent SGD update.

在此基础上, 算法1以MSE为loss函数, SGD为optimizer为例阐述了如何实现Pop-Art算法。其中主要分为三个阶段:  

- 更新标准化参数(Pop)  
- 更新函数拟合模型内层(<img alt="$h_ { {\boldsymbol \theta} }$" src="svgs/0c212e3dd00bcc0be7e55ad560026ac6.svg" align="middle" width="17.113965000000004pt" height="22.831379999999992pt"/>), (算法中红框标注部分)  
- 由SDG更新标准化层参数

**小结**: 本节主要阐述了Pop-Art算法中的Pop部分, 即如何更新标准化参数以保证后续的训练可以保障不影响已有输出的结果。另一方面, 通过算法1给出了结合Pop-Art的网络更新流程, 其中标准化层的参数将被更新两次, 第一次为保障Pop, 第二次为优化算法下的参数更新。

#### 2.2 ART: Adaptively rescaling targets

前一节中介绍了标准化参数更新过程中保障已有输出不变的基本思路, 而本节将具体给出如何进行标准化, 即Art: Adaptively rescaling targets的内涵。

> A natual choice is to normalize the targets to approximately have zero mean and unit variance. For clarity and conciseness, we consider scalar normalizations. If we have data <img alt="$\{ ( X_ i, Y_ i ) \}_ {i=1}^t$" src="svgs/42130d11734a175740b13e59c4e1f5bd.svg" align="middle" width="91.93239pt" height="26.086169999999992pt"/> up to some time <img alt="$t$" src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg" align="middle" width="5.936155500000004pt" height="20.222069999999988pt"/>, we then may desire

<p align="center"><img alt="$$&#10;\begin{aligned}&#10;&amp; \sum_ {i=1}^t (Y_ i - \mu_ t) / \sigma_ t = 0 \quad &amp; \text{and} \quad &amp; \frac{1}{t} \sum_ {i=1}^t ( Y_ i - \mu_ t )^2 / \sigma_t ^2 = 1,\\&#10;\text{such that} \quad \mu_t &amp;=\frac{1}{t}\sum_ {i=1}^t Y_i \quad &amp; \text{and} \quad \sigma_t^2 &amp;=\frac{1}{t} \sum_ {i=1}^t Y_i^2 - \mu_t^2.&#10;\end{aligned}&#10;$$" src="svgs/6cdad7742bddda1b2cd56580ce6ca043.svg" align="middle" width="505.99559999999997pt" height="103.90446pt"/></p>

> This can be generalized to incremental updates

<p align="center"><img alt="$$&#10;\mu_t = (1-\beta_t) \mu_{t-1} +\beta_t Y_t~\text{and}~\sigma_t^2 = \nu_t - \mu_t^2, \text{where}~ \nu_t = (1 - \beta_t) \nu_{t-1} + \beta_t Y_t^2.&#10;$$" src="svgs/11923b8eedbbab19d238723ed28a64b9.svg" align="middle" width="542.7113999999999pt" height="18.312359999999998pt"/></p>

> Here <img alt="$\nu_t$" src="svgs/b96ce619e9618788ad604f351c957414.svg" align="middle" width="13.086150000000004pt" height="14.155350000000013pt"/> estimates the second moment of the targets and <img alt="$\beta_t\in [0, 1]$" src="svgs/0a550c41abe1f9062c064428cfaf16a3.svg" align="middle" width="68.05359pt" height="24.65759999999998pt"/> is a step size. If <img alt="$\nu_t -\mu_t^2$" src="svgs/6d5125c6f26b74912240f25615270ae8.svg" align="middle" width="50.45667000000001pt" height="26.76201000000001pt"/> is positive initially then it will always remain so, although to avoid issues with numerical precision it can be useful to enforce a lower bound explicitly by requiring <img alt="$\nu_t -\mu_t^2 \geq \epsilon$" src="svgs/d86aecf0c5cf788e0d6a62ad52f807e7.svg" align="middle" width="79.86858pt" height="26.76201000000001pt"/> with <img alt="$\epsilon &gt;0$" src="svgs/b7e0fd4eae1532edec52a4554babe404.svg" align="middle" width="36.809355000000004pt" height="21.18732pt"/>. For full equivalence to above one we can use <img alt="$\beta_t = 1/t$" src="svgs/74f162db9f6cabad0259229548d5bfa1.svg" align="middle" width="59.37789pt" height="24.65759999999998pt"/>. If <img alt="$\beta_t = \beta$" src="svgs/722fa5db799a3e8c44a77b2bb5d5f958.svg" align="middle" width="47.16888pt" height="22.831379999999992pt"/> is constant we get exponential moving averages, placing more weight on recent data points which is appropriate in non-stationary settings.

自然地, 标准化的思路是将现有的数据统一到均值为0, 方差为1的规模, 考虑到数据本身是源源不断出现的, 更一般地可以用以上的增量式更新方式update均值(<img alt="$\mu_t$" src="svgs/87eefe082e181864d1321025c2705ecd.svg" align="middle" width="14.870790000000003pt" height="14.155350000000013pt"/>)和标准差(<img alt="$\sigma_t$" src="svgs/5ba3f1b75931f41283dac26b10c8c182.svg" align="middle" width="14.358960000000003pt" height="14.155350000000013pt"/>)。两种特殊情况: 1) <img alt="$\beta_t = 1/t$" src="svgs/74f162db9f6cabad0259229548d5bfa1.svg" align="middle" width="59.37789pt" height="24.65759999999998pt"/>, 则每个样本的权重一致; 2) <img alt="$\beta_t=\beta$" src="svgs/867dbaac43e71a1520ff55b9f46957d0.svg" align="middle" width="47.16888pt" height="22.831379999999992pt"/>为定值, 则意味着指数式的滑动平均, 最近的样本具有更高的权重, 适用于non-stationary的情况。另一方面, 从数值精度考虑, 有必要为更新的<img alt="$\sigma_t^2$" src="svgs/a7fb002779eacea5794d0e6ec7ae0753.svg" align="middle" width="16.535475000000005pt" height="26.76201000000001pt"/>(即<img alt="$\nu_t - \mu_t^2$" src="svgs/4a2439caf4a3e9cf078b45183f574586.svg" align="middle" width="50.45667000000001pt" height="26.76201000000001pt"/>)设置一个下限, <img alt="$\epsilon$" src="svgs/7ccca27b5ccc533a2dd72dc6fa28ed84.svg" align="middle" width="6.672451500000003pt" height="14.155350000000013pt"/>, 以防出现除0的情况。

> A constant <img alt="$\beta$" src="svgs/8217ed3c32a785f0b5aad4055f432ad8.svg" align="middle" width="10.165650000000005pt" height="22.831379999999992pt"/> has the additional benefit of never becoming negligibly small. Consider the first time a target is observed that is much larger than all previously observed targets. If <img alt="$\beta_t$" src="svgs/b2891c2d243ec9bb147096d2353d6bb4.svg" align="middle" width="14.263755000000003pt" height="22.831379999999992pt"/> is small, our statistics would adapt only slightly, and the resulting update may be large enough to harm the learning. If <img alt="$\beta_t$" src="svgs/b2891c2d243ec9bb147096d2353d6bb4.svg" align="middle" width="14.263755000000003pt" height="22.831379999999992pt"/> is not too small, the normalization can adapt to the large target before updating, potentially making learning more robust.

本段进一步解释了<img alt="$\beta$" src="svgs/8217ed3c32a785f0b5aad4055f432ad8.svg" align="middle" width="10.165650000000005pt" height="22.831379999999992pt"/>取定值的一个优点。假设<img alt="$\beta$" src="svgs/8217ed3c32a785f0b5aad4055f432ad8.svg" align="middle" width="10.165650000000005pt" height="22.831379999999992pt"/>逐步递减至较小值时, 若此时首次出现一个相对大的目标值, 由于<img alt="$\beta$" src="svgs/8217ed3c32a785f0b5aad4055f432ad8.svg" align="middle" width="10.165650000000005pt" height="22.831379999999992pt"/>太小, 导致新纳入的样本对已有的统计量(<img alt="$\mu$" src="svgs/07617f9d8fe48b4a7b3f523d6730eef0.svg" align="middle" width="9.904950000000003pt" height="14.155350000000013pt"/>和<img alt="$\sigma$" src="svgs/8cda31ed38c6d59d14ebefa440099572.svg" align="middle" width="9.982995000000003pt" height="14.155350000000013pt"/>)的影响很微小, 可以近似认为统计量不变, 那么根据算法1, 其中关键步骤1对统计量的更新就可以忽略, 此时第二步中由于新的样本target值过大导致对模型的训练产生大的影响。相反, 如果<img alt="$\beta$" src="svgs/8217ed3c32a785f0b5aad4055f432ad8.svg" align="middle" width="10.165650000000005pt" height="22.831379999999992pt"/>不太小, 这样的情况下, 在函数拟合网络(<img alt="$h_ { {\boldsymbol \theta} }$" src="svgs/0c212e3dd00bcc0be7e55ad560026ac6.svg" align="middle" width="17.113965000000004pt" height="22.831379999999992pt"/>)之前, 较大的target值将由统计量的更新而削弱进而增强模型整体的鲁棒性。

> **Proposition 2.** *When using updates above to adapt the normalization parameters <img alt="$\sigma$" src="svgs/8cda31ed38c6d59d14ebefa440099572.svg" align="middle" width="9.982995000000003pt" height="14.155350000000013pt"/> and <img alt="$\mu$" src="svgs/07617f9d8fe48b4a7b3f523d6730eef0.svg" align="middle" width="9.904950000000003pt" height="14.155350000000013pt"/>, the normalized targets are bounded for all <img alt="$t$" src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg" align="middle" width="5.936155500000004pt" height="20.222069999999988pt"/> by*
<p align="center"><img alt="$$&#10;-\sqrt{(1 - \beta_t) / \beta_t} \leq (Y_t - \mu_t) / \sigma_t \leq \sqrt{(1 - \beta_t) / \beta_t}.&#10;$$" src="svgs/f97207fa1ebd5278b1ee1e0b387d9d5f.svg" align="middle" width="340.33725pt" height="19.726245pt"/></p>

> For instance, if <img alt="$\beta_t = \beta = 10^{-4}$" src="svgs/d49e092afac3c6a5cdaafb490c6b184f.svg" align="middle" width="102.35148000000001pt" height="26.76201000000001pt"/> for all <img alt="$t$" src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg" align="middle" width="5.936155500000004pt" height="20.222069999999988pt"/>, then the normalized target is guaranteed to be in <img alt="$(-100, 100)$" src="svgs/6044cdaa4da611a15e9e83bb3c68b225.svg" align="middle" width="82.19211pt" height="24.65759999999998pt"/>. 

以上的Proposition表明了上述增量式统计参数更新下获得的target范围与<img alt="$\beta_t$" src="svgs/b2891c2d243ec9bb147096d2353d6bb4.svg" align="middle" width="14.263755000000003pt" height="22.831379999999992pt"/>的关系, 对于参数选择具有一定指导效果。

> It is an open question whether it is uniformly best to normalize by mean and variance. In the appendix we discuss other normalization updates, based on percentiles and mini-batches, and derive correspondences between all of these.

以上给出的是均值|标准差的标准化方式, 该种方式是否具有普适性仍然是一个开放式的问题。作者在附录中讨论了其他的标准化方式, 尤其是**mini-batches**值得关注。

**小结**: 本节介绍了Pop-Art中的"Art"部分, 即自适应target缩放。提出了一个增量式的标准化方式, 其中通过参数<img alt="$\beta_t$" src="svgs/b2891c2d243ec9bb147096d2353d6bb4.svg" align="middle" width="14.263755000000003pt" height="22.831379999999992pt"/>的不同取值方式可以应对不同的场景。针对target non-stationary的情形, 建议使用<img alt="$\beta_t$" src="svgs/b2891c2d243ec9bb147096d2353d6bb4.svg" align="middle" width="14.263755000000003pt" height="22.831379999999992pt"/>为定值, 以增强模型鲁棒性, 便于应对target出现突然的变化。（否则, 若<img alt="$\beta_t$" src="svgs/b2891c2d243ec9bb147096d2353d6bb4.svg" align="middle" width="14.263755000000003pt" height="22.831379999999992pt"/>随<img alt="$t$" src="svgs/4f4f4e395762a3af4575de74c019ebb5.svg" align="middle" width="5.936155500000004pt" height="20.222069999999988pt"/>减小, "异常"的target对统计参数的影响将很小, 但对模型更新的"伤害"却很大。）

#### 2.3 An equivalence for stochastic gradient descent

> We now step back and analyze the effect of the magnitude of the errors on the gradients when using regular SDG. This analysis suggests a different normalization algorithm, which has an interesting correspondence to Pop-Art SGD.

在本节中, 作者将讨论跨度广泛的target对SGD算法的影响, 并提出相应的标准化算法。该算法与前述的Pop-Art SGD (算法1)有着有趣的关联。

> We consider SGD updates for an unnormalized multi-layer function of form <img alt="$f_ { {\boldsymbol \theta}, {\bf W}, {\boldsymbol b} } (X) = {\bf W} h_ { \boldsymbol \theta}(X) + {\boldsymbol b}$" src="svgs/b89ea407d35f61bf1f59e1d031b7c5a7.svg" align="middle" width="190.19170499999998pt" height="24.65759999999998pt"/>. The update for the weight matrix <img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/> is 

<p align="center"><img alt="$$&#10;{\bf W}_ t = {\bf W}_ {t-1} + \alpha_t {\boldsymbol \delta}_ t h_ { { \boldsymbol \theta}_ t} (X_t)^\intercal,&#10;$$" src="svgs/2a826400e4c702cadaec453e0adbcf57.svg" align="middle" width="209.47574999999998pt" height="16.438356pt"/></p>

> where <img alt="${\boldsymbol \delta}_ t = f_ { {\boldsymbol \theta}, {\bf W}, {\boldsymbol b} } (X) - Y_t$" src="svgs/80c0691f522310bd8305c3ede2199813.svg" align="middle" width="145.703085pt" height="24.65759999999998pt"/> is gradient of the squared loss, which we here call **unnormalized error**. The magnitude of this update depends linearly on the magnitude of the error, which is appropriate when the inputs are normalized, because then the ideal scale of the weights depends linearly on the magnitude of the targets.

本段着手分析最后一层的权重(<img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/>)的更新与误差项(<img alt="$f_ { {\boldsymbol \theta}, {\bf W}, {\boldsymbol b} } (X) - Y_t$" src="svgs/6164c852e14cc14ad0919c489a7b3da4.svg" align="middle" width="108.78549pt" height="24.65759999999998pt"/>)之间的关系, 指出权重的更新规模与误差项的规模呈线性关系。进一步误差项本身的规模又与target的规模呈线性关系。正常情况下(即target的规模在相对小的范围内时), 以上的更新是没问题的。

> Now consider the SGD update to the parameters of <img alt="$h_ { {\boldsymbol \theta} }$" src="svgs/0c212e3dd00bcc0be7e55ad560026ac6.svg" align="middle" width="17.113965000000004pt" height="22.831379999999992pt"/>, <img alt="${\boldsymbol \theta}_ t = {\boldsymbol \theta}_ {t-1}  - \alpha {\boldsymbol J}_ t {\bf W}_ {t-1}^\intercal {\boldsymbol \delta}_ t$" src="svgs/cc69cb59ed596e20470a9b517507743a.svg" align="middle" width="174.925905pt" height="25.6047pt"/> where <img alt="${\boldsymbol J}_ t = ( \nabla h_ { {\boldsymbol \theta}, 1 }  (X), \ldots, \nabla h_ { {\boldsymbol \theta}, m } (X) )^\intercal$" src="svgs/7a491d6d3be2fa78eda0a36f9eb562ee.svg" align="middle" width="241.17835499999998pt" height="24.65759999999998pt"/> is the Jacobian for <img alt="$h_ { {\boldsymbol \theta} }$" src="svgs/0c212e3dd00bcc0be7e55ad560026ac6.svg" align="middle" width="17.113965000000004pt" height="22.831379999999992pt"/>. The magnitudes of both the weights <img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/> and the erros <img alt="${\boldsymbol \delta}$" src="svgs/ef2db80fd6c60a9bf24092a1f40a1215.svg" align="middle" width="9.212280000000005pt" height="22.831379999999992pt"/> depend linearly on the magnitude of the targets. This means that the magnitude of the update for <img alt="${\boldsymbol \theta}$" src="svgs/09ea36f6e30cbbd2fe751158784ed2e5.svg" align="middle" width="9.760245000000003pt" height="22.831379999999992pt"/> depends **quaratically** on the magnitude of the targets. *There is no compelling reason for these updates to depend at all on these magnitudes because the weights in the top layer already ensure appropriate scaling.* In other words, for each doubling of the magnitudes of the targets, the updates to the lower layers quadruple for no clear reason.

进一步, lower layer的网络参数(<img alt="${\boldsymbol \theta}$" src="svgs/09ea36f6e30cbbd2fe751158784ed2e5.svg" align="middle" width="9.760245000000003pt" height="22.831379999999992pt"/>)更新幅度根据公式既与<img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/>规模呈线性关系, 又与误差项规模呈线性关系, 两者均与target的规模呈线性关系并且是相乘的关系, 综合导致lower layer网络参数更新幅度与target的规模呈平方关系。而事实上并没有显著的理由保证如此的关系。（相反这可能造成对参数的"破坏"）

**注**: 一般多层网络中, lower layer | upper layer的概念是根据搭建的顺序而言, 接近Input的为lower layer, 而接近Output的为upper layer。（此前对这两者理解正好相反🤣）

> This analysis suggests an algorithmic solution, which seems to be novel in and of itself, in which we track the magnitude of the targets in a separate parameter <img alt="$\sigma_t$" src="svgs/5ba3f1b75931f41283dac26b10c8c182.svg" align="middle" width="14.358960000000003pt" height="14.155350000000013pt"/>, and then multiply the updates for all lower layers with a factor <img alt="$\sigma_t^{-2}$" src="svgs/0d565a477fd661861c27bbe250b0ae9d.svg" align="middle" width="26.809530000000006pt" height="28.89513000000001pt"/>. A more general version of this for matrix scallings is given in Algorithm 2.

根据以上的分析, 作者提出了相应的解决方案, 即跟踪target的规模<img alt="$\sigma_t$" src="svgs/5ba3f1b75931f41283dac26b10c8c182.svg" align="middle" width="14.358960000000003pt" height="14.155350000000013pt"/>, 并对lower layer的更新相应乘以<img alt="$\sigma_t^{-2}$" src="svgs/0d565a477fd661861c27bbe250b0ae9d.svg" align="middle" width="26.809530000000006pt" height="28.89513000000001pt"/>从而消除此处引入的target规模的影响。具体的算法流程在Algorithm 2中给出（其中<img alt="${\bf W} \leftarrow {\bf W} - \alpha {\boldsymbol \delta} {\boldsymbol g}^\intercal$" src="svgs/df2b13e034551dff1ded630ad81110cd.svg" align="middle" width="122.07145499999999pt" height="22.831379999999992pt"/>中<img alt="${\boldsymbol g}$" src="svgs/3df5c9a7a58493944eea553b2055d657.svg" align="middle" width="9.566205000000004pt" height="14.61206999999998pt"/>应该是<img alt="${\boldsymbol h}$" src="svgs/a0c5d722828357bd74fbe43f7f445706.svg" align="middle" width="10.974150000000005pt" height="22.831379999999992pt"/>）。

<p align="center">
<img src="https://user-images.githubusercontent.com/16682999/64753079-747f9c00-d554-11e9-9bb3-e93407d09307.png" alt="algorithm 2" width="800">
</p>

算法2中的核心步骤是<img alt="${\boldsymbol \theta} \leftarrow {\boldsymbol \theta} - \alpha {\boldsymbol J} ( {\color{red} {\boldsymbol \Sigma} ^{-1} } {\bf W})^\intercal {\color{red} {\boldsymbol \Sigma}^{-1} } {\boldsymbol \delta}$" src="svgs/db39950acd76933ad0c313abf76e90db.svg" align="middle" width="200.461305pt" height="29.260439999999992pt"/>, 其中对<img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/>和<img alt="${\boldsymbol \delta}$" src="svgs/ef2db80fd6c60a9bf24092a1f40a1215.svg" align="middle" width="9.212280000000005pt" height="22.831379999999992pt"/>分别做了"半标准化"处理, 即将其magnitude置为一。

**小结**: 至此, 作者提出了第二个算法。该算法的思路不同于算法1对target进行标准化, 而是看准target规模对模型参数update的影响会分别通过误差项(<img alt="${\boldsymbol \delta}_ t$" src="svgs/9bd6306c104510dcd2962710573f3aad.svg" align="middle" width="14.178120000000003pt" height="22.831379999999992pt"/>)和top layer权重(<img alt="${\bf W}$" src="svgs/ce01fa56c40e0a3b42fb4a4c6c6d67fe.svg" align="middle" width="19.805940000000003pt" height="22.557149999999986pt"/>)引入而造成quadratic的影响, 考虑在lower layer模型参数update中消除该影响。

> We prove an interesting, and perhaps surprising, connection to the Pop-Art algorithm.

> **Proposition 3.** *Consider two functions defined by*

<p align="center"><img alt="$$&#10;f_ { {\boldsymbol \theta}, {\boldsymbol \Sigma}, {\bf W}, {\boldsymbol b} } (x) = {\boldsymbol \Sigma}({\bf W} h_ { {\boldsymbol \theta} } (x) + {\boldsymbol b} ) + {\boldsymbol \mu} \quad \text{and} \quad f_ { {\boldsymbol \theta}, {\bf W}, {\boldsymbol b} } (x) = {\bf W} h_ { {\boldsymbol \theta} } (x) + {\boldsymbol b},&#10;$$" src="svgs/659962984fe3cb7dcea2c918e94bf12b.svg" align="middle" width="495.04124999999993pt" height="17.031959999999998pt"/></p>

> *where <img alt="$h_ { {\boldsymbol \theta} }$" src="svgs/0c212e3dd00bcc0be7e55ad560026ac6.svg" align="middle" width="17.113965000000004pt" height="22.831379999999992pt"/> is the same differentiable function in both cases, and the functions are initialized identically, using <img alt="${\boldsymbol \Sigma}_ 0 = {\bf I}$" src="svgs/ccbc63de47f79763c55b599a0ba72412.svg" align="middle" width="50.113965pt" height="22.557149999999986pt"/> and <img alt="${\boldsymbol \mu}={\bf 0}$" src="svgs/a9fee4b0dbf83df610b0b48669e20e3d.svg" align="middle" width="43.00593pt" height="21.18732pt"/>, and the same initial <img alt="${\boldsymbol \theta}_ 0$" src="svgs/d89fa1c025a215ce0a649f721bd0b721.svg" align="middle" width="16.312890000000003pt" height="22.831379999999992pt"/>, <img alt="${\bf W}_ 0$" src="svgs/9f410b003037820f319071b8d2fc3f82.svg" align="middle" width="26.358420000000002pt" height="22.557149999999986pt"/> and <img alt="${\boldsymbol b}_ 0$" src="svgs/ad635100e33dc6d4795fe40a81b298fb.svg" align="middle" width="15.114165000000005pt" height="22.831379999999992pt"/>. Consider updating the first function using Algorithm 1 (Pop-Art SGD) and the second using Algorithm 2 (Normalized SDG). Then, for any sequence of non-singular scales <img alt="$\{ {\boldsymbol \Sigma}_ t \}_ {t=1}^\infty$" src="svgs/93ede31b4d2181aa0a4961a274c7bfad.svg" align="middle" width="57.488805pt" height="24.65759999999998pt"/> and shift <img alt="$\{ {\boldsymbol \mu}_ t \}_ {t=1}^\infty$" src="svgs/e82018da0d0a8a5ed808e3577c477e70.svg" align="middle" width="55.472010000000004pt" height="24.65759999999998pt"/>, the algorithms are equivalent in the sense that 1) the sequences <img alt="$\{ {\boldsymbol \theta}_ t \}_ {t=0}^\infty$" src="svgs/86209e8cb79c2f82abc3b5dad86cb30c.svg" align="middle" width="53.596125pt" height="24.65759999999998pt"/> are identical, 2) the outputs of the functions are identical, for any input.*

以上的Proposition阐述了算法2和算法1的等效性。当初始的上下层权重一致时, 并且算法1中初始的统计算法按照"单位化"设置, 则两个算法下在任意时刻的结果均是一致的。

> **The proposition shows a duality between normalizing the targets, as in Algorithm 1, and changing the updates, as in Algorithm 2.** This allows us to gain more intuition about the algorithm. In particular, in Algorithm 2 the updates in top layer are not normalized, thereby allowing the last linear layer to adapt to the scale of the targets. That said, these methods are complementary, and it is straightforward to combing Pop-Art with other optimization algorithms than SGD.

再次强调以上Proposition的重要性, 保障了两个算法间的对偶性。另一方面, 算法2中的top layer并未进行标准化处理, 从而保留了其适应target的调整空间。Proposition表明Pop-Art算法本身可以作为对其他优化算法的一个补充。

**小结**: 本节提出了重要的Proposition, 阐述了算法2和算法1的等效性, 因此Pop-Art算法可以作为对其他优化算法的一个补充。

### 总结与思考

本文考虑强化学习中target跨度大而导致模型学习效果差的问题进行了研究, 提出了Pop-Art算法解决该问题。其算法的核心包括两个部分: 1) Pop (Presering outputs precisely): 即无论标准化参数如何变化, 已有的输出不会变化; 2) Art (Adaptive rescaling target): 即自适应目标值放缩, 保障新的目标值能合理的放缩标准化。此外, 本文还分析了target跨度大导致模型学习效果差的原因: target的跨度影响将"平方式"地影响lower layer的参数更新。在此基础上, 作者提出了针对lower layer更新的修正算法, 并证明了该算法与前述Pop-Art算法的等效性。

**疑问**: 强化学习算法中多为mini-batch的训练模式, 那么在随机选取的batch中可能出现如下的情况: <img alt="$[ a_1, \ldots, a_n, b_1, \ldots, b_m]$" src="svgs/baba2abc52b0cddb5feffddf6b5af85a.svg" align="middle" width="157.168605pt" height="24.65759999999998pt"/>。其中<img alt="$a$" src="svgs/44bc9d542a92714cac84e01cbbb7fd61.svg" align="middle" width="8.689230000000004pt" height="14.155350000000013pt"/>, <img alt="$b$" src="svgs/4bdc8d9bcfb35e1c9bfb51fc69687dfc.svg" align="middle" width="7.054855500000005pt" height="22.831379999999992pt"/>序列分别在不同的范围, 同组中彼此差异仅体现在小数点后3位, 而两组间的差异为个位数。在这种情况下, 标准化处理是否能够有效区分组内的差异呢？此batch的处理是否应该将样本逐一输入进行处理？

**此疑问在文章的附加部分中有所涉及, 即需要替换"Art"部分。**