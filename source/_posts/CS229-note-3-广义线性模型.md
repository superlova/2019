---
title: CS229-note-3-广义线性模型
date: 2019-04-26 21:08:10
tags: ["机器学习", "笔记"]
categories:
  - CS229
mathjax: true
---
在前面的学习中，我们主要讨论回归和分类的问题，在回归问题中我们默认的分布模型为正态分布$y | x ; \theta \sim \mathcal{N}\left(\mu, \sigma^{2}\right)$，在分类问题中的模型为伯努利二项分布$y | x ; \theta \sim$ Bernoulli $(\phi)$。这里的$\mu$与$\phi$是$x$与$\theta$的函数

我们接下来要介绍广义线性模型（generalized linear models，简称 GLM），之前的两种模型都是广义线性模型的特例，除此之外广义线性模型还可以导出其他不同的模型，适用于其他类型的分类或回归问题。

## 1. 指数族（The Exponential Family）
在介绍广义线性模型之前，要先介绍指数族分布。如果一个分布的概率密度函数可以写成：
$$
p(y ; \eta)=b(y) \exp \left(\eta^{T} T(y)-a(\eta)\right)
$$
则称该分布属于指数族。这里的$\eta$称为该分部的natural parameter。T(y)称之为充分统计量(sufficient statistic)，在我们讨论的例子中，T(y)=y。$a(\eta)$是一个log partition函数。$e^{-a(\eta)}$是一个用来让整个函数正规化（normalization）的常数，也就是让整个$p(y;\eta)$函数对y积分或加和之后等于一。

选定T、a、b之后，就会得到一个参数为$\eta$的分布族，当我们改变$\eta$的值，就可以得到该分布族的一个分布。

接下来将证明Bernoulli 与 Gaussian（正态）分布其实也都是指数族分布的一种，也就是说上面的T、a、b经过适当的选择之后，就可以得到这两个分布的概率密度函数。

