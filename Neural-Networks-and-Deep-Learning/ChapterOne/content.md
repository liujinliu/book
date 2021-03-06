﻿<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
作者：Michael Nielsen
英文在线：http://neuralnetworksanddeeplearning.com/index.html

# 使用神经网络识别手写数字
 人类的视觉系统是世界上最美妙的系统之一。考虑下面的手写数字  
 <!--more-->  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/1?raw=true)  
大多数人可以很快的识别出来是504192。这件事情很大程度上迷惑了人类，事实上这件事情远没有看起来这么简单。我们的大脑每个半球都包含一层主要的视觉感知皮层，我们可以把这一层称为V1，这一层包含超过1.4亿的神经元，并且他们彼此之间的连接数超过10亿。不仅如此，整个识别过程，除了V1层的处理外，还会涉及V2,V3,V4和V5，做着一步一步更复杂的图片处理。在上百万年的进化中，人类最终实现了每个人都自带一台超级计算机，并用来理解世界——识别手写数字才得以看起来毫不费力。我们在接受识别视觉信息方面是如此擅长，所有这一切就像是无意识完成的，我们甚至不会因此夸奖我们的视觉系统做了一件多么复杂的事情。
如果我们尝试写个电脑程序来识别上面的手写数字，就会发现视觉识别是一件多么复杂的事情。直觉上我们认为可以通过形状来判断，“9就是上面一个圆，然后右边下来有个钩”，这种表述在算法实现的难度暂且不说，当年尝试把这些规则更加细化，你会发现自己需要处理各种特殊情况，常常会掉入异常的沼泽地，整件事情走向绝望。
神经网络尝试采用一种不同的方式解决这个问题，首先是采用足够多的手写数字作为训练标本.  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/2?raw=true)  
然后开发一个系统，可以从这些标本里得到训练。换句话说，神经系统从这些标本中自动进化出一套识别手写数字到数字的对应规则。并且随着训练标本的增多，神经系统可以被训练的更准确的识别手写数字。这上面只列出来100个标本，我们实际上可以准备更多，上千个，上亿个。
这一章我们将会开发一个电脑程序，实现一个神经网络来识别手写数字。这个程序只有74行，没有使用任何开源框架，但是对手写数字的识别准确率可以达到96%以上。并且在接下来的章节，我们将改进策略，准确率可以达到99%以上。事实上, 好的商业技术已经被广泛被银行应用来进行支票的识别, 邮局则使用这种技术进行地址的识别.
我们聚焦在识别手写数字这个问题是因为这是一个认识神经网络的非常棒的原型问题. 这个问题首先具有一定的复杂性---识别手写数字不是一个简单的事情, 但同时这个问题又不需要很复杂的解决方案或是需要一个强大的计算机. 本书中我们将不断的回顾这个问题, 持续探讨一些更好的方法改进算法. 最后我们会讨论深度神经网络在诸如自然语言处理领域的应用. 
这一章如果只是介绍如何写一份代码来实现对手写数字的识别的话, 那么内容一定短的多. 实际上, 在这个过程中, 我们会不断的介绍一些关于神经网络方面的概念. 包括两个重要的人工神经的建模(感知器和S型神经网络). 并且介绍一种在神经网络中使用的典型的学习算法---随机梯度下降算法. 在本章的末尾, 针对本章提供的理论, 我会尽量做到让读者知其然, 知其所以然.
## 预测器
什么是神经网络? 为了开始介绍这些, 我会首先解释一个人工神经---预测器. 预测器在上世纪50年代和60年代由Frank Rosenblatt提出, 他同时受到了更早时候Warren McCulloch和Walter Pitts的工作的启发. 今天, 更多时候我们使用另外的模型来对神经网络建模---其中最主要的一种被称为S型神经网络. 晚些我们会研究S型神经网络, 但是首先让我们花点时间来了解下预测器, 这同时可以帮助我们认识为什么我们选择S型神经网络. 
预测器是如何工作的呢? 一个预测器接受一些二进制的输入\\(x_1,x_2,......\\), 并且得到一个二进制的输出:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/3.png?raw=true)  
上面这个例子所表示的预测器包含三个输入,\\(x_1,x_2,x_3\\). 一般来说预测器的模型并没有规定输入的个数. Rosenblatt提出了一种简单的规则来计算预测器的输出.他引入了权重的概念,\\(w_1,w_2,w_3\\), 权重表明了对应的输入对输出结果的影响力. 神经元的输出, 0或者1, 取决于加权和\\(\sum_jw_jx_j\\)与门限值的比较结果.门限值同样是神经元的一个参数, 下面的代数表达或许更准确些:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/4.png?raw=true)  
以上就是一个预测器工作的全部内容.
这是一个基本的数学模型. 你可以理解预测器其实就是根据一些证据来作出一个决策. 我来举个例子, 跟现实可能不是很符合, 但却有助于理解这里提到的模型(我们接下来将会讲更现实一点的模型). 假设周末就要来了, 并且你听说你的城市里将会有一个奶酪节, 你喜欢奶酪. 但你是否一定会去还要取决下面的几个条件:

1. 天气好么
2. 你的男(女)朋友是否愿意陪你一起
3. 奶酪节的地点距离地铁近么, 因为你没有车

