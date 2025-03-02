---
title: "SE-ML04-决策树"
date: 2024-09-30T08:54:34-08:00
categories: 
- "机器学习"
featureimage: https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202409301003448.png
summary: "决策树就是一颗 `if-else` 树。树的每一个结点代表一个决策（测试），同时也代表了这个决策所对应的一个样本空间。我们的目标就是通过多次产生 `if-else` 分支，尽可能地让决策树的叶子结点只..."
---

## 基本思想

决策树就是一颗 `if-else` 树。树的每一个结点代表一个决策（测试），同时也代表了这个决策所对应的一个样本空间。我们的目标就是通过多次产生 `if-else` 分支，尽可能地让决策树的叶子结点只包含相同标签的样本。

既然是一棵树，我们就从**递归建树**的角度来理解决策树的基本思想。

### 基础

决策树的根节点代表了整个样本空间，没有任何划分或决策。

下图的例子只有连续属性，将就看吧。

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202409301003448.png)

### 递归

假设我们在某一结点下。只考虑这个结点所代表的子样本空间。

建树就是要加深树的深度，即产生 `if-else` 分支。具体来说就是在当前的样本空间中插入决策边界，划分出的多个新的样本空间就是儿子们的样本空间。

我们每次递归只选择样本的**某一个属性**进行划分。

#### 离散属性

将这个属性的每一个取值都划出一个空间。

假如这个属性$a$取值集合是$D={x, y, z}$，我们就划出三个空间，对应这三个取值。这样这个结点下就生出了三棵子树。`if-else`如下：

```c
if(a == x) {
	SubTree1;
}else if(a == y) {
	SubTree2;
}else {
	SubTree3;
}
```

#### 连续属性

基本方法就是将连续属性变成离散属性。常用的方法是二分法。

对连续属性$A$，取合适的值$x$。通过如下函数，我们能得到一个离散属性$B=\{X, Y\}$。

$$
b=f(a)=\left\{
  \begin{aligned}
  X, a\le x \\
  Y, a\gt x
  \end{aligned}
  \right.
$$

取合适的值一般会取中位数。

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202409301008219.png)

如上图，我们横着插了一条决策边界，这条边界代表如下决策：

```c
if(x <= 0.0596) {
	LeftSubTree;
}
else {
	RightSubTree;
}
```

### 停止条件

- 只要当前结点的样本空间只存在一类标签的样本，我们就停止递归此结点。

> 这样会导致十分严重的过拟合。解决方法看后面。

- 当前属性集为空，或是所有样本在所有属性上取值相同，无法划分
- 当前结点包含的样本集合为空，不能划分

## 取最优划分属性

我能想到三种取决策边界的方法。

### 随便取

没错，~~随机永远是你大爷~~。我们随机取一个属性进行划分。

只要递归次数够多，一定能到达停止条件。只是这样建出来的树可能够让大半个中国在树底下乘凉了。

### 信息增益

> ID3 决策树

假定当前样本集合$D$中第$k$类样本所占的比例为$p_k$
定义**信息熵**如下：

$$Ent(D)=-\sum_{k=1}^{|y|}p_klog_2p_k$$

> 信息熵是度量样本集合“纯度”最常用的一种指标

说人话，**样本集合越杂，信息熵越大**。

假设离散属性$a$的取值集合为$\{a^1,a^2,...,a^V\}$，$D^v$是$D$中在$a$上取值等于$a^v$的样本集合。
由此定义以属性$a$对$D$进行划分的**信息增益**为：

$$
	Gain(D,a)=Ent(D)-\sum_{v=1}^{V}\frac{|D^v|}{|D|}Ent(D^v)
$$

可以看到，信息增益就是以属性$a$划分前后的信息熵差值来定义的。而划分后的信息熵是按样本数为权重求和来的。

信息增益越大，说明以这个属性划分后信息熵降低得越多，划分后的子样本空间最纯。所以我们对所有属性中**增益最大**的进行划分就好了。

### 增益率

> C4.5 决策树

只有一页 PPT。

增益率是对信息增益的补充。

