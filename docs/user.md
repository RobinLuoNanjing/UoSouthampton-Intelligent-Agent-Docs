# 知己
上一章节，我们已经学习了如何预测对手模型。那么这一章节，我们将会学习如何预测自己的模型(User model)🐷。

因为大部分人在预测对手模型时，都采用了Johny Black的算法。这也就意味着，在预测对手模型时，大家的水平基本上都一样，那么，能拉开差距的就是预测User Model🥬。

## Elicitation

### Elicitation的概念
在Lab4的开始，讲到了一个概念，叫 preference elicitation (偏好启发)🧐。什么意思呢？我们可以通过Lab4中的一张图更好的理解其中的意思。
![Elicitaion](img/user/elicitation.png)

作为Agent的你，其实是不知道User具体的偏好是啥。我们需要通过问询User你喜欢啥，来判断User的偏好。当然了，User自己也不可能精确的告诉你他更喜欢哪个attribute,相应的evaluation是多少。对于User来说，他只能告诉你一个"an ordinal ranking of outcomes"。就是User给了你一堆offer,假设10个offer，他能告诉你我这10个offer的utility从小到大的顺序。那么具体User的Utility space是什么样的，你是不会知道的🤨。

### Elicitation cost
所以，每一次negotiation的一开始，你都会通过内置的API获得一堆offer的排序 (通过 `getBidRanking()`方法，索引为0是最低的bid。)。这个ranking的数目是有限的，根据不同的场景，可以获得不同数量的bids.

那么，如果我还想获得更多的bids的排序怎么办(有些人是有这些需求的，因为当bids越多，你对User model的预测将会越准)？你是可以通过`elicitRank`方法额外的查询一定数目的bids。但是会产生相应的cost,称之为elicitation cost,或者称之为bother cost。你问的越多，最后会在你的效用上扣除相应的"咨询费用"💰。

干货🥬：从我和上届学长的经验来说，你基本上不需要进行额外的征询。原因有几个方面。

首先，对于很多高级一点的算法，比方说你用机器学习算法，或者进化算法，因为计算量都比较大，很多时候，你给我的bids越多，那么我就算的越多。但系统基本上开局只给你一分钟时间算User model。我用的遗传算法，有个学长用的梯度下降算法，都面临的问题就是，在一些Domain比较大的场景下，可能给的bid的数目非常多，接近1500个(你们可以查一下agent negotiation的评价手册，里面有提到最高的bids数目的上限是多少)。面对这么多bids的ranking，我和那个学长最后的处理方式都是选择去掉很多bids。保证能在系统设置的时间范围内算完🤣。我删都来不及呢，何必还要自掏腰包，去"咨询"额外的信息呢。

其次，elicitation cost不容易控制。我们这届出现过因为在这个点处理失误，导致最后分数偏低的，因为咨询的数目不知道在什么情况下，咨询了太多次了。

再其次，我觉得像agent GG这种时间复杂度为O(1),都没有征询过一次，还表现的如此稳健。也侧面说明了，可能额外征询是个鸡肋🍗(当然，也有可能我没有考虑很多特殊的场景)。

总而言之，能不用`elicitRank`就不用好了。

### 安装9.1.12版本 Genius
Lab4中也提到了，要想使用elicitation的机制，必须需要更新一下jar包到9.1.12版本。别忘了。


## 遗传算法预测对手模型
