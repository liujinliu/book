<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
作者：Michael Nielsen
英文在线：http://neuralnetworksanddeeplearning.com/index.html

# 改进神经网络的学习算法
<!--more-->  
一个高尔夫运动员刚开始学习高尔夫的时候, 大部分时候都在学习基础挥杆动作. 但他们会渐渐学会高级杆法. 同样的, 我们也在接下来会学习如何改进反向传播算法, 从而进一步改进我们在第一章的神经网络的性能.  
本章我们将要讨论的东西包括一个改进的代价函数--"交叉熵代价函数(cross-entropy cost funciton)". 四种"正则化"方法((L1 and L2 regularization, dropout, and artificial expansion of the training data). 这个帮助我们的网络更好的工作. 本章还会介绍如何更好的选取网络权重和偏置的初始值. 并且还会讨论一些方法以更好的选择hyper-parameters. 本章还会简要讨论几种其他技术, 每个讨论都是分开独立的, 读者完全可以略过自己不感兴趣的内容. 我们还会通过代码实现我们讨论的改进方法, 来改进第一章所使用的识别手写数字的神经网络.  
当然, 我们的讨论只涵盖了众多神经网络技术中的很小的部分. 但这一小部分都是很重要的部分. 掌握这些重要的技术可以加深我们对神经网络可能出现的问题的理解, 同时这也将使你准备好在需要的时候采用其他的技术.  
我们大多人是讨厌错误的. 我第一次在观众面前弹钢琴时, 感觉很紧张, 结果表现很差. 我很困惑, 因为我不知道哪里弹错了, 直到有人给我明确的指出来才发现. 但接下来我就很快的改正了, 然后剩下的演出我弹的好了很多. 我想说的是, 当错误可以被明确的指出来时, 这可以更快的帮助我们学习, 这同样使用于训练神经网络.  
理想情况下, 我们希望我们的神经网络可以快速的学习, 但实际中是这样么? 为了回答这个问题, 看下面的这个简单的例子, 这个神经元只有一个输入:  
![这里写图片描述](https://github.com/liujinliu/book/blob/master/Neural-Networks-and-Deep-Learning/ChapterThree/img/1.png?raw=true)  
我们来训练这个神经网络做个比较傻的事情, 就是把输入1变成输出0. 这很简单, 我们即使不借助任何帮助, 手动也能算出合适的权重和偏置. 但是, 让我们上面的这个神经网络是如何做这件事情的吧.  
