<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
作者：Michael Nielsen
英文在线：http://neuralnetworksanddeeplearning.com/index.html

# 改进神经网络的学习算法
<!--more-->  
一个高尔夫运动员刚开始学习高尔夫的时候, 大部分时候都在学习基础挥杆动作. 但他们会渐渐学会高级杆法. 同样的, 我们也在接下来会学习如何改进反向传播算法, 从而进一步改进我们在第一章的神经网络的性能.  
本章我们将要讨论的东西包括一个改进的代价函数--"交叉熵代价函数(cross-entropy cost funciton)". 四种"正则化"方法((L1 and L2 regularization, dropout, and artificial expansion of the training data). 这个帮助我们的网络更好的工作. 本章还会介绍如何更好的选取网络权重和偏置的初始值. 并且还会讨论一些方法以更好的选择hyper-parameters. 本章还会简要讨论几种其他技术, 每个讨论都是分开独立的, 读者完全可以略过自己不感兴趣的内容. 我们还会通过代码实现我们讨论的改进方法, 来改进第一章所使用的识别手写数字的神经网络.  
当然, 我们的讨论只涵盖了众多神经网络技术中的很小的部分. 但这一小部分都是很重要的部分. 掌握这些重要的技术可以加深我们对神经网络可能出现的问题的理解, 同时这也将使你准备好在需要的时候采用其他的技术.  
## 交叉熵(cross-entropy)代价函数
我们大多人是讨厌错误的. 我第一次在观众面前弹钢琴时, 感觉很紧张, 结果表现很差. 我很困惑, 因为我不知道哪里弹错了, 直到有人给我明确的指出来才发现. 但接下来我就很快的改正了, 然后剩下的演出我弹的好了很多. 我想说的是, 当错误可以被明确的指出来时, 这可以更快的帮助我们学习, 这同样使用于训练神经网络.  
理想情况下, 我们希望我们的神经网络可以快速的学习, 但实际中是这样么? 为了回答这个问题, 看下面的这个简单的例子, 这个神经元只有一个输入:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/1.png?raw=true)  
我们来训练这个神经网络做个比较傻的事情, 就是把输入1变成输出0. 这很简单, 我们即使不借助任何帮助, 手动也能算出合适的权重和偏置. 但是, 让我们上面的这个神经网络是如何做这件事情的吧.  
先说一下, 我选择初始化权重为0.6, 初始化偏置为0.9. 这只是一次普通的选择, 我没有采用什么原则去筛选. 初始时候神经元输出了0.82. 想要得到输出0, 我们的神经元还需要进一步的学习. 代价函数的形式采用第一章所提到的, 学习速率\\(\eta = 0.15\\). 下面这个图显示了学习曲线(需要说的一点是, 作者的英文原文这里是一个动态图, 这里只是将最后运行的结果截图放这里了).  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/2.png?raw=true)  
从曲线图里可以看到, 神经元很快就学到了合适的权重和偏重, 得到了输出为0.09, 这虽然不是0, 但已经很好了. 现在让我们选择初始的权重和偏置为2. 在这种情况下, 我们初始得到神经元的输出为0.98, 这有点离谱了. 看下面的图展示了这种情况下这个神经元的学习曲线:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/3.png?raw=true)  
学习速率依然选择\\(\eta = 0.15\\), 从上面的曲线可以看到初始时候学习的很慢. 但训练的批数大概过了一半之后, 学习就大大加快了.  
上面这个曲线在我们人类看来是很奇怪的. 我在本章开始的时候提到, 当我们明确知道我们错了的时候, 我们学习的很快, 但我们这个神经元一开始不仅表现的输出错的离谱, 而且学习起来还很慢. 而且, 事实上这个不仅存在于我们这个简单的神经元模型中, 在一般的神经网络中也是存在的. 这是为什么? 我们可以做什么来改进这一点呢?  
考虑到我们的神经元是根据两个偏微分来改变权重和偏置, 也就是\\(\partial C / \partial w\\)和\\(\partial C / \partial b\\). 因此说"学习速度慢", 其实就是这些偏微分的值小. 我们下面将给出计算偏微分的公式. 我们采用quadratic cost function, 即公式(6), 可以得到:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/4.png?raw=true)  
\\(a\\)是输入\\(x=1\\)时, 神经元的输出, \\(y=0\\)是对应的需要的输出. 为了更明显一点, 我们还记得\\(a = \sigma (z)\\), \\(z=wx+b\\). 应用微分的链式传导法则可以得到:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/5.png?raw=true)  
因为我将\\(a=1, y=0\\)代入了, 因此才可以得到最右边的公式. 我们将会进一步研究\\({\sigma}^/ (z)\\). 现在我们回顾下\\(\sigma\\)函数的图形的样子:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/6.png?raw=true)  
从图上我们可以看到, 当神经元的输出接近1时, 函数变的很平缓, 也就是说\\({\sigma}^/ (z)\\)很小. 通过公式(55)和(56)可以得到\\(\partial C / \partial w\\)和\\(\partial C / \partial b\\)也会很小. 这正是神经元学习变慢的原因. 接下来我们将看到, 对与一般意义上的神经网络, 这个结论也是存在的.  

