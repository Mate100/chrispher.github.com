---
layout: post
title: ch5-神经网络基础
category: 机器学习
tags: [机器学习, PRML]
path: /notes/PRML/2015-2-13-ch5_Neural_Network-basic.md
description: 神经网络基础知识和使用BP算法来训练模型以及训练中的一些技巧
---

本篇主要是神经网络相关知识。笔记分两部分，这是第一部分：基础入门为主。

<!-- more -->

注：本文仅属于个人学习记录而已！参考Chris Bishop所著[Pattern Recognition and Machine Learning(PRML)](http://research.microsoft.com/en-us/um/people/cmbishop/PRML/)以及由Elise Arnaud, Radu Horaud, Herve Jegou, Jakob Verbeek等人所组织的Reading Group。

###目录
{:.no_toc}

* 目录
{:toc}

###1、引入

在线性回归与分类模型中，我们看到这些使用了基函数的线性模型具有很好的分析特性和计算特性，但是实际应用中受到维灾难的限制，因此我们有必要根据数据自动的调整这些基函数。方式之一是SVM(Support vector machines，之后会讲)，是通过先以训练数据点为中心定义一些基函数，之后在训练过程中选择子集(即选择支持向量)。SVM的优势之一是虽然训练包含了非线性优化，但是优化目标却是凸的，所以可以直接靠优化求解，最后得到的基函数会比实际训练数据少很多。另外一种关联向量机（relevance vector machine），它是从一堆基向量中选择基函数，会得到稀疏模型，与SVM不同，它得到的是概率输出，且是非凸优化。

还有一种方式预先定义了基函数，但是是可调整的（即基函数是含有参数的，可以在训练过程优化），比如多层感知器。一般而言这种方法得到的模型比较紧凑，在测试的时候会比较快捷，但是代价就是非凸优化。而“神经网络”这个词是源于在生物信息系统中，尝试用数学来表征信息处理的过程。这里模型已经不是单纯的模仿生物神经学习过程，而是有了一定的数学理论。

###2、前向网络

回想一下广义线性回归模型$$y(x, w) = f( \sum^M_{j=0} w_j \Phi_j(x))$$，这里w是模型的系数(权重)，而$$\Phi$$是选定的基函数。如果f是映射到唯一的，那么就是回归问题。如果是后验概率值，就是回归问题。对于前向神经网络(Feed-forward Neural Networks)而言，式子跟刚才的一样，但是用到两次，一次是定义基（基函数是非线性变换，如果是线性的话，和后面的线性组合就重复了，无意义），一次是获得线性组合的输出。而我们的神经网络就是这一系列的函数变换组合。

首先我们对输入做M个线性组合，如下：

$$a_j = \sum_{i=1}^D w_{ji}^{(1)} x_i + w^{(1)}_{j0}$$

这里(1)表示神经网络中的第一层，j取值是1到M，而参数$$w_{ji}^{(1)}$$表示权重，$$w^{(1)}_{j0}$$是偏置项(biases)，而$$a_j$$通常称之为激活量(activations)，之后对激活量做非线性变换得到$$z_j = h(a_j)$$，非线性函数也称激活函数，可以选择sigmoid函数，或者tanh函数等等。合起来就是最开始说的广义线性回归。$$z_j$$层在神经网络中，称之为隐含层。接着再继续上面的操作，得到下一层的激活量：

$$a_k = \sum^M_{j=1}  w_{kj}^{(2)} z_j + w^{(2)}_{k0}$$

注意权重w的下标，由ji到了kj。之后再跟着非线性变换，不断重复。到一定的层数后，跟一层输出层,这里只用一层隐含层后，输出层为$$y_k = \sigma(a_k);  \ \  \sigma(a) = \frac{1}{1+exp(-a)}$$。如果是多元分类，那么跟一层softmax 激活函数即可。那么整个结构如下图所示（每个激活量对应的圆称之为神经元）：

<img src="/images/prml/ch5_basic_nural_network.jpg" height="100%" width="100%">

把中间输入输出合并起来如下：

$$y_k(x, w) = \sigma(\sum^M_{j=1} w^{(2)}_{kj} h(\sum^D_{i=1} w^{(1)}_{ji} x_i + w^{(1)}_{j0})+ w^{(2)}_{k0})$$ 

我们看到神经网络就是输入变量的多次非线性变换和线性组合得到输出，其中w是可以调整的参数。这种就是前向传递，把x输入之后，我们就能够得到输出y。需要注意的是上面的图并不代表概率图。之后，我们后用概率来解释神经网络。

当然，我们可以在输入中增加一列1，那么就可以把第一层的偏置项舍去了，同时为了简化，我们把隐含层的偏置项也舍去，那么可以得到简化的神经网络如下：

$$y_k(z,w) = \sigma(\sum^M_{j=0} w^{(2)}_{kj} h(\sum^D_{i=0} w^{(1)}_{ji} x_i ))$$

我们简单的说一下他的一些特点。如果隐含层也是线性的话,那么线性变换组合仍是线性的，隐含层就没有意义了。此外，如果隐含层神经元的数量少于输入维数或输出维数，那么神经网络可以看成一种降维模式，这在ch12章会讲述到。此外，上面说的神经网络是基础的网络，也可以有很多其他种类的神经网络，比如：

- 有多层隐含层
- 每个神经元不一定全部连接到下一层
- 有些链接可以跳过一些层或者多个层
- 两层及两层以上的网络，理论上可以以任意精度逼近任意函数
- 有许多对称权重空间，即对于相同的输入，不同的权重也可能有相同的输出

###3、网络训练

对于连续输出y，训练网络的一个简单想法是最小二乘法，如下：

$$E(w) = \frac{1}{2} \sum^N_{n=1} \mid \mid y(x_n, w) - t_n \mid \mid^2$$

当然，这里我们使用最大似然估计（最终得到的结果一致），假设$$p(t \mid x,w) = N(t \mid y(x, w), \beta^{-1})$$, 那么对数似然函数为：

$$\frac{\beta}{2} \sum^N_{n=1}(y(x_n, w) - t_n)^2 - \frac{N}{2} \ln \beta + \frac{N}{2} \ln(2 \pi)$$

令w的导数为0（之后，我们考虑贝叶斯下的神经网络），得到与最小二乘法一致的结果。需要注意的是在神经网络模型中，我们更倾向于考虑最小误差函数，而不是对数似然函数。这里需要注意的是，误差函数存在许多局部极小值，因此导数为0的点并不一定是全局最优解！在得到$$w_{ML}$$之后，我们对$$\beta$$取导数得到：$$\frac{1}{\beta_{ML}} = \frac{2}{N} E(w_{ML})= \frac{1}{N} \sum^N_{n=1} (y(x_n, w_{ML}) - t_n)^2$$。如果我们的目标值是多元的，那么$$\frac{1}{\beta_{ML}} = \frac{2}{NK}E(w_{ML}) =\frac{2}{NK} \sum^N_{n=1} (y(x_n, w_{ML}) - t_n)^2$$，K对应目标值的数量，我们这里假设了他们共享同一个精度。

此外，如果我们单层的看，每层的输出是$$y_k = a_k$$，那么有$$\frac{\partial E}{\partial a_k} = y_k - t_k$$，这是为后面提出误差反馈(error backpropagation)做准备。

现在考虑二元分类和多元分类。区别不大，可以类比于之前提到的logistic回归和softmax回归。对于二元分类，$$y = \sigma(a) = \frac{1}{1+exp(-a)}$$，那么有$$p(t \mid x,w) = y(x, w)^t (1 - y(x,w))^{1-t}$$，他的误差函数就是交叉熵，如下：

$$E(w) = -\sum^N_{n=1} (t_n \ln y_n + (1-t_n) \ln (1-y_n))$$

这里的$$y_n$$表示$$y(x_n,w)$$。Simard发现：分类问题中，使用交叉熵，相比于最小二乘法训练更快，泛化效果更好。对于K元分类，我们的似然函数为$$p(t \mid x,w) = \prod^K_{k=1} y_k(x,w)^{t)k}(1 -y_k(x,w))^{1-t_k}$$，取负对数之后得到误差函数如下：