我们可以采用三个参数\\(x_1,x_2,x_3\\)来表示这三种因子, \\(x_1=1\\)表示天气好, \\(x_1=0\\)表示天气糟糕, \\(x_2=1\\)表示你的女(男)朋友愿意陪你一起, \\(x_2=0\\)表示他不愿意,\\(x_3=1\\)表示那附近有地铁, \\(x_3=0\\)表示天那附近没有地铁.  
现在假设你非常喜欢奶酪, 即使没人陪你或是那附近没有地铁你一样愿意去, 但可能确实很在意天气, 因为如果天气很糟糕, 你真的没有办法过去. 你可以使用预测器来为你的这次决定建模, 权重分别为\\(w_1=6,w_2=2,w_3=2\\), 权重越大表明这个条件所占的比重越大, 然后假设你选择的门限值是5. 你可以看到, 只要天气好, 这个预测器的输出一定是1, 天气不好, 这个预测器的输出一定是0.  
通过修改这个门限, 我们可以得到不同的结果, 比如我们将门限降到3, 那么如果天气好, 或者地点附近有地铁并且你的女(男)朋友愿意陪你, 你都会去. 换句话说, 降低门限之后我们得到了一个新的预测器, 而这个预测器的模型表明了更强烈的去的欲望.  
显然, 预测器模型没有完整的展示我们人类做决定的过程. 但这个例子还是展示了对不同的输入因子施以不同的权重是如何影响最终的输出的. 让人高兴的是, 更复杂的预测器网络可以提供更微妙的决策输出.  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/5.png?raw=true)  
上面所示的网络中, 第一列预测器---我们称其为预测器的第一层---会根据输入生成三个输出(决策). 但是第二层是做什么的呢? 通过对第一层的输出(也即时第二层的输入)的加权运算, 得到一个新的输出(决策). 通过这种方式, 第二层预测器可以在一个更复杂更抽象的层面上工作. 然后, 第三层在此基础上可以作出更复杂的决策. 通过这种方式, 多层预测器可以从事更加复杂精密的预测.  
顺便说一句, 当我定义预测器的时候, 我说了预测器只有一个输出. 在上面这个网络, 看起来是有多个输出的. 事实上, 它依然是一个输出, 多个箭头仅仅是为了表明预测器的输出被用作多个预测器的输入.  
让我们来简化下对预测器的描述. 公式\\(\sum_jw_jx_j > threshold\\)显得有些笨重, 我们可以引入一些符号来简化这个表达式. 首先, 使用点乘, \\(w\cdot x \equiv \sum_jw_jx_j\\), \\(w\\)和\\(x\\)都是向量, 元素是相应的权重和输入. 接下来将门限值移到等式的另一边. 定义 \\(b \equiv -threshold\\)作为预测器的偏置. 于是, 预测器可以被重写为:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/6.png?raw=true)  
你可以认为这个偏置的大小表明了使预测器输出1的难易程度. 或者, 从更生物学的角度做个比较, 越大的偏置, 激起预测器的欲火就更容易. 对一个预测器来说, 给一个足够大的偏置, 则预测器很容易就可以输出1, 相反, 如果偏置很小, 甚至是负数, 那么显然使预测器输出1会难度很大. 引入偏置对于描述预测器来说只是一个很小的改动, 但接下来我们将看到这将给表达式带来极大的简化. 因此, 接下来, 我们不再使用门限值这个说法, 统一用偏置来代替(原文叫bias).  
我之前将预测器描述为通过对输入进行加权运算来得出决策的一种方式. 另外一种描述方式是用预测器来表示逻辑电路, 比如与们, 非门, 与非门. 比如下面这个预测器, 它具有两个输入, 相应的权重都是-2, 全局的偏置是3.  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/7.png?raw=true)  
显然上面这个预测器输入为0,0/0,1/1,0时候输出都是1, 但是输入1,1时候输出为0. 我们用预测器实现了一个与非门.  
上面这个与非门的例子表明了我们可以用预测器实现简单的逻辑功能. 实际上我们可以用预测器实现任意的逻辑功能. 那是因为与非门的组合可以实现任意的逻辑功能. 比如我们可以使用与非门实现两个bit的相加. 这需要实现按位和的功能\\(x_1 \oplus x_2\\), 同时, 为了实现进位计算,我们还需要实现按位乘\\(x_1x_2\\).  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/8.png?raw=true)  
使用两输入的预测器替代与非门, 权重设置为-2, 偏执设置为3, 我们可以得到一个预测器的网络.  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/9.png?raw=true)  
一个需要注意的地方是上面这个网络, 最左边预测器的输出被当作最下面的预测器的输入被使用了两次.当我定义预测器时候我并没有说这种情况是否允许, 事实上这样做是没有问题的. 当然如果你不想看到这样, 我们大可以将那两条线合并成一条, 但这样权重就变为-4, 像下面这样:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/10.png?raw=true)  
实际上, 我们还可以多画一层预测器, 像下面这样  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/11.png?raw=true)  
这种画法, 有输出没输入, 只是我们做的一个简化记号, 并不意味着一个没有输入的预测器.  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/12.png?raw=true)  
想象一下, 如果真的有这样一个预测器, 那么显然\\(\sum_jw_jx_j\\)会衡等于0, 也就是说预测器的输出将完全取决于偏置. 因此输入预测器仅仅提供输入, 这样理解是比较合理的.  
上面的加法器的例子如何使用预测器来仿真一个包含多个与非门的电路. 然而因为与非门是可以应用于通用计算的, 所以预测器也可以用于通用的计算领域.  
这个结论既让人高兴也让人失望. 让人高兴的是预测器可以像其他计算设备一样具有强大的计算能力, 但让人失望的是看起来预测器不过是一个新的类型的与非门, 这可不是什么大新闻.  
然而, 这个实验依然揭示了一点, 就是我们可以在这个神经元基础上开发一种学习算法, 可以自适应的调整自己的权重和偏置. 权重和偏置并非由程序直接生成, 而且根据对外部仿真的响应来调整参数. 通过简单的学习去解决问题, 而不是像传统思想一样事先摆好一堆与非门, 我们的神经网络可以处理很多在传统逻辑上很难解决的问题.  

