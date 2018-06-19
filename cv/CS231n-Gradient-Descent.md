# 梯度是什么?

这里的梯度, 特指$$f: \mathbb{R}^n\mapsto \mathbb{R}$$的梯度.

简单地说, 梯度就是函数$$f$$在点$$P$$ ($$P\in\mathbb{R}^n$$) 处的各个维度的偏导数组成的向量[^1].

以二元函数$$f: \mathbb{R}^2\mapsto\mathbb{R}, f(x_1, x_2)=x_1^2\sin{x_2}$$为例.

$$f$$在各个维度的偏导数(partial derivative)为

* $$\dfrac{\partial{f}}{\partial{x_1}}=2x_1\sin{x_2}$$;
* $$\dfrac{\partial{f}}{\partial{x_2}}=x_1^2\cos{x_2}$$.

所谓梯度($$\nabla{f}$$), 就是这些偏导数组成的向量:

$$
\nabla{f}=\left[\dfrac{\partial{f}}{\partial{x_1}}, \dfrac{\partial{f}}{\partial{x_2}}\right]^T
$$

# 数值法求梯度

用偏导数的定义来求梯度向量的各个维度[^2].

设$$x_0=[x_1, x_2,..., x_i, ..., x_n]^T, x_1=[x_1, x_2, ..., x_i+h, ..., x_n]^T$$.

函数$$f(x)$$在输入维度$$x_i$$上的偏导数定义为: 
$$
\dfrac{\partial{f}}{\partial{x_i}}=\lim_{h\to0}{\dfrac{f(x_1)-f(x_0)}{h}}
$$

## a. 朴素实现

一种朴素的实现如下:

```python
def f_callback(x, param):
    """
    待优化的函数原型
    x - 一个输入点
    param - 函数的参数
    y - 返回一个实数
    """
    return y

def gradient(f, x0, param, h=1e-5):
    """
    一种朴素的求函数f在点x0处的梯度的算法
    f  - 函数
    x0 - 点
    h  - 精度
    """
    
    fx0  = f(x0, param)
    x1   = np.copy(x0)
    grad = np.zeros(x0.shape)
    for i in range(len(x0)):
        x1_i_old = x1[i]
        x1[i] = x1_i_old + h # 计算x_1
        fx1 = f(x1, param) # 计算f(x_1)
        grad[i] = (fx1-fx0)/h # 计算梯度
        x1[i] = x1_i_old # 还原x_1
    return grad
```

## b. 改进

用式2的等价形式
$$
\dfrac{\partial{f}}{\partial{x_i}}=\lim_{h\to0}{\dfrac{f(x_1)-f(x_2)}{2h}}
$$
其中$$x_2=[x_1, x_2, ..., x_i-h, ..., x_n]^T$$.

这个方法有着更好的数值精度.

# 梯度下降

## a. 梯度下降

Gradient Descent

直接计算整个损失函数, 需要用到整个训练集.

缺点: 

* 过拟合
* 慢

```python
while True:
    grad = gradient(loss_f, weights, training_set)
    delta  = -learning_rate * grad
    weights += delta
```

## b. 随机梯度下降

Stochastic Gradient Descent

所谓随机, 就是

* 取的位置随机
* 取之前被随机打乱

随机打乱有助于提高梯度下降的性能, 详见[^3]

缺点:

* 过于自由, 导致训练loss波动太大, 不易收敛
* 容易被一个坏样本带偏

```python
while True:
    i = rand(range(0,len(training_set)))
    grad = gradient(loss_f, weights, training_set[i])
    delta = -learning_rate * grad
    weights += delta
```

## c. 小批量随机梯度下降

Mini-batch (Stochastic) Gradient Descent

> 目前所说的SGD, 除非特别注明, 否则都是指Mini-Batch SGD.

```python
while True:
    grad = gradient(loss_f, weights, training_set[i, i+batch_size])
    delta = -learning_rate * grad
    weights += delta
    i += batch_size
    i = min(i, len(training_set))
```

### (1) 新概念: batch size

每次计算梯度时所使用的样本数.

也就是, 假设训练样本只有`batch_size`这么多, 用这些来计算梯度.

### (2) 特性

收敛性: GD > MbGD > SGD

效率: SGD > MbGD > GD

仍然有问题:

* 如果训练样本的相似度较低, 收敛性仍然不好
* 容易陷入局部最优 (原因待考)

## d. 动量法

Momentum

```python
delta_last = xxxxx
while True:
    grad = gradient(loss_f, weights, training_set[i, i+batch_size])
    delta = momentum * delta_last - learning_rate * grad
    weights += delta
    delta_last = delta
```

momentum即动量, 它模拟的是物体运动时的惯性:

* 更新的时候在一定程度上保留之前更新的方向;
* 同时利用当前batch的梯度微调最终的更新方向.

这样一来, 可以在一定程度上增加稳定性, 从而学习得更快, 并且摆脱局部最优的能力比SGD好 (原因待考).

## e. Ada系列的参数更新算法

Adagrad[^4]

RMSProp[^5]

AdaDelta[^6]

Adam[^7]

# 参考文献

[^1]:  The gradient. https://www.khanacademy.org/math/multivariable-calculus/multivariable-derivatives/partial-derivative-and-gradient-articles/a/the-gradient
[^2]:  Strategy #3: Following the Gradient. https://cs231n.github.io/optimization-1/#opt3
[^3]: Why random reshuffling beats stochastic gradient descent. https://pdfs.semanticscholar.org/b4a3/28267e029f2f3b4641a488ffc2be7026cfaf.pdf
[^4]: Adaptive subgradient methods for online learning and stochastic optimization. http://www.jmlr.org/papers/volume12/duchi11a/duchi11a.pdf
[^5 ]: Rmsprop: Divide the gradient by a running average of its recent magnitude. https://zh.coursera.org/learn/neural-networks/lecture/YQHki/rmsprop-divide-the-gradient-by-a-running-average-of-its-recent-magnitude
[^6]: ADADELTA: an adaptive learning rate method. https://arxiv.org/pdf/1212.5701.pdf
[^7]: Adam: A method for stochastic optimization. https://arxiv.org/pdf/1412.6980.pdf