![](https://runzblog.oss-cn-hangzhou.aliyuncs.com/postimg/202409301449718.png)

### 基尼指数

> CART 决策树

基尼指数**本质**反映了从$D$中随机抽取两个样例，其类别标记不一致的概率。

定义如下：

$$
\begin{equation}
	\begin{aligned}
		Gini(D) &= \sum^{|y|}_{k=1}\sum_{k^{'}\neq k}p_kp_{k^{'}} \\
				&= 1-\sum^{|y|}_{k=1}p_k^2 \\
				&= 1-\sum^{|y|}_{k=1}(\frac{|D^k|}{|D|})^2
	\end{aligned}
  \end{equation}
$$

其中，$p_k$是样本点属于第$k$类的概率，$D^k$是样本集合$D$中属于第$k$类的样本子集。

而属性$a$划分下的基尼指数为：

$$
	Gini\_index(D,a)=\sum_{v=1}^{V}\frac{|D^v|}{|D|}Gini(D^v)
$$

即按样本数对基尼指数加权平均求和。同样地，我们取$Gini\_index$最小那个属性就行了。

## 剪枝

前面说了，决策树的停止条件之一是叶子只包含一类标签的样本，而这样会导致很强的过拟合，在训练集上也会达到 100%的正确率。所以我们需要剪枝来增强泛化性能。

### 预剪枝

> 及早停止树的生长

#### 限制树的高度

这个很好理解。结点一旦长到一定高度了，就直接不让它继续往下长了。

#### 评估法

我随便起的名字。

思路是每次生长的时候，评估生长后的决策树在**验证集**上的表现。然后根据一定的策略决定是否继续生长下去。

比如，如果生长后验证集准确率降低了，我就不让树继续长了；否则，树就可以继续长。

#### 其他

剪枝这个东西，可操作空间相当大。根据实际任务去选择合适的剪枝方案才是可行之法。同时，不同的剪枝方法也可以相互融合，生成效果更好的剪枝。

### 后剪枝

> 随后删除或折叠信息量很少的结点

#### 评估法

还是评估法，没想到吧！

思路是将已经长好的结点收缩。

详细一点：

1. 我们考虑要剪枝的结点$k$，以$k$为根节点的子树有叶子$\{l_1, l_2, l_3...l_n\}$

2. 从 1 到 n，将这棵子树坍缩成叶子，并根据验证集评估整颗决策树。

3. 选择评估结果最好的叶子，再决定是否剪枝即可。

中间涉及的一些策略很含糊，因为这是根据实际去替换的部分。

### 对比

- 时间开销

预剪枝：测试时间开销降低，训练时间开销降低

后剪枝：测试时间开销降低，训练时间开销增加

- 过/欠拟合风险

预剪枝：过拟合风险降低，欠拟合风险增加

后剪枝：过拟合风险降低，欠拟合风险基本不变

- 泛化性能

后剪枝 通常优于 预剪枝

## 其他

### 连续值和离散值

- 连续值 -> 离散值

一般而言，可以用**二分法**。

即取属性值的中位数$med$，对于取值小于$med$的归为一号类，其他归为二号类。

- 离散值 -> 连续值

离散值编号就行。


## 代码实现

我们来通过 Python 实现一个决策树。这个决策树是将所有属性视作连续值，并且使用二分法，所以建出来是二叉树。

> 完全是我闭门造车实现的，只是借鉴了`sklearn`的api，具体实现应该十分不标准，请勿过度参考！

### 结点类

既然是一颗树，我们先来定义树的结点类。

```python
class Node:
    def __init__(self, ind, splitter, label):
        self.ind = ind
        self.splitter = splitter
        self.lson = None
        self.rson = None
        self.label = label
    def test(self, x):
        if self.lson is None or self.rson is None:
            return self.label
        if x[self.ind] < self.splitter:
            return self.lson.test(x)
        else:
            return self.rson.test(x)
```

初始化函数中：
- `ind`：结点负责的属性编号
- `splitter`：结点对ind号属性做决策的决策边界。小于splitter到左子树，否则~~柚子树~~。
- `label`：此结点的类别标记。表示决策做到这个结点时将样本认定为哪个类。

`test`函数：输入样本`x`，输出预测类编号。

### 决策树类

接下来，写一个决策树类。负责管理Node，并向外提供接口。

```python
class MyDecisionTree:
    def __init__(self, criterion='gini', max_depth=255):
        # criterion
        self.criterion = Criterion()
        if criterion == 'gini':
            self.criterion = Gini()
        if criterion == 'entropy':
            self.criterion = Entropy()
        if criterion == 'random':
            self.criterion = Random()
        # depth
        self.max_depth = max_depth

    def fit(self, x, y):
        self.root = self.build(x, y, 0)

    def score(self, x, y):
        acc = 0
        for i in range(len(y)):
            tar = self.root.test(x[i])
            if tar == y[i]:
                acc += 1
        print(f"Accuracy: {acc * 1.0 / len(y)}")

    def build(self, x, y, depth):
        if depth >= self.max_depth:
            return None
        clazz_count = len(np.unique(y))
        if clazz_count == 0:
            return None
        if clazz_count == 1:
            return Node(0, 0, np.unique(y)[0])
		
        ind, splitter = self.criterion(x, y)
        node = Node(ind, splitter, np.argmax(np.bincount(y)))
        
        l_rows = np.where(x[:, ind] <= splitter)[0]
        r_rows = np.where(x[:, ind] > splitter)[0]

        node.lson = self.build(x[l_rows], y[l_rows], depth + 1)
        node.rson = self.build(x[r_rows], y[r_rows], depth + 1)

        return node
```

- 初始化函数可以选择取划分属性的标准、树的最大深度。
- `fit`: 在输入的数据集上进行训练
- `score`：在输入的数据集上进行测试
- `_build`：递归建树。包含了停止条件的检查以及划分属性的选择。

### 选取属性策略

基于Criterion父类实现了`Gini`、`Entropy`和`Random`的属性选取策略。

使用了二分法，将样本属性的中位数作为分类边界。所以`Gini`和`Entropy`中的$\frac{|D^v|}{|D|}$可以认为是$\frac{1}{2}$。

```python
class Criterion:
    def __call__(self, x, y) -> tuple[int, int]:
        pass

class Gini(Criterion):
    def __call__(self, x, y) -> tuple[int, int]:
        median = np.median(x, axis=0)
        ind, gini_min = 0, 1
        for i in range(x.shape[1]):
            l_rows = np.where(x[:, i] <= median[i])[0]
            r_rows = np.where(x[:, i] > median[i])[0]
            gini_i = 0.5 * (Gini.clac_gini(y[l_rows]) + Gini.clac_gini(y[r_rows]))
            if gini_i < gini_min:
                ind = i
                gini_min = gini_i
        return ind, median[ind]

    def clac_gini(y):
        uy, counts = np.unique(y, return_counts=True)
        probabilities = counts / counts.sum()
        gini = 1 - np.sum(probabilities ** 2)
        return gini

class Entropy(Criterion):
    def __call__(self, x, y) -> tuple[int, int]:
        median = np.median(x, axis=0)
        ind, gain_max = 0, 0
        for i in range(x.shape[1]):
            l_rows = np.where(x[:, i] <= median[i])[0]
            r_rows = np.where(x[:, i] > median[i])[0]
            gain_i = Entropy.clac_entropy(y) - 0.5 * (Entropy.clac_entropy(y[l_rows]) + Entropy.clac_entropy(y[r_rows]))
            if gain_i > gain_max:
                ind = i
                gain_max = gain_i
        return ind, median[ind]

    def clac_entropy(y):
        uy, counts = np.unique(y, return_counts=True)
        probabilities = counts / counts.sum()
        ent = - np.sum(probabilities * np.log2(probabilities))
        return ent

import random
class Random(Criterion):
    def __call__(self, x, y) -> tuple[int, int]:
        ind = random.randint(0, x.shape[1] - 1)
        median = np.median(x[:, ind])
        return ind, median
```

### 使用与测试

在breast cancer数据集上做测试。

```python
ds = datasets.load_breast_cancer()
x = np.array(ds.data)
y = np.array(ds.target)

x_train, x_test, y_train, y_test = train_test_split(x, y, test_size=0.2, shuffle=True)
```

借鉴了sklearn的api，所以使用方法基本是一致的。

```python
tree = MyDecisionTree(criterion= 'gini', max_depth=20)

tree.fit(x_train, y_train)

tree.score(x_train, y_train)
tree.score(x_test, y_test)
```

Output:

```
Accuracy: 0.978021978021978
Accuracy: 0.9473684210526315
```

效果还不错。

## 后记

还有很多东西可以实现，比如剪枝、计算熵和基尼指数的矩阵加速、属性贡献率等等。但是我懒得在这里花太多时间了。