Bernoulli(φ)的概率密度函数为
$$
p(y ; \phi)=\left\{\begin{array}{ll}{\phi,} & {\text { if } \mathrm{y}=1} \\ {1-\phi,} & {\text { if } \mathrm{y}=0}\end{array}\right.
$$
当我们改变φ的时候，就可以得到不同的Bernoulli分布。上面这个式子可以改写成：
$$
\begin{aligned} p(y, \phi) &=\phi^{y}(1-\phi)^{1-y} \\ &=\exp (y \log \phi+(1-y) \log (1-\phi)) \\ &=\exp \left(\left(\log \left(\frac{\phi}{1-\phi}\right)\right) y+\log (1-\phi)\right) \end{aligned}
$$
这样就可以看出来，natural parameter为$\eta=\log{\frac{\phi}{1-\phi}}$，此时
$$
\begin{aligned} T(y) &=y \\ a(\eta) &=-\log (1-\phi) \\ &=\log \left(1+e^{\eta}\right) \\ b(y) &=1 \end{aligned}
$$

同理，如果设置正态分布的方差为1，则正态分布的概率密度函数也可以写成：
$$
\begin{aligned} p(y ; \mu) &=\frac{1}{\sqrt{2 \pi}} \exp \left(-\frac{1}{2}(y-\mu)^{2}\right) \\ &=\frac{1}{\sqrt{2 \pi}} \exp \left(-\frac{1}{2} y^{2}\right) \cdot \exp \left(\mu y-\frac{1}{2} \mu^{2}\right) \end{aligned}
$$
其中
$$
\begin{aligned} \eta &=\mu \\ T(y) &=y \\ a(\eta) &=\mu^{2} / 2 \\ &=\eta^{2} / 2 \\ b(y) &=(1 / \sqrt{2 \pi}) \exp \left(-y^{2} / 2\right) \end{aligned}
$$

事实上，很多分布都属于指数组分布，包括多项式分布、泊松分布、指数分布、gamma分布、beta分布、狄利克雷分布等等

## 2. 构造广义线性模型（GLMs）
以下部分摘自@飞龙翻译的CS229笔记。
设想你要构建一个模型，来估计在给定的某个小时内来到你商店的顾客人数（或者是你的网站的页面访问次数），基于某些确定的特征 $x$ ，例如商店的促销、最近的广告、天气、今天周几啊等等。我们已经知道泊松分布（Poisson distribution）通常能适合用来对访客数目进行建模。知道了这个之后，怎么来建立一个模型来解决咱们这个具体问题呢？非常幸运的是，泊松分布是属于指数分布族的一个分布，所以我们可以对该问题使用广义线性模型（Generalized Linear Model，缩写为 GLM）。在本节，我们讲一种对刚刚这类问题构建广义线性模型的方法。

进一步泛化，设想一个分类或者回归问题，要预测一些随机变量 $y$ 的值，作为 $x$ 的一个函数。要导出适用于这个问题的广义线性模型，就要对我们的模型、给定 $x$ 下 $y$ 的条件分布来做出以下三个假设：

$y | x; \theta ∼ Exponential Family(\eta)$，即给定 $x$ 和 $\theta, y$ 的分布属于指数分布族，是一个参数为 $\eta$ 的指数分布。——**假设1**

给定 $x$，目的是要预测对应这个给定 $x$ 的 $T(y)$ 的期望值。咱们的例子中绝大部分情况都是 $T(y) = y$，这也就意味着我们的学习假设 $h$ 输出的预测值 $h(x)$ 要满足 $h(x) = E[y|x]$。 （注意，这个假设通过对 $h_\theta(x)$ 的选择而满足，在逻辑回归和线性回归中都是如此。例如在逻辑回归中， $h_\theta (x) = [p (y = 1|x; \theta)] =[ 0 \cdot p (y = 0|x; \theta)+1\cdot p(y = 1|x;\theta)] = E[y|x;\theta]$。译者注：这里的$E[y|x$]应该就是对给定$x$时的$y$值的期望的意思。）——**假设2**

自然参数 $\eta$ 和输入值 $x$ 是线性相关的，$\eta = \theta^T x$，或者如果 $\eta$ 是有值的向量，则有$\eta_i = \theta_i^T x$。——**假设3**

上面的几个假设中，第三个可能看上去证明得最差，所以也更适合把这第三个假设看作是一个我们在设计广义线性模型时候的一种 “设计选择 design choice”，而不是一个假设。那么这三个假设/设计，就可以用来推导出一个非常合适的学习算法类别，也就是广义线性模型 GLMs，这个模型有很多特别友好又理想的性质，比如很容易学习。此外，这类模型对一些关于 $y$ 的分布的不同类型建模来说通常效率都很高；例如，我们下面就将要简单介绍一些逻辑回归以及普通最小二乘法这两者如何作为广义线性模型来推出。

### 2.1 普通最小二乘法（Ordinary Least Squares）
我们这一节要讲的是普通最小二乘法实际上是广义线性模型中的一种特例，设想如下的背景设置：目标变量 $y$（在广义线性模型的术语也叫做响应变量response variable）是连续的，然后我们将给定 $x$ 的 $y$ 的分布以高斯分布 $N(\mu, \sigma^2)$ 来建模，其中 $\mu$ 可以是依赖 $x$ 的一个函数。这样，我们就让上面的$ExponentialFamily(\eta)$分布成为了一个高斯分布。在前面内容中我们提到过，在把高斯分布写成指数分布族的分布的时候，有$\mu = \eta$。所以就能得到下面的等式：

$$ \begin{aligned} h_\theta(x)& = E[y|x;\theta] \ & = \mu \ & = \eta \ & = \theta^Tx\ \end{aligned} $$

第一行的等式是基于**假设2**；第二个等式是基于定理当 $y|x; \theta ∼ N (\mu, \sigma ^2)$，则 $y$ 的期望就是 $\mu$ ；第三个等式是基于**假设1**，以及之前我们此前将高斯分布写成指数族分布的时候推导出来的性质 $\mu = \eta$；最后一个等式就是基于**假设3**。

### 2.2 逻辑回归（Logistic Regression）
接下来咱们再来看看逻辑回归。这里咱们还是看看二值化分类问题，也就是 $y \in {0, 1}$。给定了$y$ 是一个二选一的值，那么很自然就选择伯努利分布（Bernoulli distribution）来对给定 $x$ 的 $y$ 的分布进行建模了。在我们把伯努利分布写成一种指数族分布的时候，有 $\phi = 1/ (1 + e^{−\eta})$。另外还要注意的是，如果有 $y|x; \theta ∼ Bernoulli(\phi)$，那么 $E [y|x; \theta] = \phi$。所以就跟刚刚推导普通最小二乘法的过程类似，有以下等式：

$$ \begin{aligned} h_\theta(x)& = E[y|x;\theta] \ & = \phi \ & = 1/(1+ e^{-\eta}) \ & = 1/(1+ e^{-\theta^Tx})\ \end{aligned} $$

所以，上面的等式就给了给了假设函数的形式：$h_\theta(x) = 1/ (1 + e^{−\theta^T x})$。如果你之前好奇咱们是怎么想出来逻辑回归的函数为$1/ (1 + e^{−z} )$，这个就是一种解答：一旦我们假设以 $x$ 为条件的 $y$ 的分布是伯努利分布，那么根据广义线性模型和指数分布族的定义，就会得出这个式子。

再解释一点术语，这里给出分布均值的函数 $g$ 是一个关于自然参数的函数，$g(\eta) = E[T(y); \eta]$，这个函数也叫做规范响应函数（canonical response function）， 它的反函数 $g^{−1}$ 叫做规范链接函数（canonical link function）。 因此，对于高斯分布来说，它的规范响应函数正好就是识别函数（identify function）；而对于伯努利分布来说，它的规范响应函数则是逻辑函数（logistic function）。$^*$注

* 很多教科书用 $g$ 表示链接函数，而用反函数$g^{−1}$ 来表示响应函数；但是咱们这里用的是反过来的，这是继承了早期的机器学习中的用法，我们这样使用和后续的其他课程能够更好地衔接起来。

### 2.3 Softmax 回归
咱们再来看一个广义线性模型的例子吧。设想有这样的一个分类问题，其中响应变量 $y$ 的取值可以是 $k$ 个值当中的任意一个，也就是 $y \in {1, 2, ..., k}$。例如，我们这次要进行的分类就比把邮件分成垃圾邮件和正常邮件两类这种二值化分类要更加复杂一些，比如可能是要分成三类，例如垃圾邮件、个人邮件、工作相关邮件。这样响应变量依然还是离散的，但取值就不只有两个了。因此咱们就用多项式分布（multinomial distribution）来进行建模。

下面咱们就通过这种多项式分布来推出一个广义线性模型。要实现这一目的，首先还是要把多项式分布也用指数族分布来进行描述。

要对一个可能有 $k$ 个不同输出值的多项式进行参数化，就可以用 $k$ 个参数 $\phi_1,...,\phi_ k$ 来对应各自输出值的概率。不过这么多参数可能太多了，形式上也太麻烦，他们也未必都是互相独立的（比如对于任意一个$\phi_ i$中的值来说，只要知道其他的 $k-1$ 个值，就能知道这最后一个了，因为总和等于$1$，也就是$\sum^k_{i=1} \phi_i = 1$）。所以咱们就去掉一个参数，只用 $k-1$ 个：$\phi_1,...,\phi_ {k-1}$ 来对多项式进行参数化，其中$\phi_i = p (y = i; \phi)，p (y = k; \phi) = 1 −\sum ^{k−1}{i=1}\phi i$。为了表述起来方便，我们还要设 $\phi_k = 1 − \sum_{i=1}^{k−1} \phi_i$，但一定要注意，这个并不是一个参数，而是完全由其他的 $k-1$ 个参数来确定的。

要把一个多项式表达成为指数组分布，还要按照下面的方式定义一个 $T (y) \in R^{k−1}$:

$$
T(1)=\left[ \begin{array}{c}{1} \\ {0} \\ {0} \\ {\vdots} \\ {0}\end{array}\right], T(2)=\left[ \begin{array}{c}{0} \\ {1} \\ {0} \\ {\vdots} \\ {0}\end{array}\right], T(3)=\left[ \begin{array}{c}{0} \\ {0} \\ {1} \\ {\vdots} \\ {0}\end{array}\right], \cdots, T(k-1)=\left[ \begin{array}{c}{0} \\ {0} \\ {0} \\ {\vdots} \\ {1}\end{array}\right], T(k)=\left[ \begin{array}{c}{0} \\ {0} \\ {0} \\ {\vdots} \\ {0}\end{array}\right]
$$

这次和之前的样例都不一样了，就是不再有 $T(y) = y$；然后，$T(y)$ 现在是一个 $k – 1$ 维的向量，而不是一个实数了。向量 $T(y)$ 中的第 $i$ 个元素写成$(T(y))_i$ 。

现在介绍一种非常有用的记号。指示函数（indicator function）$1{\cdot }$，如果参数为真，则等于$1$；反之则等于$0$（$1{True} = 1, 1{False} = 0$）。例如$1{2 = 3} = 0$, 而$1{3 = 5 − 2} = 1$。所以我们可以把$T(y)$ 和 $y$ 的关系写成 $(T(y))_i = 1{y = i}$。（往下继续阅读之前，一定要确保你理解了这里的表达式为真！）在此基础上，就有了$E[(T(y))_i] = P (y = i) = \phi_i$。

现在一切就绪，可以把多项式写成指数族分布了。写出来如下所示：

$$
\begin{aligned} p(y ; \phi) &=\phi_{1}^{1\{y=1\}} \phi_{2}^{1\{y=2\}} \ldots \phi_{k}^{1\{y=k\}} \\ &=\phi_{1}^{1\{y=1\}} \phi_{2}^{1\{y=2\}} \cdots \phi_{k}^{1-\sum_{i=1}^{k-1} 1\{y=i\}} \\ &=\phi_{1}^{(T(y))_{1}} \phi_{2}^{(T(y))_{2}} \ldots \phi_{k}^{1-\sum_{i=1}^{k-1}(T(y))_{i}} \\ &=\exp \left((T(y))_{1} \log \left(\phi_{1}\right)+(T(y))_{2} \log \left(\phi_{2}\right)\right) \\ &=\exp \left((T(y))_{1} \log \left(\phi_{1} / \phi_{k}\right)+(T(y))_{2} \log \left(\phi_{2} / \phi_{k}\right)\right) \\ &=b(y) \exp \left(\eta^{T} T(y)-a(\eta)\right) \end{aligned}
$$

其中：

$$ \begin{aligned} \eta &= \begin{bmatrix} \log (\phi _1/\phi _k)\ \log (\phi _2/\phi _k)\ \vdots \ \log (\phi _{k-1}/\phi _k)\ \end{bmatrix}, \ a(\eta) &= -\log (\phi _k)\ b(y) &= 1\ \end{aligned} $$

这样咱们就把多项式方程作为一个指数族分布来写了出来。

与 $i (for\quad i = 1, ..., k)$对应的链接函数为：

$$ \eta_i =\log \frac {\phi_i}{\phi_k} $$

为了方便起见，我们再定义 $\eta_k = \log (\phi_k/\phi_k) = 0$。对链接函数取反函数然后推导出响应函数，就得到了下面的等式：

$$ \begin{aligned} e^{\eta_i} &= \frac {\phi_i}{\phi_k}\ \phi_k e^{\eta_i} &= \phi_i \qquad\text{(7)}\ \phi_k \sum^k_{i=1} e^{\eta_i}&= \sum^k_{i=1}\phi_i= 1\ \end{aligned} $$

这就说明了$\phi_k = \frac 1 {\sum^k_{i=1} e^{\eta_i}}$，然后可以把这个关系代入回到等式$(7)$，这样就得到了响应函数：

$$ \phi_i = \frac { e^{\eta_i} }{ \sum^k_{j=1} e^{\eta_j}} $$

上面这个函数从$\eta$ 映射到了$\phi$，称为 Softmax 函数。

要完成我们的建模，还要用到前文提到的假设3，也就是 $\eta_i$ 是一个 $x$ 的线性函数。所以就有了 $\eta_i= \theta_i^Tx (for\quad i = 1, ..., k − 1)$，其中的 $\theta_1, ..., \theta_{k−1} \in R^{n+1}$ 就是我们建模的参数。为了表述方便，我们这里还是定义$\theta_k = 0$，这样就有 $\eta_k = \theta_k^T x = 0$，跟前文提到的相符。因此，我们的模型假设了给定 $x$ 的 $y$ 的条件分布为：

$$ \begin{aligned} p(y=i|x;\theta) &= \phi_i \ &= \frac {e^{\eta_i}}{\sum^k_{j=1}e^{\eta_j}}\ &=\frac {e^{\theta_i^Tx}}{\sum^k_{j=1}e^{\theta_j^Tx}}\qquad\text{(8)}\ \end{aligned} $$

这个适用于解决 $y \in{1, ..., k}$ 的分类问题的模型，就叫做 Softmax 回归。 这种回归是对逻辑回归的一种扩展泛化。

假设（hypothesis） $h$ 则如下所示:

$$
h_{\theta}(x)=\mathrm{E}[T(y) | x ; \theta]\\
=\mathrm{E} \left.\left[ \begin{array}{c}{1\{y=1\}} \\ {1\{y=2\}} \\ {\vdots} \\ {1\{y=k-1\}}\end{array}\right| x ; \theta \right]\\
=\left[ \begin{array}{c}{\phi_{1}} \\ {\phi_{2}} \\ {\vdots} \\ {\phi_{k-1}}\end{array}\right]\\
=\left[ \begin{array}{c}{\frac{\exp \left(\theta_{1}^{T} x\right)}{\sum_{j=1}^{k} \exp \left(\theta_{j}^{T} x\right)}} \\ {\frac{\exp \left(\theta_{2}^{T} x\right)}{\sum_{j=1}^{k} \exp \left(\theta_{j}^{T} x\right)}} \\ {\vdots} \\ {\frac{\exp \left(\theta_{k-1}^{T} x\right)}{\sum_{j=1}^{k} \exp \left(\theta_{j}^{T} x\right)}}\end{array}\right]
$$

也就是说，我们的假设函数会对每一个 $i = 1,...,k$ ，给出 $p (y = i|x; \theta)$ 概率的估计值。（虽然咱们在前面假设的这个 $h_\theta(x)$ 只有 $k-1$ 维，但很明显 $p (y = k|x; \theta)$ 可以通过用 $1$ 减去其他所有项目概率的和来得到，即$1− \sum^{k-1}_{i=1}\phi_i$。）

最后，咱们再来讲一下参数拟合。和我们之前对普通最小二乘线性回归和逻辑回归的原始推导类似，如果咱们有一个有 $m$ 个训练样本的训练集 ${(x^{(i)}, y^{(i)}); i = 1, ..., m}$，然后要研究这个模型的参数 $\theta_i$ ，我们可以先写出其似然函数的对数：

$$ \begin{aligned} l(\theta)& =\sum^m_{i=1} \log p(y^{(i)}|x^{(i)};\theta)\ &= \sum^m_{i=1}log\prod ^k_{l=1}(\frac {e^{\theta_l^Tx^{(i)}}}{\sum^k_{j=1} e^{\theta_j^T x^{(i)}}})^{1(y^{(i)}=l)}\ \end{aligned} $$

要得到上面等式的第二行，要用到等式$(8)$中的设定 $p(y|x; \theta)$。现在就可以通过对 $l(\theta)$ 取最大值得到的 $\theta$ 而得到对参数的最大似然估计，使用的方法就可以用梯度上升法或者牛顿法了。