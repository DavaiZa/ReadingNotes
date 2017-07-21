# 1. 资料导航

## 1.1. MXNet和KVStore

### 1.1.1. 源码编译

详见 [如何从源码编译安装MXNet的python模块?](Build_MXNet_from_source.md)

### 1.1.2. MXNet和KVStore架构理解

* Symbol: 声明式 (Declarative) 的符号表达式 (就是更贴近数学语言的程序语言啦)
* NDArray: 命令式 (Imperative) 的张量计算 (一脸懵逼???什么是张量? 就是有现实意义、有计量单位的数组啦. [链接: 数学不行还学AI-第四话-知乎](https://zhuanlan.zhihu.com/p/25268020)）
* **KVStore: **多设备间的数据交互. 通过push/pull实现交互. **KVStore是参加RDMA2017要关注的模块** .
  * `push`: worker向server推键值对
  * `pull`: server从worker按照键来取值
  * `updater`: push同一个键时所作的操作. 默认为覆盖. 用户可以绑定自己的updater回调函数, 来实现梯度下降\权值更新等目的.

以上概念还需要啃这些论文和网页才能透彻理解:

* [[EB/OL] MXNet设计和实现简介](https://github.com/dmlc/mxnet/issues/797)
* [[J] A Flexible and Efficient Machine Learning Library for Heterogeneous Distributed Systems](https://www.cs.cmu.edu/~muli/file/mxnet-learning-sys.pdf)
* [[EB/OL] MXNet System Architecture](http://mxnet.io/architecture/overview.html)
* [[EB/OL] Deep Learning Programming Style](http://mxnet.io/architecture/program_model.html)

### 1.1.3. pslite架构理解  

角色: server\worker\scheduler

![pslite架构](https://raw.githubusercontent.com/dmlc/dmlc.github.io/master/img/ps-arch.png)

- **Worker**. A worker node performs the main computations such as reading the data and computing the gradient. It communicates with the server nodes via `push` and `pull`. **For example, it pushes the computed gradient to the servers, or pulls the recent model from them.**
- **Server.** A server node **maintains and updates the model weights**. Each node maintains only a part of the model.
- **Scheduler.** The scheduler node monitors the aliveness of other nodes. It can be also used to send control signals to other nodes and collect their progress.

代码结构:

![ps-lite代码结构](http://img.voidcn.com/vcimg/000/006/461/913_c87_27e.svg)

参考文献:

1. [[EB/OL]PS-Lite Documents](http://ps-lite.readthedocs.io/en/latest/). 这个文档介绍了PS-Lite的原理和用法.
2. [[EB/OL]PS-Lite源码分析 - 程序园](http://www.voidcn.com/blog/kangroger/article/p-6643933.html). 这个博文详细分析了PS-Lite的结构, 各个模块(类)的职能.
3. [[EB/OL]PS-Lite源码剖析 - 作业部落](https://www.zybuluo.com/Dounm/note/529299). 这个博文详细剖析了PS-Lite的类职能, 消息处理流程和实现细节. 并给出了简单的例子.
4. [[EB/OL]【深度学习&分布式】Parameter Server详解 - 仙道菜 - CSDN博客](http://blog.csdn.net/cyh_24/article/details/50545780). 这个博文详细解释了Parameter Server的优点(重点在几个动图), 架构(重点在push, pull, range push和range pull), 以及build blocks(由哪些算法和方法组成).
5. [Scaling Distributed Machine Learning with the Parameter Server](https://www.cs.cmu.edu/~muli/file/parameter_server_osdi14.pdf). 注意看Algorithm 1, 里面有ps-lite如何用来做分布式的随机梯度下降.

# 2. 常见问题

## 2.1. MXNet如何整合原生InfiniBand?

**Q:** [How to use native Infiniband instead of IPoIB](https://github.com/dmlc/mxnet/issues/4766)

**A:** MXNet didn't support it. be more accurate, **ps-lite doesn't support native IB**. If you want to contribute, please take a look project dmlc/ps-lite. a good start point will be class Van (reference include/ps/internal/van.h) You can leverage RDMA to do message send/receive.