$$E(w) = - \sum^N_{n=1} \sum^K_{k=1} (t_{nk} \ln y_{nk} + (1-t_{nk}) \ln (1 -y_{nk}))$$

这里的$$y_{nk}$$ 表示$$y_k(x_n,w)$$。对比一下之前的线性分类模型。我们以标准的2层网络来对比，我们看到第一层中的权重，在线性分类下是独立求解的，而在神经网络中是相互影响的，而且各个输出直接是共享输入特征的。另外第一层可以看成是非线性特征变换，而这些不同输出之间的特征共享，可以节约计算，和提高模型泛化能力。最后，多元分类最后可以输出多个神经元（多个激活值），如下：

$$y_k(x,w) = \frac{exp(a_k(x,w))}{\sum_j exp(a_j(x,w))}$$

这里需要注意的是$$0 \le y_k \le 1; \sum_k y_k = 1$$。注意$$a_k(x,w)$$增加一些值并不影响结果，但是会导致某些特征空间方向是一个常数值(某个y不需要求，可以用1减去其他的y值得到导致的，这个问题可以通过增加正则项解决)。另外，针对激活值的导数，形式一致。

###4、参数优化
在上一节我们简单的说明了如何训练网络，这里着重讲了如何具体的得到参数值，即如何得到向量w使得E(w)最小。令梯度等于0，即$$\nabla E(w) = 0$$这样能够得到函数的极值，但不一定是最小值。虽然我们的目标是全局最小值，但是误差函数具有高度的非线性，使得其有非常多的局部最小值，因此基本不可能知道得到的结果是不是全局最小值。所以，我们还是采用迭代的方法进行参数优化，即初始化$$w^{(0)}$$,之后根据$$w^{(\tau+1)} = w^{(\tau)} + \Delta w^{(\tau)}$$来更新权重值。不同的算法在选择$$\Delta w^{(\tau)}$$上有不同的策略，多数算法选择使用梯度信息。为了明白梯度信息的重要性，我们使用泰勒展开式来分析误差函数的局部近似解。

