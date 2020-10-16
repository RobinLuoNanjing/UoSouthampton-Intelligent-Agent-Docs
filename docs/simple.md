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
(下面的操作都是在你们的lab2中)
1. 首先创建一个新的package,命名为groupn,其中n是你们组完队之后老师用邮箱发给你们的组号。
2. 从文件夹 bilateralexamples 复制文件 RandomBidderExample.java 进入你们的package。
3. RandomBidderExample.java 文件，将其包名(第一行)改为你们之前的定义的报名，比如group10。
4. 把class改名为 Myagent。
5. (Optional) 在getDescription()函数中，改变对你agent的描述。(虽然这步没什么用，纯属好玩)

完成上面5步，至此，你已经有了自己的agent😝！只不过他只是一个example复制来的，就是一个没有复杂算法的工具人🤖。之所以要复制而来，那是因为，你的agent里的代码都需要保持类似的结构，从而才能在Genius里运行。

### 可选步骤
Lab2 第2部分说的是，告诉你每次都要编译一下自己的java文件(右键 click compile groupn.java)，这样才能运行。 第3部分其实是可选的，就是给你添加了每个API方法的英文文档在编译器中。

## 理解你的Agent有哪些属性和方法
从现在开始的每一章节，每个实验都会开始介绍Genius的一些特性与方法💆‍♂️。这是比较重要的内容，因为你们的算法创新其实都是建立于Genius提供的API之上的哈。

### UtilitySpace
首先你们所要做的，就是将下面的代码复制到```init()```函数里,也就是```super.init(info);```的后面。或许你们有些人没有java基础，或者有些人只是复习了java的数据结构，对下面的还是一头雾水😭。没关系，我会带你们一行行的理解代码哒，到后面你们熟练了自然而然熟能生巧😁。哦，对了，在插入代码的过程中，可能会报错，就是要你import相应的包，你把鼠标悬浮在报错的地方，导入就可以啦😬。
```java
AbstractUtilitySpace utilitySpace = info.getUtilitySpace();
AdditiveUtilitySpace additiveUtilitySpace = (AdditiveUtilitySpace) utilitySpace;

List< Issue > issues = additiveUtilitySpace.getDomain().getIssues();

for (Issue issue : issues) {
    int issueNumber = issue.getNumber();
    System.out.println(">> " + issue.getName() + " weight: " + additiveUtilitySpace.getWeight(issueNumber));

    // Assuming that issues are discrete only
    IssueDiscrete issueDiscrete = (IssueDiscrete) issue;
    EvaluatorDiscrete evaluatorDiscrete = (EvaluatorDiscrete) additiveUtilitySpace.getEvaluator(issueNumber);

    for (ValueDiscrete valueDiscrete : issueDiscrete.getValues()) {
        System.out.println(valueDiscrete.getValue());
        System.out.println("Evaluation(getValue): " + evaluatorDiscrete.getValue(valueDiscrete));
        try {
            System.out.println("Evaluation(getEvaluation): " + evaluatorDiscrete.getEvaluation(valueDiscrete));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



1.  ```init()```
这是你的Agent的初始化函数，可以把它当做是整个Agent的入口🚪。也就是说，比赛的刚开始，你需要把所有的状态准备好，最主要的是预测自己的模型。这么说有点抽象，那我打个比方吧，就想运动员准备比赛一样，比赛前需要补能量，热身啥的，咱们的agent也需要时间去做准备。也就是在1分钟左右的时间内，去计算自己的模型。然后才进行Negotiation。在这里，咱们先别管如何预测自己的模型。只需要知道init的作用就好啦。

2.  ```super.init(info)```
这个是继承父类的初始化方法。封装，继承和多态，是面向对象的三个基本特性。没有面向对象语言基础(c++,java,python)的童鞋，可以补一补相关的内容。当然，你也可以不去理解这个代码的意思🙅‍♀️，不影响你后面的设计。

3. 下面的代码的意思为首先要通过```info.getUtilitySpace()```获得一个类型为``` AbstractUtilitySpace``` 的 ```utilitySpace ```。这个```utilitySpace ```就是你个人的preference profile,然后你需要将它转为```AdditiveUtilitySpace```,因为我们只考虑这个情况。🙋‍♂️有人肯定会问，不是不知道自己的preference嘛，不是要自己计算的嘛。对，这个实验只是帮你了解这些用法，所以才用这个方法。真实比赛的时候，是获取不到自己真实的```utilitySpace ```。
```java 
AbstractUtilitySpace utilitySpace = info.getUtilitySpace();
AdditiveUtilitySpace additiveUtilitySpace = (AdditiveUtilitySpace) utilitySpace;
```



### Concession Strategy
