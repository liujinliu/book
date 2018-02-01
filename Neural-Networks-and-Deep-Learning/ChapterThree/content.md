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
从图上我们可以看到, 当神经元的输出接近1时, 函数变的很平缓, 也就是说\\({\sigma}^/ (z)\\)很小. 通过公式(55)和(56)可以得到\\(\partial C / \parital w\\)和\\(\partial C / \parital b\\)也会很小. 这正是神经元学习变慢的原因. 接下来我们将看到, 对与一般意义上的神经网络, 这个结论也是存在的.  

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