对于很多函数，一般用泰勒展开到二次项就可以，对误差函数E(w)的泰勒展开式如下：

$$E(w) \approx E(\hat(w)) + (w - \hat(w))^T b + \frac{1}{2}(w - \hat(w))^T H (w - \hat(w))$$

这里的$$b=\nabla E \mid _{w=\hat(w)}$$，是E在$$w=\hat(w)$$的一阶导数值，而$$H = \nabla \nabla E$$，是hessian矩阵，元素值是是E在$$w=\hat(w)$$的二阶导数值。如果我们把E的一阶导数展开(只展开到一阶)，那么会有$$\nabla E \approx b + H(w-\hat(w))$$。对于一个局部最小值$$w^*$$，E的一阶导数为0，那么我们有：

$$E(w) = E(w^*) + \frac{1}{2}(w - w^*)^T H (w - w^*)$$

这里的H是E在$$w^*$$处的hessian矩阵。我们考虑一下它的几何性质，H的特征向量是u，即$$Hu_i = \lambda_i u_i$$，这里我们用特征向量的线性组合来表示$$w-w^*$$，即$$w-w^* = \sum alpha_i u_i$$，即相当于把原始的点投影到特征向量所在的平面。那么误差函数就可以写为$$E(w) = E(w^*) + \frac{1}{2} \sum_i \lambda_i \alpha_i^2$$。这里的H是正定的，即对于任意v，有$$v^THv > 0$$，还可以得到它的所有的特征值都是正的。