### 引入交叉熵代价函数(cross-entropy cost function)
如何解决前面提到的学习变慢的问题呢? 我们可以通过引入一种新的代价函数--交叉熵代价函数. 为了理解交叉熵, 我们来看下面的模型. 这个模型有多个输入\\(x_1, x_2,....,\\), 相应的权重为\\(w_1, w_2, ....,\\), 偏置为\\(b\\).  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/7.png?raw=true)  
这个神经元的输出, \\(a = \sigma (z)\\), 其中\\(z = \sum_j w_j x_j + b\\), 表示输入的加权和. 我们定义交叉熵函数如下:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/8.png?raw=true)  
\\(x\\)表示输入的训练数据, \\(n\\)表示输入数据的总个数.  
上面的公式并不能很直观的看出来解决了我们之前提到的问题, 甚至作为一个代价函数也不是很直观. 我们先来解释他如何作为一个代价函数吧.  
函数所具有的两个属性使他可以被用作代价函数, 首先, 他是非负, 因为对于这个求和操作的每一个独立的元素都是小于0的(\\(0<a<1\\)). 然后前面的因子\\(\frac{-1}{n}\\)是负, 因此整个代价函数是恒小于0的.  
然后, 还可以看到, 当神经元的输出跟想要的输出越接近, 这个函数的值就越接近0. 我们假设训练数据\\(a \approx 0\\), 同时我们希望的输出是0. 当神经网络是按我们预想工作的时候, 输入输出就是这样的. 这样, 公式(57)中的第一部分就是0了. 第二部分也会趋向0, 因为第二部分趋向于\\(-\ln 1\\). 当想要的输出是1, 同时\\(a \approx 1\\)可以得到类似的结论.  
总结来说, 交叉熵(cross-entropy)函数是恒为正数, 并且结果越趋近0, 表明神经网络的输出越接近我们想要的输出. 这两个特性正是它可以作为代价函数使用的原因. 当然跟之前我们采用的代价函数想比, 交叉熵函数还具有个更吸引人的特性就是他可以避免前面的学习变慢的问题. 为了说明这个问题, 我们计算交叉熵函数对权重的偏微分. 将\\(a = \sigma (z)\\)代入公式(57), 并且应用链式法则我们可以得到:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/9.png?raw=true)  
对上面的公式进行化简可以得到:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/10.png?raw=true)  
考虑到\\(\sigma (z) = 1/(1+ e^{-z} )\\), 可以得到\\({\sigma}^/ = \sigma (z) (1 - \sigma  (z))\\), 上面的公式被进一步简化为:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/11.png?raw=true)  
上面这个表达式简直美极了. 我们可以看到权重迭代学习的速率取决于\\(\sigma (z) - y\\), 也就是输出与实际的差值. 差值越大, 神经元学习的越快. 这正是我们想要的. 同样的, 我们可以得到下面偏置迭代学习的公式:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/12.png?raw=true)  
现在, 让我们采用交叉熵函数来做代价函数, 重新训练我们之前提到的单一神经元. 首先选择初始化权重为0.6, 初始化偏置为0.9  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/13.png?raw=true)   
跟之前一样, 学习的速率很快, 这没什么让人兴奋的. 我们来看之前我们表现糟糕的情况,  选择初始的权重和偏置为2  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/14.png?raw=true)   
结果很让人满意, 学习速率变得很快.  
我在换了代价函数后, 没有说采用的\\(\eta \\)是多少. 在采用老的代价函数时, \\(\eta = 0.15\\). 我们是否需要在新的模型里也使用同样的\\(\eta \\)呢? 其实这很难比较, 因为代价函数变了. 事实上, 在换了代价函数后, 我选择的\\(\eta = 0.005\\).  
事实上我们之前关注的一直是神经元在输出明显错误时候的学习速率, 而并非整个学习过程运行的绝对速率, 因此, 这时候讨论\\(\eta\\)是没有太大意义的.  
交叉熵代价函数很容易推广到多个神经元, 多层网络的场景. 假设\\(y = y_1, y_2,.....\\)是我们希望得到的输出. \\(a_1^L,  a_2^L...\\)是神经元的实际输出. 交叉熵函数可以定义如下:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/15.png?raw=true)   
顺便说下, 我一直使用的交叉熵这个词, 可能会让人有点疑惑, 因为你可能在其他地方也听说过这个词. 实际上, 我们可以借助于定义两个变量的随机分布来理解他. \\(p_j, q_j\\), 他们的随机分布公式为 \\(\sum_j p_j \ln q_j\\). 如果我们将单个神经元的输出看作是输入a和(1-a)的一个随机分布, 这个定义是可以跟公式(57)联系起来的.  
然而, 当我们的神经网络在最后一层包含多个S型神经元时, \\(a_j^L\\)一般都不会服从某种随机分布. 因此, 像\\(\sum_j p_j \ln q_j\\)这样的定义几乎没什么意义. 你可以认为公式(63)是在每个神经元上计算交叉熵然后再取和. 这样, 你可以在单个神经元上应用这个随机分布. 在这个意义上, 可以认为公式(63)是个随机分布的交叉熵.  
我们应该在哪些场景下使用交叉熵代价函数替代我们在第一章所采用的方差代价函数(quadratic cost)? 事实上, 在任何情况下, 交叉熵代价函数几乎都能表现出更好的性能.  
### 在识别MNIST手写数字中使用交叉熵代价函数  
将交叉熵代价函数作为一部分加入我们的梯度下降算法和反响传播算法中是很容易的. 接下来我将会写一段新的程序, ```network2.py```, 除了使用交叉熵作为代价函数外, 还会加入这一章将会讨论的其他一些技术. 现在, 先让我们看看新的程序在识别手写数字方面的表现吧. 跟第一章一样, 我们的神经网络采用30个隐藏神经元. 每一批的训练数据size是10, 学习速率\\(\eta = 0.5\\), 我们将会训练30批数据. ```network2.py```的接口跟之前不太一样, 但理解起来不会存在什么问题. 在Python Shell里, 你可以通过键入```help(network2.Network.SGD)```看下文档.  
```  
>>> import mnist_loader
>>> training_data, validation_data, test_data = \
... mnist_loader.load_data_wrapper()
>>> import network2
>>> net = network2.Network([784, 30, 10], cost=network2.CrossEntropyCost)
>>> net.large_weight_initializer()
>>> net.SGD(training_data, 30, 10, 0.5, evaluation_data=test_data,
... monitor_evaluation_accuracy=True)
```  
说明一下, ```net.large_weight_initializer()```用来对权重和偏置进行初始化, 初始化的方式与第一章一样. 我们在一会儿还会修改这个初始化方式, 但是现在先让我们维持原样. 我们这次得到的识别准确率有95.49%, 跟我们在第一章使用quadratic cost得到的结果差不多.  
当我们将隐藏神经元的数量提到100时, 这次我们得到了96.82%的准确率. 这比我们在第一章时候得到的96.59%的准确率要高了一些. 这看起来只是一个很小的提升, 但是如果看错误率的话, 我们看到, 错误率从3.41%降到了3.18%, 还是很可观的.  
看起来交叉熵代价函数确实是一个更好的选择. 然而目前还不能确定交叉熵足够优秀, 接下来我们将会花费一些精力去学习如何选择学习速率, 控制每一批学习数据的大小等. 我们会看到, 交叉熵代价函数绝对是一个更好的选择.  
顺便说一句, 目前我们讲到的是我们这一章将要介绍的一般性方法的一部分. 接下来, 我们将会介绍一些新的技术, 可以帮助我们得到更好的结果. 看到结果得到改进总是令人兴奋的, 但如何解释这种改进通常是个棘手的问题. 我们通常都要耗费很多工作在优化其他hyper-parameters上, 然后才可以看到在结果上的改进. 这将需要强大的计算能力, 甚至耗费很长的时间. 我们在本章将会在一些基础模型上讨论优化方法, 但你要记住, 这些测试没有经过严格的论证, 现实中要保持警惕, 因为很多时候进展并非像教程中说的这般顺利.  
好了, 我们将交叉熵已经够多的了. 为什么要讲这么多在这上面? 尤其是考虑到这只带来了一点点很小的改进. 接下来我们将会讲到其他的技术, 特别介绍下--regularization, 这将会给我们带来很大的改进. 那么, 为什么花这么多精力介绍交叉熵代价函数呢? 部分原因是交叉熵函数是一个使用很广泛的代价函数, 这值得我们去多了解一些. 但更重要的原因是神经元学习速度慢是一个很普遍的问题, 我们在本章会一遍一遍的反复看这个问题. 我花了这么多话介绍交叉熵函数是因为在解决神经元学习速度慢上这是一种方法.  
### 交叉熵从何而来  
目前为止, 我们对交叉熵的讨论一直是代数上的, 实际应用上的. 这很有用, 但依然还有些问题没有回答. 比如: 交叉熵到底是什么意思? 有没有更直观的解释? 交叉熵这个概念是如何第一次出现在我们脑海中的呢?  
让我们从最后一个问题开始. 假设我们发现了神经网络学习速率很慢的问题, 并且我们确定根源在公式(55)和(56)中的\\({\delta}^/ (x)\\). 盯着这些公式看一会儿之后(可能是好一会儿), 我们可能会想, 如果能让\\({\delta}^/ (x)\\)消失就好了. 这样的话, 对输入单一的输入\\(x\\), 代价\\(C = C_x\\)将会满足下面的公式:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/16.png?raw=true)   
如果我们找到一个代价函数使得上面的设想成真, 我们显然可以得到一个神经网络, 越是错误明显越是学习的快. 接下来你将会看到, 基于上面的公式,我们最终可以得到交叉熵. 根据链式法则, 我们得到:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/17.png?raw=true)   
代入\\({\delta}^/ (z) = \delta (z) (1 - \delta (z))=a(1-a)\\)上面的公式变为:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/18.png?raw=true)   
比照公式(72)我们得到:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/19.png?raw=true)   
在a上积分得到:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/20.png?raw=true)   
当然, 我们的输入不止一个, 因此推广开来在输入上取平均得到下面的式子:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/21.png?raw=true)   
这里的constant是在所有输入上取平均的constant. 好的, 我们得到了交叉熵函数, 这证明了交叉熵函数不是我们凭空捏造的. 我们可以很自然很简单的发现他.  
关于交叉熵更直观的解释是什么呢? 深入解释这一点将会比较困难. 然后, 还是有必要提一下, 关于交叉熵的标准解释来自于信息论. 简单来说, 交叉熵用来衡量不确定性. 我们的神经网络是要计算下面的函数\\(x \rightarrow y = y(x)\\). 而信息论其实是计算这个函数\\(x \rightarrow a = a(x)\\). 假设我们认为a是我们网络输出y是1的概率, \\(1-a\\)是输出0的概率. 那么, 交叉熵函数就是衡量结果在多大程度上让我们"surprised"的程度.  一般来说, 但我们得到的输出就是我们想要的, 我们不会感到很"surprised", 如果输出跟我们想要的相差比较远, 我们就会感觉很"surprised". 在信息论中, 对"surprised"的定义会有更明确的解释. 抱歉我不知道如何简短的明确的解释这个概念. 在Wikipedia上有关于这个的一个[简单描述](https://en.wikipedia.org/wiki/Cross_entropy#Motivation), 另外如果还想了解更多细节, 可以参考[Cover and Thomas](http://books.google.ca/books?id=VWq5GG6ycxMC)所著信息论的第五章.  
### Softmax  
本章我们主要是采用交叉熵函数来解决学习速率慢的问题. 但我还是想大致的描述另外一个方法, softmax神经元层. 我们并不打算马上采用softmax层, 所以, 如果你着急, 你可以跳过这部分. 但是, softmax依然是值得去了解下的. 一方面是这个东西挺有意思, 另外一个原因是在第六章我们介绍深度学习网络时, 会采用softmax.  
softmax的想法来自于想给我们的神经网络定一个不同的输出层. 就像我们在S型神经元用到的. 定义加权输入为\\(z_j^L = \sum_k w_{jk}^L a_k^{L-1} + b_j^L\\). 但是我们得到输出所采用的公式略有不同, 我们在softmax层对\\(z_j^L\\)应用softmax函数, 我们得到第j个输出神经元的激励\\(a_J^L\\)为:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/22.png?raw=true)   
分母是我们在所有输出上取和.  
如果你对softmax函数不熟悉. 那么公式(78)看起来将会很晦涩. 很难一样看出来这是我们想要的. 而且很难直观的看出来这有助于我们解决神经元学习速率慢的问题. 为了更好的理解公式(78), 假设我们有一个神经网络, 有四个输出. 相应的有四个加权输入, \\(z_1^L, z_2^L, z_3^L, z_4^L\\). 下面的图左边是可能的输入(强烈建议去作者原链接去看下这个图), 拉动滑块可以改变输入的大小, 右边是对应的输出.  开始我们试着拉动滑块增大\\(z_4^L\\):  
 ![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/23.png?raw=true)   
 随着\\(z_4^L\\)的增大, 你可以看到对应输出的\\(a_4^L\\)增大, 并且可以看到其他输出的减少(强烈建议读者去作者英文原文试验下, 非常漂亮). 类似的, 如果你减小\\(z_4^L\\), 那么\\(a_4^L\\)将减小, 同时其他的输出将变大. 事实上, 如果你仔细观察, 会发现在任何一种情况下, 其他输出的变化都是对\\(a_4^L\\)的补偿. 这是因为输出的和都是恒为1的. 我们通过对公式(78)进行证明可以得到:  
  ![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/24.png?raw=true)   
 结果就是, 如果\\(a_4^L\\)增大, 其他的输出就必须相应的减小, 以使得最终的和是1. 当然, 对于其他的神经元也是一样的.  
 公式(78)还是说明了另外一点, 就是所有输出激励都是非负的. 结合上一段所说的, 我们可以这么说, softmax层的输出就是一个非负数的集合, 然后这个集合内数字的和是1.. 换句话说, softmax层的输出可以看作是一个概率分布.  
 softmax层的输出可以看作是一个概率分布是一件让人开心的事情. 在很多问题上, 将神经网络的输出看作是期望值的概率是很方便的事情. 因此, 在识别手写数字的问题上, 我们可以认为\\(a_j^L\\)是网络对判定真实数字是j的概率.  
 需要说下, 如果输出层是sigmoid layer, 那么我们不能假设输出构成一个概率分布.  
 对softmax 函数和softmax层的工作方式我们有了初步了解. 我们在这里做一个总结, 公式(78)保证了输出激励全为非负, 并且和是1. 因此, softmax函数的形式没那么难理解了. 他保证了输出激励服从一个概率分布. 你可以认为softmax层是对\\(z_j^L\\)的一次再缩放, 然后是缩放后的结果服从概率分布.  
