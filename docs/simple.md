# 一个简单的Agent

## 理论名词解释

### SAOP: Stacked Alternating Offers Protocol
所谓堆叠式交替报价协议(SAOP,这个谷歌翻译真是绝了😐 )，我们可以理解为一个negotiation协议，它是双方(Bilateral)交替报价，如果你报出当前的offer，那好，对方可以做出三个反应来应对你的offer：

1. 对方也出一个offer👊 ，看你同意不同意。(这意味着对方不同意你的offer，所以提出了一个新的，堆叠覆盖了你的offer)。
2. 对方同意你的offer🤝 。皆大欢喜。
3. 对方选择离开，不跟你进行交易😭 。这种情况基本上很少考虑，因为对方的目的就是最大化utility，如果对方就这么一走了之，那么utility就为0。

其实你们上课会提到其他的protocal。但是coursework就是只有SAOP协议的形式。


### Reservation Value
Reservation Value(保留值)，可以理解为你的底线，utility如果低于这个值，你就直接拒绝交易了。这就有点像妈妈👩‍🦳在菜市场买一个市场价为20块的鱼🐟，砍价到1块钱，渔贩子的reservation value是2块，所以他干脆就不卖给妈妈了。

需要注意的是，有时候比赛会设定这样的规则，如果双方都没达成交易，那双方各分0.5 utility。这时候，你可以设定一个reservation value=0.5,如果对面报价低于0.5，那你索性跟他死磕到底，反正最后最低也能拿到0.5 utility。

但是如果比赛规则为，双方没达成交易，两边都得0分，那reservation value还是设低点吧，有总比没有好🤦‍♀️
。

### Time Pressure
Time pressure描述的是一种情境，意味你的出价策略会随着seconds或者rounds流逝而发生改变。比如你会不断自己的offer的utility来尽快达成协议。

lab2 里提到了一个discount factor $\delta$ (打折因子)💰。什么意思呢？就是有些时候的negotiation会加入discount factor,你的offer的 utility会随着时间的流逝，而大打折扣。这会促进双方都快点达成协议，而不是死耗着。

当然，coursework应该不需要考虑这种情况。

## 运行你的Agent
在进行这一部分之前，得确定你已经完成了Quick Start章节的环境的安装并且已经成功的把[这个视频](https://www.youtube.com/watch?v=ES_bpdRiSNM)里的操作完成哈😊。

### 创建一个Agent
1. 首先创建一个新的package,命名为groupn,其中n是你们组完队之后老师用邮箱发给你们的组号。
2. 