那回到刚才的问题，为什么梯度信息是重要的呢？因为使用梯度信息能够明显的提高误差函数优化的速度。具体分析如下：在误差函数的二阶近似中，我们看到误差平面是由b和H决定的，一共包含W(W+3)/2个参数(H是对称的，W是指w的维度)，那么局部最优解的是依赖于$$O(W^2)$$个参数,我们至少需要收集$$O(W^2)$$个独立信息。如果不使用梯度信息，那么对这些$$O(W^2)$$信息还需要做$$O(w)$$次评估。简单的解释，就是使用梯度信息，能够比是不用梯度信息减少得到最小值的要走的步数。

最简单的梯度下降应用是$$w^{(\tau+1)} = w^{(\tau)} + \eta \nabla E(w^{(\tau))}$$，这里$$\eta$$是学习速率(learning rate).这里可以使用所有的数据集进行一次权重的更新，也可以使用一部分数据进行权重更新。一次使用所有的数据进行更新称之为batch方法，这里的batch数等于整个训练集大小。尽管梯度下降看起来很合理，但是事实表明这是一个很烂的方法，后面会有提及。当然，使用batch的方法，也可以使用共轭梯度下降或者牛顿法，这些方法会更鲁棒且收敛更快，因为与梯度下降不同，这些方法会使得误差在每次迭代中都会下降，除非已经达到了最小值。

为了得到一个好的最小值，可能需要跑多次梯度下降，使用不同的初始值，之后在独立的测试集合中对比最后的结果，选择较优解。另外，on-line下的梯度下降在大数据集上训练神经网络模型会更加的实用。其中一个on-line比batch好的地方是在处理冗余数据中，on-line算法更加的高效。比如训练数据重复一遍之后，误差是都加倍了，但是batch算法中计算加倍了。另外一点是on-line算法每次都是根据单个点的梯度计算，可能不是全局的梯度，导致会错过一些局部最优解。

###5、误差反馈算法

我们知道要采用梯度下降的方法进行参数优化，这里我们主要讨论如何高效的得到误差的梯度。这里引入了误差反馈(error backpropagation，误差反向传播)的方法，即使用在网络中向前和向后传递的一些局部信息。需要注意backpropagation在神经网络中，表示了很多东西，可能是指模型，也可能指算法。

使用迭代算法的过程中，主要包括两个步骤：

- 1、计算权重下的梯度，包括了误差在网络中的反向传播；
- 2、根据梯度调整权重；

这两步是独立的，backpropagation可以用于其他的神经网络模型训练。另外第二步也有很多比梯度下降更有效的方法。

接下来我们开始推导。对于服从iid的训练集数据，误差是$$E(w) = \sum_{n=1}^N E_n(w)$$，首先我们考虑线性模型，输出是$$y_k$$，即$$y_k = \sum_i w_{ki}x_i$$，那么误差函数可以细化为$$E_n = \frac{1}{2} \sum_k (y_{nk} - t_nk)^2$$，这里的$$y_{nk} = y_k(x_n, w)$$，那么误差的梯度就是:

$$\frac{\partial E_n}{\partial w_{ji}} = (y_{nj} - t_{nj})x_{ni}$$

这就可以解释为“局部”计算，是在$$w_{ji}$$层的输出的误差信号$$y_{nj} - t_{nj}$$与在$$w_{ji}$$层的输入变量$$x_{ni}$$之间的乘积。当然，之前我们也看到了使用sigmoid函数或softmax的结果（交叉熵）与上式很类似。接下来我们看看这种简单的形式，可以扩展成什么样子。

对于一般的前向传递神经网络，每一层计算$$a_j = \sum_j w_{ji} z_i$$，这里$$z_i$$是上一层的单个神经元的激活值。对于j层，$$z_j = h(a_j)$$，这里的h是非线性变换，可以选择sigmoid函数等等。这里的输入单元和输出单元都可以是一个神经元，也可以是多个神经元。对于每个类的训练集，我们输入到网络，得到所有隐含层和输出层的激活值，这个过程称之为前向传递(forward propagation)。