**神经网络学习变慢问题**: 我们前面简单了解了softmax层, 但还没看到softmax层如何解决神经网络学习变慢的问题. 为了说明这个问题, 我们定义一个对数形式的代价函数. 定义\\(x\\)是神经网络的输入, \\(y\\)为对应的输出. 那么对数形式的代价函数如下:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/25.png?raw=true)   
举个例子, 我们使用这个神经网络识别MNIST手写数字, 输入一个图片, 数字7, 那么对应的代价函数为\\(-\ln a_7^L\\). 为了更直观说下, 假设我们的神经网络工作的很好, 那么输出将会给出估计值\\(a_7^L\\)很接近1, 因此, 代价函数\\(-\ln a_7^L\\)会很接近0. 反过来, 如果输出认为不是7, 那么\\(a_7^L\\)很接近0, 代价函数\\(-\ln a_7^L\\)会很大. 因此, 这个对数形式的代价函数正是按照我们预想的工作的.  
说了半天, 神经网络学习变慢的问题呢? 为了分析这个问题, 想一下, 学习速率取决于\(\partial C / \partial w_{jk}^L\\)和\\(\partial C / \partial b_j^L\\), 通过对对数形式的代价函数做计算可得到:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/26.png?raw=true)   
将这些公式跟我们之前讨论使用交叉熵作为代价函数时的推导结果比较下你就能看到, 这两个公式保证了softmax层可以帮我们解决学习速率变慢的问题. 事实上, 将S型神经网络+softmax层, 类比做采用交叉熵函数作为代价函数的S型神经网络是很有用的.  
你可能会迷惑了, 那我应该采用那个呢? S型神经网络+softmax层? S型神经网络+交叉熵代价函数? 事实上, 大部分时候, 这两个工作的都不错. 本章我们主要采用交叉熵代价函数. 在第六章, 我们将介绍卷积神经网络, 那时候我们将会使用softmax. 一种更直观的解释是, 当你的输出希望更像是一个概率分布时, 推荐你使用softmax.  