## S型神经网络
学习算法听起来棒极了. 但是我们应该如何为一个神经网络设计学习算法呢? 假设我们有一个预测器网络, 我们需要让他自学习并解决一些问题. 比如输入是一个手写数字扫描出来的原始图片的像素数据. 我们希望可以让预测器学习到合适的权重和偏置, 这样它的输出可以正确的识别这个手写数字. 为了看清这个学习算法是如何工作的, 假设我们在权重(或者偏置)上做一个很小的改动, 我们希望这个很小的改动同样可以反应在输出上发生了一个很小的改动. 一会儿我们就会发现, 这种特性使得学习成为可能. 并且这正是我们想要的(显然下面这个网络用来识别手写数字有点太天真了).  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/13.png?raw=true)  
如果真的是像我们描述的那样, 一个小的权重或是偏置的更改, 带来一个小的输出的更改, 那么我们就可以利用这一特性一点点达成我们的目标. 假设我们的网络错误的将8认成了9, 我们可以小幅修改权重或是偏置, 使得结果更偏向9一点, 重复这一操作, 网络就一步步得到了学习.  
然而问题是如果网络中存在预测器, 我们上面所说的将很难存在. 事实上, 一个小的权重或是偏置的更改可能完全将预测器的输出反转. 这个反转会使得网络的剩余部分的行为难以控制. 所以当我们好不容易让这个网络识别出9之后, 在其他数字上很可能已经无法控制并且一团糟糕.也许存在某些聪明的办法可以解决这个问题, 但目前为止我们找不到让预测器去学习的办法.  
我们可以解决这个问题---引入一个新的神经网络, Sigmoid型神经网络, 我接下来都称其为S型神经网络. S型神经网络跟预测器有点类似, 但是它可以保证一个小的权重或是偏置的变化仅仅会在输出上引起一个小的变化. 正是这一点使得S型神经网络可以进行学习进化.  
好的, 现在我来描述下S型神经网络, 这初看起来跟我之前描述的预测器是一样的.  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/14.png?raw=true)  
就像一个预测器, S型神经元同样包含一些输入\\(x_1,x_2,x_3......\\), 但不一定是0/1, 这些输入可以在0和1之间任意取值. 同样的, S性神经元同样包含一系列权重\\(w_1,w_2,w_3......\\)和一个全局的偏置\\(b\\). 但输出不是0/1, 而是一个Sigmoid函数所表示的结果\\(\sigma(w \cdot x+b)\\):  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/15.png?raw=true)  
写的更确切一点是像下面这样, 输入\\(x_1,x_2,x_3......\\), 权重\\(w_1,w_2,w_3.....\\)和偏置\\(b\\)的S型神经元的输出:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/16.png?raw=true)  
第一眼看上去, S型神经元与预测器差别很大. Sigmoid函数看起来有点晦涩. 事实上, S型神经元跟预测器有很多类似的地方, 上面的公式体现的不过是一个技术上的细节, 不应该成为我们理解上的障碍.  
为了理解S型神经元与预测器的相似之处, 假设\\(z \equiv w \cdot x+b\\)是一个很大的整数. 那么\\(e^{-z} \approx 0 \\) , \\(\sigma (z) \approx 1\\). 换句话说, \\(z=w \cdot x+b\\) 是一个很大的正数, 这个S型神经元的输出就会非常接近1.  反过来, 如果\\(z=w \cdot x+b\\)是一个很小的负数, 那么\\(e^{-z} \approx \infty\\), \\(\sigma (z) \approx 0\\). 以上的表现都使得S型神经元跟预测器有很多相似之处. 当然在大部分情况下\\(z=w \cdot x+b\\)根据模型的情况会有一个适度的大小, 这使得S型神经元跟预测器有较大的不同.  
也许你会问, \\(\sigma\\)的代数形式是什么样的? 我应该如何理解? 其实, \\(\sigma\\)的确切形式并不重要---真正重要的是当它被画出来时候的样子, 下面就是这个函数的图形:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/17.png?raw=true)  
相比于下面这个阶越函数, 这个显然更平滑一些.  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/18.png?raw=true)  
如果\\(\sigma\\)表现为阶越函数的话, 那么S型神经元就完全退化为预测器, 因为\\(w \cdot x+b\\)是正数还是负数直接决定了输出是1还是0. 事实上, 最终起作用的正是\\(\sigma\\)函数的平滑属性, 而并非它的确切形式. 平滑意味着在权重上和(或)偏置上小的改动\\(\Delta w_j, \Delta b\\)最终在输出上仅仅会带来一个小的改动\\(\Delta output\\). 他们之间近似遵循下面的公式:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/19.png?raw=true)  
上面的公式中, \\(\partial output/ \partial w_j\\) 和\\(\partial output / \partial b\\)表示对\\(w_j\\)和\\(b\\)的偏微分. 当然, 如果你不懂偏微分没关系. 上面的式子其实不过表明了一个很简单的东西. 即\\(\Delta output\\) 跟\\(\Delta w_j, \Delta b\\)是呈线性相关的. 线性特性使得挑选\\(\Delta w_j, \Delta b\\)以得到一个想要的在结果上的变化变得相对容易很多. 在回想下S型神经元与预测器有很多相似之处, 得出如何改变\\(\Delta w_j, \Delta b\\)变的更容易了. 
如果\\(\sigma\\)函数的表现形式不重要, 那我们为什么不使用公式(3)呢? 事实上, 在接下来的章节, 我们在有些时候会使用一些\\(f(w \cdot x)+b\\)其他的激活函数形式. 换其他激活函数影响的主要是公式(5)中偏微分的值. 接下来我们会看到, 当计算偏微分时, 使用\\(\sigma\\)函数将给我们带来简化. 因为指数在微分上有让人喜欢的特性. \\(\sigma\\)函数是一个在神经网络上普遍应用的函数, 并且也是本书大部分时候采用的函数.  
我们应该如何解释S型神经元的输出呢? 显然, 一个很大的不同是S型神经元输出不止0或1, 而是0和1之间的任何数字. 当我们想用输出表示输入到神经元的图片像素的平均密度, 会发现这个很有用. 但有时候这也会带来干扰. 假设我们希望输出告诉我们输入的图片是8还是9, 那么如果输出只有0或1这对我们就很友好了, 就像预测器干的那样. 当然, 针对这一点, 我们在S型神经元上引入一个判断机制会比较好一点, 比如输出大于0.5都表明输入的图片是9, 所有小与等于0.5都表示输入的图片不是9. 当前, 接下来讲的时候使用这个规则我都会提前表述, 相信不会带来误解.  
## 神经网络的架构
接下来这一节我会介绍一种神经网络, 它可以比较高效的识别手写数字. 在开始之前, 我们有必要解释一些术语, 假设我们有以下的神经网络:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/20.png?raw=true)  
最左边叫做输入层, 其中的神经元叫输入神经元, 最右边叫做输出层, 包含一个输出神经元. 中间的这层叫做隐藏层, 因为这一层神经元既不是输人也不是输出. "隐藏"这个词乍一听有点神秘---当我第一次听说的时候我也以为这一定表示什么高深的数学概念, 但相信我, 它仅仅表示这一层既不是输入也不是输出. 上面这个例子只包含一个隐藏层, 实际上有些, 甚至是大多数神经网络都有多层隐藏层, 下面这个四层神经网络就包含两个隐藏层:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/21.png?raw=true)  
因为历史原因, 这样的多层网络有时候也被称为多层预测器或是MLPs(multilayer perceptrons), 因为这个网络是由S型神经元组成的, 而不是预测器, 所以接下来我将会使用MLP这个术语, 因为我觉得这容易有些混淆, 所以在此特别说一下.  
网络的输入和输出设计通常都是很简单的.举个例子, 假设我们要判断是一个手写数字是否是9. 一种比较自然的方式是使用图片的像素点强度编码后的值作为神经元的输入. 如果图片是一个\\(64 \times 64\\)的灰度图, 那么我们将可以得到\\(4096=64 \times 64\\)个神经元, 输入值分布于0~1之间. 输出只包含一个神经元, 大于0.5表示这个图片是9, 小与0.5表示这个图片不是9.  
输入神经元和输出神经元的设计都是很简单直接的, 那么如何设计隐藏层就是个技术活了, 或者说是一件关乎艺术的事情. 特别的, 想使用几个简单的经验法则来设计隐藏层是不可能的. 为了让从神经网络获得想要的输出, 研究人员开发了很多设计方法. 例如有些方法帮助我们确定如何在网络训练时间和网络隐藏层的数量上取得均衡. 在晚些时候我们将看到这种设计方法.  
目前为止, 我们讨论的是前馈网络---前一层的神经元的输出将作为下一层的输入. 这意外着网络中没有环的存在, 信息流是一直向前流动的. 如果我们引入循环, 那么\\(\sigma\\)函数的输入将收到输出的影响, 这让人很难理解, 因此我们目前不允许这样的循环出现.  
然而, 现实中确实是存在包含反馈循环的人工神经网络的. 这种模型被称作[递归神经网络](https://en.wikipedia.org/wiki/Recurrent_neural_network). 关于递归神经网络这里就不展开了, 递归神经网络的影响力不如前馈神经网络, 部分原因是目前为止, 递归神经网络的算法没有前馈神经网络强大. 这里要说的一点是, 递归神经网络依然很有趣, 在模拟我们的大脑工作上, 递归神经网络显然更符合逻辑. 而且有些在前馈网络看来很难解决的问题, 可能是递归神经网络的用武之地. 但是本书将不会展开去讨论递归神经网络, 而主要关注点在应用更广泛的前馈神经网络上.  
## 一个识别手写数字的简单网络
我们将识别手写数字分成两个子问题. 首先我们可以将包含一串数字的图片分成一个一个的包含一个数字的图片. 比如下面的图片可以被分解:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/22.png?raw=true)  
分解之后为6张独立图片:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/23.png?raw=true)  
分成6张独立图片对我们人类来说很简单, 但对机器来说却有不小的挑战. 一旦数字被分解成独立的, 接下来就需要程序识别出单个图片, 比如, 我们需要我们的机器识别出下面这个张图片是一个5  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/24.png?raw=true)  
我们将把主要精力放在第二个问题上, 即识别单个图片. 这是因为, 相信我, 当你可以做到第二点的时候, 第一个不会再有什么难度. 为了识别单个手写数字, 我们使用下面的三层网络:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/25.png?raw=true)  
网络的第一层神经元是对输入像素值的编码. 我们下一节要讨论的, 将使用28*28像素的扫描手写数字图片. 因此输入层包含\\(784=28 \times 28\\)个神经元. 显然我上图没有画这么多输入神经元, 这当然是为了简化, 而且希望你可以谅解. 输入像素是灰度图, 0表示白色, 1表示黑色, 0到1至今的数值表明灰色(从浅到深).  
第二层是隐藏层, 我们用n来表示隐藏层的神经元个数. 我们将尝试使用不同的n, 上面这个例子使用了15.  
网络的输出层采用10个神经元. 如果第一个神经元非常接近1, 那么表示我们这个网络推测出来这个图片代表的是0. 如果第二个神经元输出非常接近1, 那么我们说这个网络推测出来这个数字是1. 更准确一点说, 输出神经元分别代表了输入图片是0~9的数字的概率, 预测结果就是输出最大的那个神经元所代表的数字. 比如输出最大的是第6个神经元, 那么这个网络就是预测输入的图片表示的是6.  
你可能会想, 为什么我们输出采用10个, 因为如果每一个输出是一个二进制数的话, 那么4bit就足够了, 因为\\(2^4 = 16\\), 而我们实际上只想对应10个输出结果. 这样看来, 似乎我们的网络有些效率低了, 但其实这是经验得来的数据, 采用10个输出比采用4个输出效果会更好. 但你可能不满足于知其然, 接下来我将简单解释下.  
为了理解我们这么做的原因, 我们需要从最基础的说起. 首先考虑10个输出的情况. 让我们关注第一个输出神经元, 它用来判断输入的图片是否是0. 它通过权衡隐藏层的结果来作出判断. 但是隐藏层究竟都做了什么呢? 我们假设隐藏层的第一个神经元判断图片是否存在下图所示的内容:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/26.png?raw=true)  
它可以通过在分配加权值时对与图片中阴影重合部分加大加权值, 而对其他非阴影部分采用较小加权值来实现. 同样的, 我们假设隐藏层的第二, 第三, 第四个神经元是为了判断下面的特征是否存在:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/27.png?raw=true)  
你也许已经猜到了, 这四个特征图片一起组成了0:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/28.png?raw=true)  
因此如果这四个特征图片都满足的话, 我们说输入的图片是0. 当然这不是我们判断图片是0的唯一途径, 事实上, 通过对上面的特征进行稍微的改动, 比如方向上的, 或是稍微扭曲下, 我们都可以认为输入的图片是0. 但至少在这种情况下, 我们可以肯定的说输入图片是0.   
假设网络按照我们上面描述的那样工作, 那么我们就可以相对简单的解释为什么选择10个输出神经元而不是4个. 因为如果选择4个输出神经元, 很难想象能有什么算法让输入对应到具体的bit位上.  
当然, 不排除有更聪明的算法使得使用4个输出也得到不错的结果, 但显然我这里描述的网络工作方式简化了很多工作. 这对你相信是有益的.  
## 梯度下降学习算法
现在我们已经有了一个设计好的网络, 我们应该如何训练它来识别手写数字呢? 首先要做的就是选择训练的数据集. 我们将使用[MINIST数据集合](http://yann.lecun.com/exdb/mnist/), 它包含成千上万的手写数字的扫描图像, 并且已经替我们做好了归类. 以下是它的一些图片:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/29.png?raw=true)  
你会发现这就是我们在文章开始处所引用的图片例子. 当然我们训练完这个网络后, 测试时候不会使用我们的训练数据.   
MINIST数据包含两部分, 第一部分包含60000个图像作为训练数据. 这些图像是250个人的手写数字的扫描, 其中一半是美国人口普查局的雇员, 还有一半是高中生. 这些图像都是灰度图, 大小是\\(28 \times 28\\)像素. 第二部分是10000个用来测试的图像. 也是同样大小.为了让测试更有说服力, 测试图片来自不同的250人, 尽管他们也是由美国人口普查局的雇员和高中生组成, 但他们是不同的人.  
我们是用\\(x\\)来表示输入. 每个输入是一个\\(28 \times 28 = 784\\)长度的向量. 每个元素代表了图片上一个灰度值. 我们定义输出为\\(y = y(x)\\), \\(y\\)是一个长度为10的向量. 现在假设有个训练图片是6, 那么我们希望的输出为\\(y(x) = (0,0,0,0,0,0,1,0,0,0)^T\\).我们需要一种算法得到合适的权重和偏置, 使得输入为\\(x\\)时, 输出尽可能接近\\(y(x)\\). 为了衡量这一点, 我们定义下面的成本函数:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/30.png?raw=true)  
这里, \\(w\\)代表网络中所有权重的集合, \\(b\\)代表网络中所有偏置的集合, \\(n\\)代表所有输入的训练的个数, \\(a\\)表示输入为\\(x\\)时对应的输出. \\(||v||\\)表示向量\\(v\\)的长度. 我们称\\(C\\)为平方成本函数, 更普遍的叫法是平均方差函数或是MSE. 显然, 如果\\(y(x)\\)对每一个输入\\(x\\), 都与输出\\(a\\)很接近, 那么\\(C(w,b) \approx 0\\). 所以如果我们算法的目标就是找到合适的\\(w,b\\)使得\\(C(w,b) \approx 0\\), 我们将要采用的算法就是梯度下降算法(gradient descent).  
接下来我将会介绍梯度下降算法, 首先我们把目标换下, 让我们忘记成本函数中的权重, 偏置, 输入的图片像素等等, 我们最小化的目标是一个一般意义上的函数, 它具有多个变量.  
好的, 现在, 假设我们要最小化函数\\(C(v)\\), 这可以是一个多变量的任意实数函数, \\(v=v_1, v_2, v_3.....\\), 为了便于画图理解, 我们假设\\(C\\)只包含两个变量\\(v_1,v_2\\):  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/31.png?raw=true)  
我们一眼就能从图上看到最小值在什么地方, 那是因为我把提供的这个例子太简单了, 事实上\\(C\\)可以很复杂, 可以具有很多变量, 我们是很难一眼看出来的.  
一种方法是微积分, 通过计算每个点的导数来找到最小值的位置, 如果变量多了, 这种算法将是噩梦, 而现实中的神经网络通常具有成千上万的变量, 我们显然无法这样做.  
幸运的是, 有一种算法在这里有比较好的效果. 假设我们的函数图像是一个山谷, 你看下上面的图, 就会发现我这个比喻挺恰当. 然后继续想象有一个球在山坡上滚动. 经验告诉我们这个球将会滚到谷底. 或许我们可以利用这种方式找的函数的最小值. 我们随机的为小球选择一个起点, 然后模拟球向下滚动的动作. 函数\\(C\\)的导数会告诉我们关于这个山谷的形状, 然后根据这个信息我们就可以知道球应该朝哪个方向运动.  
根据我的描述, 也许你会认为我该谈什么牛顿的经典力学公式了. 事实上我们不会纠结于物理公式, 我们目的是找到函数\\(C\\)的最小值, 球的例子只是激发我们的想法. 现在假设我们就是上帝, 我们如何设计一套规则来让小球始终向着山谷的下面运动呢 ?  
现在, 让我们假设我们让球在\\(v_1\\)方向移动了一点点\\(\Delta v_1\\), 在\\(v_2\\)方向移动了一点点\\(\Delta v_2\\), 通过计算我们可以得到\\(C\\)的变化近似满足下面的条件:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/32.png?raw=true)  
我们将试着找到一种方式得到\\(\Delta v_1\\), \\(\Delta v_2\\), 使得\\(\Delta C\\)是负数. 这样就能保证我们的小球是向山谷下方运动的. 我们定义\\(\Delta v\\)表示\\(v\\)的变化, \\(\Delta v \equiv (\Delta v_1, \Delta v_2)^T\\), 我们同时定义\\(C\\)的梯度为\\((\frac{\partial C}{\partial v_1}, \frac{\partial C}{\partial v_2})^T\\). 我们同时定义梯度向量如下:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/33.png?raw=true)  
根据上面的定义, 我们可以得到下面的公式:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/34.png?raw=true)  
这个公式解释了为什么\\(\nabla C\\)叫做梯度向量: \\(\nabla C\\)反映了\\(v\\)的变化如何对应\\(C\\)的变化. 这个公式更令人兴奋的一点是, 它表明了我们应该如何选择\\(\Delta v\\)使得\\(\Delta C\\)是负数.  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/35.png?raw=true)  
\\(\eta\\)是一个比较小的正数(也被称作学习速率). 这样公式(9)告诉我们\\(\Delta C=-\eta {|| \nabla C||}^2\\). 显然, 这可以保证\\(\Delta C \leq 0\\). 因此, 我们使用公式(10)计算\\(\Delta v\\), 然后根据下面的公式移动小球:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/36.png?raw=true)  
我们会一直使用这个公式, 直到得到全局的最小值的点.  
总结一下, 梯度下降算法就是计算梯度\\(\nabla C\\), 然后让小球朝相反的方向运动.像下面这样:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/37.png?raw=true)  
为了使得梯度下降算法正确的工作, 我们需要选择足够小的\\(\eta\\)以使得公式(9)是一个比较好的近似. 否则我们很可能得到\\(\Delta C>0\\), 这显然不是想看到的. 当然, 我们也不希望\\(\eta\\)太小, 这样我们每次移动的就会特别小, 然后整个算法运行起来就会特别慢. 在实际使用中, \\(\eta\\)是经常变化的, 以使得公式(9)是一个比较好的近似, 然后整个算法运行的又不是特别慢.  
我之前举例子使用的是两个变量的函数, 事实上对任意多变量, 我们的算法都可以很好的工作. 假设\\(C\\)包含\\(m\\)个变量, \\(v_1,......v_m\\), 我们选取\\(\Delta v=(\Delta v_1,......,\Delta v_m)^T\\), 那么我们可以得到下面的公式:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/38.png?raw=true)  
梯度向量\\(\nabla C\\)如下:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/39.png?raw=true)  
跟两个参数的做法一样, 我们按下面的公式选择\\(\Delta v\\):  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/40.png?raw=true)  
重复下面的步骤, 我们就可以找到\\(C\\)的最小值的点:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/41.png?raw=true)  
上面的公式给了我们一个迭代得到\\(C\\)的最小值位置的方式, 但上面的公式并不总是能够按照我们预期的那样去工作, 有很多因素会影响到算法的执行效果. 我们接下来的章节会讨论这些问题, 但是在实际中, 梯度下降算法通常都工作的很好, 尤其是在神经网络中, 我们会发现梯度下降算法有强大的能力.  
事实上, 直观上我们也能感觉到梯度下降算法是寻找最小值的最佳策略. 假设我们需要在\\(C\\)下降最快的方向上移动一小步\\(\Delta v\\), 这等同于最小化\\(\Delta C \approx \nabla C \cdot \Delta v\\). 我们限制每一次移动的距离\\(||\Delta v|| = \epsilon\\). 换句话说, 我们每一次移动的是一个固定距离, 然后我们尝试选择一个移动方向使得\\(C\\)减小的最快. 我们可以证明\\(\Delta v=-\eta \nabla C\\), \\(\eta = \epsilon / ||\nabla C||\\)正是我们要找的. 因此, 梯度下降算法可以认为是在\\(C\\)下降最快方向上寻找最小值点的算法. 	
人们研究了很多梯度的计算方法, 比如模拟真是球体滚落的方法, 这种方法有优点也有缺点, 一个主要的缺点是这种方式需要计算函数的二阶导数, 这是非常耗时的. 假设我们需要针对所有变量求解而二阶偏导数\\({\partial}^2 C / \partial v_j \partial v_k \\), 如果参数个数有100万个的话, 那么我们需要最终计算的偏导数将会有一兆. 这是非常耗时的计算, 寻找梯度下降算法的替代算法是一个有趣的领域. 但在本书中, 我们将会使用的还是梯度下降算法.  
如何应用梯度下算法到我们的神经网络中呢? 实际上就是使用梯度下降算法得到权重\\(w_k\\)和偏置\\(b_l\\), 使得代价公式(6)得到最小值. 我们使用权重和偏置替代\\(v_j\\)来重写梯度下降更新的公式. 换句话说, 小球位置的变化的将会取决于\\(\partial C/ \partial w_k\\)和\\(\partial C/ \partial b_l\\):  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/42.png?raw=true)  
梯度下降算法实际中应用还是会遇到一些挑战, 我将会在接下来的章节讨论这些. 现在, 我只想强调一个问题. 回看公式(6), 你会发现代价函数的形式是\\(\nabla C = \frac {1}{n} {\sum}_x \nabla C_x\\), 这其实是在所有输入上取平均的一个结果, 即\\(C_x \equiv \frac {{||y(x)-a||}^2}{2}\\). 实际中应用起来, 我们需要针对每个输入\\(x\\), 独立计算他的梯度向量\\(\nabla C_x\\), 然后计算再取平均. \\(\nabla C = \frac {1}{n} \sum_x \nabla C_x\\). 不幸的是, 当输入的训练样本特别多的时候, 这是非常耗时的操作, 将会严重拖累算法的整体运行速度.  
一种叫做随机梯度下降算法可以用来加速神经网络学习的过程. 它根据对输入的数据进行采样(有限多个)进行计算得到一个估计的\\(\nabla C\\), 通过在这一有限集合上来计算平均值, 我们可以得到真实\\(\nabla C\\)的一个比较近似的结果. 这可以帮助我们加快学习算法运行的速度.  
我们再更详细的描述下这个算法, 随机梯度下降学习算法通过随机的选择一小部分训练输入, \\(X_1, X_2,..... X_m\\), 我们称之为mini-batch(原谅我实在不知道应该怎么翻译这个名词, 但觉得英文意思表达的很准确). \\(m\\)足够大, 我们希望在\\(/nabla C_j\\)上取平均可以尽量的接近\\(\nabla C\\).  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/43.png?raw=true)  
中间的部分表示在所有的训练数据上取平均. 根据上面的公式我们可以得到下面的结果:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/44.png?raw=true)  
应用随机梯度下降算法到我们的神经网络上, 使用\\(w_k, b_l\\)分别表示神经网络的权重和偏置, 随机梯度下降算法通过在一批随机选择的输入集上进行训练, 我们可以得到下面的公式:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/45.png?raw=true)  
我们反复的随机挑取输入执行上的迭代过程, 知道所有的输入都使用过. 我们称之为完成了神经网络的的阶段训练. 然后我们开始下一阶段的训练.  
顺带提一下关于代价函数缩放的问题, 在公式(6)中, 我们使用了\\(\frac {1}{n}\\)对代价函数进行了缩放. 人们有时候会忽略掉这个参数. 也就是说在所有输入得到的代价上取和, 而不是取平均. 当训练输入的个数无法确定的时候, 这种做法是合理的. 比如, 在训练的同时有更多的训练数据生成. 同样的, 公式(20)和(21)有时候也会忽略掉\\(\frac {1}{m}\\). 一般来说这两种做法差异不大, 因为这等价于我们对学习速率\\(\eta \\)进行缩放. 但有时候也得具体问题具体分析.  
我们可以将随机梯度下降算法想象成是政治上的投票. 显然在随机选择的一部分人上得到选票统计结果比让每个人都投票要容易操作的多. 假设我们的训练集合的大小\\(m=60000\\), 然后选择每一批的大小\\(m=10\\). 这意味着相比于在60000个数据上计算梯度, 我们将速度提升了6000倍!! 当然, 这样得到的梯度并不是最佳的, 但没关系, 我们不需要最佳的, 我们只需要这个结果给我指明的是代价函数下降的方向就可以了. 在实际当中, 随机梯度下降算法得到了很广泛的应用.  
关于理论部分的介绍第一章就到这里了. 我来总结一下, 对于刚接触梯度下降算法的同学来说, 如果他们没有接触过矩阵论或是线性代数的话. 在神经网络中, 代价函数\\(C\\), 是一个由很多很多参数组成的函数, 这是一个多维空间的函数, 很多很多维. 有些人可能会想了, 我需要想象出来多维空间. 然后他们就会发愁, "我连四维空间都想象不出来, 更别说五维空间, 甚至是五百万维空间了....., 我是不是缺少什么能力, 那些超级数学家才有的能力??", 当然不是, 即使是超级数学家, 也很少有人能够想象出四维空间. 我们这里采用的诀窍是使用其他的方式来描述我们的问题, 我们使用了一个代数(而不是图像)\\(\Delta C\\)来表示\\(C\\)下降方向的移动. 擅长思考多维的空间人有很多办法, 我们的代数方式是其中一种. 掌握这个方法, 你就可以很轻松的想象多维空间上的问题. 当然还有其他方式思考多维空间问题. 如果你感兴趣的话, 可以参考[这里](https://mathoverflow.net/questions/25983/intuitive-crutches-for-higher-dimensional-thinking). 我就不展开说了.  

## 应用我们的神经网络识别手写数字
我们将通过代码来更细致直观的描述这个神经网络的工作. 代码由python写成, 很短. 你可以通过下面的方法得到, 或是从[这里](https://codeload.github.com/mnielsen/neural-networks-and-deep-learning/zip/master)下载.  
```
git clone https://github.com/mnielsen/neural-networks-and-deep-learning.git
```  
顺便说一下，当我早些时候描述MNIST数据的时候，我说它被分成了60000个训练图像和10000个测试图像。这是官方对MNIST描述。不过我们的做法稍有不同，我们将保留测试图像，但将60,000个训练图像分成两部分：一组50,000个图像用来训练我们的神经网络，以及一个单独的10,000个图像的验证集。我们不会在本章中使用验证数据，但是后面的内容中，我们会讨论如何设置神经网络的某些参数，比如学习率(这些参数不是我们的网络来直接选择的)等等。从现在开始当我提到“MNIST训练数据”时，指的是我们的50,000个图像数据集，而不是原始的60,000个图像数据集.  
你还需要安装[NumPy](http://www.numpy.org/), 这在执行一些运算是可以大大简化我们的程序编写.  
首先看下我们需要使用的核心类, 这个类被用来表示一个神经网络:  
```
class Network(object):

    def __init__(self, sizes):
        self.num_layers = len(sizes)
        self.sizes = sizes
        self.biases = [np.random.randn(y, 1) for y in sizes[1:]]
        self.weights = [np.random.randn(y, x) 
                        for x, y in zip(sizes[:-1], sizes[1:])]
```  
入参```sizes```是一个list, 每个位置的元素表示对应层的神经元的个数. 下面的代码表示我们初始了一个网络, 它的第1, 2, 3层神经元的个数分别是2, 3, 1.  
```
net = Network([2, 3, 1])
```  
Network对象中的偏置和权重都是随机初始化的，随机数来自于使用numpy的```np.random.randn```函数生成均值为0, 标准差为1的高斯分布。这种随机初始化给我们的随机梯度下降算法提供了一个起点. 在后面的章节中，我们会找到更好的方式来初始化权重和偏置，但是现在我们先这样做. 需要注意的是，网络初始化代码假定第一层神经元是输入层，并且没有为这些神经元设置任何偏置，因为偏置仅在计算后面层的输出的时候使用.  
偏置和权重被存储为Numpy的矩阵列表。 例如```net.weights [1]```是一个Numpy矩阵，存储连接第二层到第三层神经元连接的权重(Python的列表索引从0开始). 我们用矩阵\\(w\\)来表示```net.weights [1]```.  \\(w_jk\\)表示第二层中的第\\(k\\)个神经元和第三层中的第\\(j\\)个神经元之间的连接的权重. \\(j\\)和\\(k\\)的这种排序可能看起来很奇怪, 交换\\(j\\)和\\(k\\)指数会更有意义吗? 使用这种顺序的一大优点是，这使得第三层神经元的激活向量可以做如下表示:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/46.png?raw=true)  
公式中\\(a\\)表示第二层神经元的激活向量, 我们将\\(a\\)与权重矩阵\\(w\\)相乘, 然后加上偏置向量\\(b\\), 然后我们对得到的向量的每个元素应用\\(\sigma\\)函数, 得到\\(a^{/}\\). 很容易证明, 公式(22)得到的结果跟公式(4)在计算S型神经元的输出上得到的结果是一致的.  
有了以上做铺垫, 写出计算神经网络输出的代码就很显然了, 我们先定义sigmoid函数:  
```
def sigmoid(z):
    return 1.0/(1.0+np.exp(-z))
```  
当输入是向量是, Numpy会自动的对每一个元素调用sigmoid函数, 得到一个向量.  
接下来我们在类```Network```中加入一个前馈函数, 当给定一个网络的输入时, 可以得到对应的输出. 这个函数就是针对网络的每一层执行公式(22):  
```
    def feedforward(self, a):
        """Return the output of the network if "a" is input."""
        for b, w in zip(self.biases, self.weights):
            a = sigmoid(np.dot(w, a)+b)
        return a
```  
下面这个函数实现了随机梯度下降算法, 这个算法不是一眼能看懂, 我接下来会解释:  

```
def SGD(self, training_data, epochs, mini_batch_size, eta,
            test_data=None):
        """Train the neural network using mini-batch stochastic
        gradient descent.  The "training_data" is a list of tuples
        "(x, y)" representing the training inputs and the desired
        outputs.  The other non-optional parameters are
        self-explanatory.  If "test_data" is provided then the
        network will be evaluated against the test data after each
        epoch, and partial progress printed out.  This is useful for
        tracking progress, but slows things down substantially."""
        if test_data: n_test = len(test_data)
        n = len(training_data)
        for j in xrange(epochs):
            random.shuffle(training_data)
            mini_batches = [
                training_data[k:k+mini_batch_size]
                for k in xrange(0, n, mini_batch_size)]
            for mini_batch in mini_batches:
                self.update_mini_batch(mini_batch, eta)
            if test_data:
                print "Epoch {0}: {1} / {2}".format(
                    j, self.evaluate(test_data), n_test)
            else:
                print "Epoch {0} complete".format(j)
```  
```training_data```是一个元组的list, 每个元组\\((x,y)\\)对应训练输入和相应的期望得到的输出. ```epochs```表示训练迭代的次数, ```mini_batch_size```表示每次训练所取的一批包含输入的个数. ```eta```表示学习速率\\(\eta\\). 还有一个入参```test_data```, 如果提供了这个入参, 那么在每次迭代训练之后都会使用这个参数来验证学习效果, 这对我们跟踪学习过程很有帮助, 但会减慢学习速度, 因为干的事情多了.  
代码的工作方式如下, 在每次迭代开始前, 首先随机打乱训练数据, 然后根据入参```mini_batch_size```把数据分成一小批一小批的. 然后针对每一批, 执行一次梯度下降算法. 函数```self.update_mini_batch(mini_batch, eta)```就是干这个的.  
```
def update_mini_batch(self, mini_batch, eta):
        """Update the network's weights and biases by applying
        gradient descent using backpropagation to a single mini batch.
        The "mini_batch" is a list of tuples "(x, y)", and "eta"
        is the learning rate."""
        nabla_b = [np.zeros(b.shape) for b in self.biases]
        nabla_w = [np.zeros(w.shape) for w in self.weights]
        for x, y in mini_batch:
            delta_nabla_b, delta_nabla_w = self.backprop(x, y)
            nabla_b = [nb+dnb for nb, dnb in zip(nabla_b, delta_nabla_b)]
            nabla_w = [nw+dnw for nw, dnw in zip(nabla_w, delta_nabla_w)]
        self.weights = [w-(eta/len(mini_batch))*nw 
                        for w, nw in zip(self.weights, nabla_w)]
        self.biases = [b-(eta/len(mini_batch))*nb 
                       for b, nb in zip(self.biases, nabla_b)]
```  
主要工作发生在下面这个函数中:  
```
 delta_nabla_b, delta_nabla_w = self.backprop(x, y)
```  
这个函数涉及到了反向传播算法, 这是计算代价函数梯度的快速方法. 对于反向传播算法, 我们在下一章会重点说, 这里我们只需要记住, 这个算法对于给定的训练输入, 可以快速计算出梯度.  
下面是所有代码放在一起, 你可以看到, 真的很短! 代码还可以从[这里](https://github.com/mnielsen/neural-networks-and-deep-learning/blob/master/src/network.py)下载到  
```
"""
network.py
~~~~~~~~~~

A module to implement the stochastic gradient descent learning
algorithm for a feedforward neural network.  Gradients are calculated
using backpropagation.  Note that I have focused on making the code
simple, easily readable, and easily modifiable.  It is not optimized,
and omits many desirable features.
"""

#### Libraries
# Standard library
import random

# Third-party libraries
import numpy as np

class Network(object):

    def __init__(self, sizes):
        """The list ``sizes`` contains the number of neurons in the
        respective layers of the network.  For example, if the list
        was [2, 3, 1] then it would be a three-layer network, with the
        first layer containing 2 neurons, the second layer 3 neurons,
        and the third layer 1 neuron.  The biases and weights for the
        network are initialized randomly, using a Gaussian
        distribution with mean 0, and variance 1.  Note that the first
        layer is assumed to be an input layer, and by convention we
        won't set any biases for those neurons, since biases are only
        ever used in computing the outputs from later layers."""
        self.num_layers = len(sizes)
        self.sizes = sizes
        self.biases = [np.random.randn(y, 1) for y in sizes[1:]]
        self.weights = [np.random.randn(y, x)
                        for x, y in zip(sizes[:-1], sizes[1:])]

    def feedforward(self, a):
        """Return the output of the network if ``a`` is input."""
        for b, w in zip(self.biases, self.weights):
            a = sigmoid(np.dot(w, a)+b)
        return a

    def SGD(self, training_data, epochs, mini_batch_size, eta,
            test_data=None):
        """Train the neural network using mini-batch stochastic
        gradient descent.  The ``training_data`` is a list of tuples
        ``(x, y)`` representing the training inputs and the desired
        outputs.  The other non-optional parameters are
        self-explanatory.  If ``test_data`` is provided then the
        network will be evaluated against the test data after each
        epoch, and partial progress printed out.  This is useful for
        tracking progress, but slows things down substantially."""
        if test_data: n_test = len(test_data)
        n = len(training_data)
        for j in xrange(epochs):
            random.shuffle(training_data)
            mini_batches = [
                training_data[k:k+mini_batch_size]
                for k in xrange(0, n, mini_batch_size)]
            for mini_batch in mini_batches:
                self.update_mini_batch(mini_batch, eta)
            if test_data:
                print "Epoch {0}: {1} / {2}".format(
                    j, self.evaluate(test_data), n_test)
            else:
                print "Epoch {0} complete".format(j)

    def update_mini_batch(self, mini_batch, eta):
        """Update the network's weights and biases by applying
        gradient descent using backpropagation to a single mini batch.
        The ``mini_batch`` is a list of tuples ``(x, y)``, and ``eta``
        is the learning rate."""
        nabla_b = [np.zeros(b.shape) for b in self.biases]
        nabla_w = [np.zeros(w.shape) for w in self.weights]
        for x, y in mini_batch:
            delta_nabla_b, delta_nabla_w = self.backprop(x, y)
            nabla_b = [nb+dnb for nb, dnb in zip(nabla_b, delta_nabla_b)]
            nabla_w = [nw+dnw for nw, dnw in zip(nabla_w, delta_nabla_w)]
        self.weights = [w-(eta/len(mini_batch))*nw
                        for w, nw in zip(self.weights, nabla_w)]
        self.biases = [b-(eta/len(mini_batch))*nb
                       for b, nb in zip(self.biases, nabla_b)]

    def backprop(self, x, y):
        """Return a tuple ``(nabla_b, nabla_w)`` representing the
        gradient for the cost function C_x.  ``nabla_b`` and
        ``nabla_w`` are layer-by-layer lists of numpy arrays, similar
        to ``self.biases`` and ``self.weights``."""
        nabla_b = [np.zeros(b.shape) for b in self.biases]
        nabla_w = [np.zeros(w.shape) for w in self.weights]
        # feedforward
        activation = x
        activations = [x] # list to store all the activations, layer by layer
        zs = [] # list to store all the z vectors, layer by layer
        for b, w in zip(self.biases, self.weights):
            z = np.dot(w, activation)+b
            zs.append(z)
            activation = sigmoid(z)
            activations.append(activation)
        # backward pass
        delta = self.cost_derivative(activations[-1], y) * \
            sigmoid_prime(zs[-1])
        nabla_b[-1] = delta
        nabla_w[-1] = np.dot(delta, activations[-2].transpose())
        # Note that the variable l in the loop below is used a little
        # differently to the notation in Chapter 2 of the book.  Here,
        # l = 1 means the last layer of neurons, l = 2 is the
        # second-last layer, and so on.  It's a renumbering of the
        # scheme in the book, used here to take advantage of the fact
        # that Python can use negative indices in lists.
        for l in xrange(2, self.num_layers):
            z = zs[-l]
            sp = sigmoid_prime(z)
            delta = np.dot(self.weights[-l+1].transpose(), delta) * sp
            nabla_b[-l] = delta
            nabla_w[-l] = np.dot(delta, activations[-l-1].transpose())
        return (nabla_b, nabla_w)

    def evaluate(self, test_data):
        """Return the number of test inputs for which the neural
        network outputs the correct result. Note that the neural
        network's output is assumed to be the index of whichever
        neuron in the final layer has the highest activation."""
        test_results = [(np.argmax(self.feedforward(x)), y)
                        for (x, y) in test_data]
        return sum(int(x == y) for (x, y) in test_results)

    def cost_derivative(self, output_activations, y):
        """Return the vector of partial derivatives \partial C_x /
        \partial a for the output activations."""
        return (output_activations-y)

#### Miscellaneous functions
def sigmoid(z):
    """The sigmoid function."""
    return 1.0/(1.0+np.exp(-z))

def sigmoid_prime(z):
    """Derivative of the sigmoid function."""
    return sigmoid(z)*(1-sigmoid(z))
```  
我们训练好的神经网络效果怎么样呢? 我们先从下载MINIST数据开始, 我们通过一个脚本```mnist_loader.py```来做这件事:  
```
>>> import mnist_loader
>>> training_data, validation_data, test_data = \
... mnist_loader.load_data_wrapper()
```  
接下来建立一个包含30个隐藏神经元的网络:  
```
>>> import network
>>> net = network.Network([784, 30, 10])
```  
接下来从MINIST的训练集合中选出30批数据, 每批数据大小是10, 学习速率\\(\eta = 3.0\\).  
```
>>> net.SGD(training_data, 30, 10, 3.0, test_data=test_data)
```  
需要说的一点是如果你按照上面的步骤执行下来的话, 运行代码会花费一些时间, 大概几分钟. 如果你着急的话, 可以减少训练数据的数量, 或是减少神经元的数量. 实际生产中, 神经网络训练起来很快, 我们这个代码只是为了让你理解神经网络学习的过程, 而不是构建一个高速的算法和代码. 当然, 一旦我们训练好了之后, 神经网络工作起来会很快. 假设我们已经训练好了神经网络, 即得到了合适的权重和偏置. 那么我们就可以将这个网络移植到浏览器上使用JavaScript来运行, 或是集成到手机app上. 下面展示了在训练过程中的输出, 输出显示了在每一批训练过后, 从10000个测试数据里成功识别的个数. 可以看到的, 仅仅是训练一批之后, 就从10000张图片里准确识别了9129张. 随着训练的进行, 识别成功率还在不断增加.  
```
Epoch 0: 9129 / 10000
Epoch 1: 9295 / 10000
Epoch 2: 9348 / 10000
...
Epoch 27: 9528 / 10000
Epoch 28: 9542 / 10000
Epoch 29: 9534 / 10000
```  
初步来看, 我们的训练的神经网络识别率达到了95%以上, 这对于初次尝试, 还是很让人兴奋的. 需要说的是, 因为我们选择初始化权重和偏置是随机的, 因此你跑这个程序的时候, 得到的结果肯定和我不太一样. 为了得到这个结果, 我跑了三次, 取了效果最好的一次.  
让我们再次回到实验中来, 这一次我们将隐藏层神经元的数量提高到100, 这次实验将会耗费比较多时间, 在作者电脑上, 每一批训练大概要几十秒. 所以你最好在程序运行时继续阅读这篇文章. 不要干等着.  
```
>>> net = network.Network([784, 100, 10])
>>> net.SGD(training_data, 30, 10, 3.0, test_data=test_data)
```  
你也许猜到了, 这次效果要比之前好, 准确率提到了96%以上. 这说明, 至少在我们这个场景下, 多些隐藏神经元是有益的.  
当然, 为了得到上述精度, 我挑选了训练的批数, 每一批数据的大小, 学习速率\\(\eta\\). 这些我称之为hyper-parameters, 以区别于权重和偏置(我们需要神经网络学习的内容). 如果hyper-parameters我们取的不好, 我们将得到让人失望的结果. 比如说, 我们取了\\(\eta = 0.001\\),  
```
>>> net = network.Network([784, 100, 10])
>>> net.SGD(training_data, 30, 10, 0.001, test_data=test_data)
```  
程序运行起来大概像下面这样.  
```
Epoch 0: 1139 / 10000
Epoch 1: 1136 / 10000
Epoch 2: 1135 / 10000
...
Epoch 27: 2101 / 10000
Epoch 28: 2123 / 10000
Epoch 29: 2142 / 10000
```  
你可以看到, 结果在一点点改善, 但速度很慢. 这说明我们应该提高学习速率\\(\eta\\), 然后我们会得到好一点的效果, 我们继续这个操作, 最好我们大概会选择\\(\eta = 1\\), 也可能到\\(\eta = 3\\), 这样我们距离一开始的结果就比较接近了. 所以, 即使我们一开始选择了一个不好的hyper-parameter, 我们依然可以有足够的信息去改善选择.  
通常意义上讲, 训练一个神经网络有时候是很困难的, 尤其是当我们发现选择的hyper-parameters使得网络的训练过程就像是在随机胡猜. 假设我们选择\\(\eta = 100\\):  
```
>>> net = network.Network([784, 30, 10])
>>> net.SGD(training_data, 30, 10, 100.0, test_data=test_data)
```  
这样我们得到的过程大概像下面这样  
```
Epoch 0: 1009 / 10000
Epoch 1: 1009 / 10000
Epoch 2: 1009 / 10000
Epoch 3: 1009 / 10000
...
Epoch 27: 982 / 10000
Epoch 28: 982 / 10000
Epoch 29: 982 / 10000
```  
现在假设我们上来就遇到了上面的问题. 当然, 根据之前的经验, 我们知道需要减小\\(\eta\\). 但如果是头一次遇到这个问题, 从输出里边我们很难看到有用的信息告诉我们该做什么. 我们当然会怀疑我们的学习速率, 我们甚至会怀疑我们这个神经网络的方方面面. 我们会怀疑是不是初始化选择的权重和偏置太离谱了?或者我们没有足够的数据进行训练?或者我们选择的批数太少了? 或者我们的网络模型本身就不适合用来识别手写数字? 学习速率太低了? 太高了?......当你第一次遇见这种问题, 你总是无法肯定该做什么.  
可以看到, 调试神经网络像是一门艺术. 我们需要一些直觉, 一些技巧. 我们将在接下来的章节来讨论如何选择合适的hyper-parameters.  
如果你感兴趣的话, 下面是我用来现在MINIST数据的代码, 注释中说明了存储的格式.  
```
"""
mnist_loader
~~~~~~~~~~~~

A library to load the MNIST image data.  For details of the data
structures that are returned, see the doc strings for ``load_data``
and ``load_data_wrapper``.  In practice, ``load_data_wrapper`` is the
function usually called by our neural network code.
"""

#### Libraries
# Standard library
import cPickle
import gzip

# Third-party libraries
import numpy as np

def load_data():
    """Return the MNIST data as a tuple containing the training data,
    the validation data, and the test data.

    The ``training_data`` is returned as a tuple with two entries.
    The first entry contains the actual training images.  This is a
    numpy ndarray with 50,000 entries.  Each entry is, in turn, a
    numpy ndarray with 784 values, representing the 28 * 28 = 784
    pixels in a single MNIST image.

    The second entry in the ``training_data`` tuple is a numpy ndarray
    containing 50,000 entries.  Those entries are just the digit
    values (0...9) for the corresponding images contained in the first
    entry of the tuple.

    The ``validation_data`` and ``test_data`` are similar, except
    each contains only 10,000 images.

    This is a nice data format, but for use in neural networks it's
    helpful to modify the format of the ``training_data`` a little.
    That's done in the wrapper function ``load_data_wrapper()``, see
    below.
    """
    f = gzip.open('../data/mnist.pkl.gz', 'rb')
    training_data, validation_data, test_data = cPickle.load(f)
    f.close()
    return (training_data, validation_data, test_data)

def load_data_wrapper():
    """Return a tuple containing ``(training_data, validation_data,
    test_data)``. Based on ``load_data``, but the format is more
    convenient for use in our implementation of neural networks.

    In particular, ``training_data`` is a list containing 50,000
    2-tuples ``(x, y)``.  ``x`` is a 784-dimensional numpy.ndarray
    containing the input image.  ``y`` is a 10-dimensional
    numpy.ndarray representing the unit vector corresponding to the
    correct digit for ``x``.

    ``validation_data`` and ``test_data`` are lists containing 10,000
    2-tuples ``(x, y)``.  In each case, ``x`` is a 784-dimensional
    numpy.ndarry containing the input image, and ``y`` is the
    corresponding classification, i.e., the digit values (integers)
    corresponding to ``x``.

    Obviously, this means we're using slightly different formats for
    the training data and the validation / test data.  These formats
    turn out to be the most convenient for use in our neural network
    code."""
    tr_d, va_d, te_d = load_data()
    training_inputs = [np.reshape(x, (784, 1)) for x in tr_d[0]]
    training_results = [vectorized_result(y) for y in tr_d[1]]
    training_data = zip(training_inputs, training_results)
    validation_inputs = [np.reshape(x, (784, 1)) for x in va_d[0]]
    validation_data = zip(validation_inputs, va_d[1])
    test_inputs = [np.reshape(x, (784, 1)) for x in te_d[0]]
    test_data = zip(test_inputs, te_d[1])
    return (training_data, validation_data, test_data)

def vectorized_result(j):
    """Return a 10-dimensional unit vector with a 1.0 in the jth
    position and zeroes elsewhere.  This is used to convert a digit
    (0...9) into a corresponding desired output from the neural
    network."""
    e = np.zeros((10, 1))
    e[j] = 1.0
    return e
```  
## 迈向深度学习
我们的神经网络给出让人印象深刻的表现, 但过程是有些神秘的. 我们初始化时候随机选择了权重和偏置, 然后合适的权重和偏置在学习的过程中由网络自己来改进. 我们没法立刻解释清楚神经网络是如何做到的. 我们是否可以找到一种方法来理解我们的神经网络是如何识别手写数字的? 有了这样的方法, 我们是否可以做的更好?  
让我说的更明显一点, 假设神经网络的研究最终带来了人工智能. 我们能理解人工智能网络是如何工作的么? 网络对我们来说也许是不透明的, 我们不理解的权重和偏置, 他们自动学习出来了. 在人工智能早期, 人们希望建立人工智能的努力可以帮助我们理解人类大脑思考背后的原理. 但我得说, 也许最终我们既无法理解人脑是如何工作, 我们也无法理解人工智能是如何工作的.  
为了解答这些问题, 我们回想下我在本章开始阶段对人工神经元的解释---一种根据输入衡量输出的手段. 假设我们接下来要从下面识别出人脸来.  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/47.png?raw=true)  
就像我们处理之前的识别手写数字的图片一样, 我们依然可以采用像素点的强度作为输入. 然后输出是一个判断, "是的, 这是人脸"或者"不是, 这不是人脸".  
现在我们先来手动设计下这个网络. 忘掉我们之前的神经网络那些东西, 我们分解问题. 我们可以将识别人脸分解成下面的一些子问题: 在图片左上有个眼睛么?在图片右上有个眼睛么?在图片中间部分有个鼻子么? 在图片下部中间位置有个嘴么?有头发么?等等等等.  
如果大部分的子问题都是"YES", 或者是比较大的概率是"YES", 那么我们可以认为这是人脸图片. 相反, 如果大部分的子问题都无法得到肯定的回答, 那么这个图片应该不是一个人脸.  
当然, 这种设计太粗糙了, 比如, 图片上的人可能没有头发, 或者我们可能只看到了部分的脸, 有可能是一个天使的图片. 不过这启发我们如果可以采用神经网络解决一个个子问题, 然后我们就可以再组合成一个网络来进行人脸的判断. 这是一种的设计方法. 当然, 实际中不一定是这么干的, 但这依然给了我们一个如何设计网络架构的启发, 下面是我们设想的框架:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/48.png?raw=true)  
值得高兴的是, 这些子问题依然可以进一步分解. 假设我们这在思考这个问题: "左上角有眼睛么?", 这个问题可以被进一步分解为: "有眉毛么?"; "有睫毛么?"; "有虹膜么?"; 等等等等. 当然问题是应该包含位置信息--"眉毛是在左上角, 在虹膜上面么?". 让我们还是将这个问题简化下吧, 我们将通过下面的方法分解这个"左上角有眼睛么?"这个子问题:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterOne/img/49.png?raw=true)  
所有这些子问题都可以进一步分解, 最终我们的每个子网络都将只需要回答很简单的问题. 这些问题可能只需要回答在图片的特定位置是否有或者没有某个特定形状.  
最终的结果我们的神经网络，将一个非常复杂的问题 - 这个图像是否显示出一张脸,  分解成单个像素级别的非常简单的问题。 它通过多层次的判断来完成，前面的神经层回答关于输入图像的非常简单和具体的问题，后面的层构建了更复杂和抽象概念的层次结构。 具有这种多层结构的网络（两个或多个隐藏层）被称为深度神经网络。