接下来就是计算梯度了，这里注意下标ij都表示层数，ji的意思是j层到i层间(反馈的顺序，正向的顺序是i到j)。根据链式求导法则，我们有：

$$\frac{\partial E_n}{\partial w_{ji}} = \frac{\partial E_n}{\partial a_j} \frac{\partial a_j}{\partial w_{ji}}$$

这里我们引入一个表示$$\sigma_j \equiv \frac{\partial E_n}{\partial a_j}$$。那么我们可以看到$$\frac{\partial a_j}{\partial w_{ji}} = z_i$$,即$$\frac{\partial E_n}{\partial w_{ji}} = \sigma_j z_i$$。可以看到，权重的梯度就是j层的$$\sigma$$和i层的激活值z的乘积，(注意z=1的情况是bias，即偏置项)他们之间的关系如下图所示（蓝色箭头表示正向传递的方向）：

<img src="/images/prml/ch5_backpropagation.jpg" height="80%" width="80%">

那我们的核心就是得到$$\sigma$$对于输出层，$$\sigma_k = y_k - t_k$$，对于j层，我们有：

$$\sigma_j \equiv \frac{\partial E_n}{\partial a_j} = \sum_k \frac{\partial E_n}{\partial a_k} \frac{\partial a_k}{\partial a_j}$$

注意到$$\frac{\partial a_k}{\partial a_j} = \frac{\partial a_k}{\partial z_j} \frac{\partial z_j}{\partial a_j} = w_{kj} h'(a_j)$$，结合上面几个式子，我们得到backpropagation的式子：$$\sigma_j = h'(a_j) \sum w_{kj} \sigma_k$$

那么erro backpropagation可以归纳为：

- 1、输入x，根据前向传递得到隐含层和输出层的各个激活值
- 2、计算输出层的$$\sigma_k$$
- 3、根据误差反馈,得到不同隐含层的$$\sigma_j$$
- 4、根据式子$$\frac{\partial E_n}{\partial w_{ji}} = \sigma_j z_i$$得到梯度值

该算法的效率比标准的数值法要高，比如权重和偏置项的总维度是W，那么误差反馈的时候需要$$O(W)$$，不需要再次计算前向传递的值，而标准的方法中是需要不断的计算误差前向和反向的变化，需要$$O(W^2)$$。

之后，课本上讲了用BP算法来获取jacobian矩阵和Hessian矩阵的应用，不再赘述。

###6、正则化

####6.1权重衰减
在神经网络中，输入和输出层是由训练数据决定的，但是隐含层的数量以及隐含层节点数量都是可调的，以达到最好的预测效果。假设隐含层所有神经元个数为M，在神经网络中，存在许多局部最优解，泛化误差并不是M的简单函数。在实际中，如何选择一个合适的M呢？我们可以选择不同的M，之后通过测试选择最优的结果。

然而，有很多方法来控制模型的复杂度来避免过拟合。类似于之前的线性多项式拟合，我们可以选择一个比较大的M，之后通过增加正则项来控制模型的复杂度。最简单的正则项是二次型，即$$\hat(E(w)) = E(w) + \frac{\lambda}{2}w^Tw$$，注意这里并没有对偏置项进行正则！增加正则项又称之为权重衰减(weight decay)。模型复杂度则由正则系数$$\lambda$$决定。

然而使用简单的权重衰减的不足之一是，它和神经网络映射中存在的缩放性有矛盾。即如果我们用原始数据和对原始数据（输入或输出）做一些线性变换，我们希望对于权重的影响也是对应的线性变换（否则不同的线性变换就会得到完全不可预期的模型），但是用简单的权重衰减并不能达到这个效果，因为简单的权重衰减把不同层的权重值放在了同一个标准上。所以，简单的方法是不用的层用不用的正则系数，即$$\frac{\lambda_1}{2}\sum_{w \in W_1} w^2 + \frac{\lambda_2}{2}\sum_{w \in W_2} w^2$$。这种正则相当于一种先验，$$p(w \mid \alpha_1,\alpha_2) \propto exp(\frac{-\alpha_1}{2}\sum_{w \in W_1} w^2 - \frac{-\alpha_2}{2}\sum_{w \in W_2} w^2)$$。需要注意的是，这个先验并不正确，它不能被归一化因为偏置项没有受限。所以，通常我们也会正则化偏置项（这样破坏了移动不变性，shift invariance）。

####6.2提前结束
另外一种正则化的方法是提前结束(early stopping)训练。在神经网络模型训练过程中，训练误差通常都是不断降低的，然而在另一个独立的验证集（valid）上却一般是先减少，之后过拟合了会导致验证集误差增大的。所以一个可行的选择是在验证集上误差最小时，停止训练，之后可以在测试集上测试泛化误差（这里数据集分割成了三个，训练集、验证集、测试集）。

提前结束有时也可以定性的解释为有效的网络自由度。在达到训练误差的最小值之前结束训练，表示了对模型复杂度的一种限制。在二次型误差函数中，使用提前结束和简单的权重衰减的具有相似的行为特征。

####6.3不变性
在很多模式识别应用中，要求在输入变量有一定的变换中，预测值具有不变性(invariant)。比如二维的图像数据，一些缩放或者点的波动，对原始数据具有很大的影响，但是不能影响最后的分类结果。当然，如果数据集足够大，那么神经网络模型就能够学习到这种不变性。但是，如果训练数据有限，或者有多种不变性（导致数据需要指数增长），那么这种方法就不切实际了。我们需要寻找其他一些方法，主要可分四种方法：

- 1、在训练集合中增加一些所需的不变性的模式（在训练数据集中引入噪声）。比如手写识别数据，我们混合由原始数据往不同的方向偏移来获生成的数据；
- 2、在正则项中引入对模型输出变化的惩罚项。在后面会提到，即Tangent propagation；
- 3、在预处理特征中清洗掉这些波动性，得到具有不变性的特征；
- 4、在神经网络模型中，利用模型特性或定义一些核函数来处理掉这些波动而获得不变性特征。比如使用局部特征表达和权值共享，后面会提到。

方法1是最经常使用的方法，比较容易实施，也可以引入特别复杂的不波动。方法3也是可以的，但是我们将很难手动设计一些所需的不变性的特征，也没有考虑哪些信息是有利于判别的。

####6.4切线反馈
切线反馈（Tangent propagation），是指如果这个特征变换由参数$$\xi$$决定，那么我们在误差函数中需要引入该变换对目标的影响，即$$\frac{\partial y_{nk}}{\partial \xi} \mid _{\xi = 0}$$，此时引入的正则项是：

$$\lambda \frac{1}{2} \sum_n \sum_k (\frac{\partial y_{nk}}{\partial \xi} \mid _{\xi = 0})^2 = \frac{1}{2} \sum_n \sum_k (\sum_{i=1}^D J_{nki} \tau_{ni})^2$$

这里J是jacobian矩阵，可以使用BP算法来求取。

####6.5使用变换后数据
在上一小节，我们引入了特征变换$$s(x,\xi)$$，由$$\xi$$决定，这里我们考虑$$\xi$$是由分布$$p(\xi)$$，那么误差函数可以表示为：

$$\hat(E) = \frac{1}{2} \int \int \int (y(s(x,\xi)) - t)^2 p(t \mid x) p(x) p(\xi) dx dt d \xi$$

这里的$$p(\xi)$$具有0均值和低方差。用泰勒展开式和一些推导后，可以得到正则项如下：

$$\omega = \frac{1}{2} \int (\tau^T \nabla y(x))^2 p(x) dx$$

如果考虑输入数据随机增加噪声，那么正则项的形式为$$\omega = \frac{1}{2} \int \mid \mid \nabla y(x) \mid \mid ^2 p(x) dx$$，称之为Tikhonov正则项。对于小噪声而言，该正则一般会有很好的泛化误差。