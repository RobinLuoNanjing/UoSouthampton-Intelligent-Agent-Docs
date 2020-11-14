# 出价与接受的博弈
![winchester](img/action/winchester.jpg)


这一章节，我会讲一讲我的Agent是如何制定出价策略和接受策略的👀。

但是这一章节我并没有太多实质性的内容可以说，因为我自己认为我的出价和接受价格的策略其实考虑的非常不仔细。😞这可能是因为当时我并没有足够多的时间去考虑这个了，毕竟除了这个Agent大作业，还有计算机视觉的大作业连续需要完成。所以算是虎头蛇尾了。

😆但我的一些策略应该还是可以给你们提供一个比较清晰的思路。就当做是抛砖引玉了，说不定你们能想到很多刁钻的策略，刁钻到能够让我赞不绝口。

## 出价

### 如何出价
你出价的时候，会怎么出？

我的选择是从高到低，且分段进行。这里的分段是根据时间分段，而不是用一个统一的函数进行求值。至于为什么我用分段，那当然还是我一点一点测出来的。因为在正式提交之前，你们也会有一个小的 [Mini Tournament](http://www.southampton.ac.uk/~eg/competition/)。这个小的Tournament是用你的agent和往届的agent比赛，从而测试你的agent的效果的。你每次提交完了之后会有结果，这个结果可以帮助你进行参数上的微调。

下面我会讲讲我的出价策略。可以看出我的出价策略代码非常的丑陋。。。这也是没办法的，当时时间紧凑，再加上不停的微调，所以根本不想在乎可读性了😂。

同时我现在读我的代码，也发现了一些问题。我会把这些问题抛出来，你们自己理解并且改进。

1. 当时间小于0.05的时候我选择不停出一张牌，来迷惑对手。当时想的是最高效用的Bid。但是现在想想，迷惑的想法是好的。但是用错了办法。因为对方大概率也是用Johny Black,所以我一直出最高效用的Bid,那会有什么情况发生呢？🤨

2. 在时间小于0.4的时候，我会尽可能出一个给对方高的offer，但是这个offer不能比我的妥协值(Concession value)高0.05，也不能低于我的 Concession value-0.1。就是尽可能给他高的，但也不能太高。我当时的想法就是，一方面，尽可能让自己的每一次offer都能走在帕累托边界上，一方面也希望这个值能够接近纳什均衡点，即便我并不知道他在哪，但是我就估计他在我的Concession value附近。我承认，我有赌的的成分🍷。但是我也想不到更好的方法啦。

3. 在高于0.4秒的时候，实际上只是将0.1改为了0.05。因为毕竟做人要厚道，对面死不妥协，那么我也不能逼他妥协。


```java
	private Bid generateRandomBidAboveTarget()
	{
		long numberOfPossibleBids=this.getDomain().getNumberOfPossibleBids();
		double time = getTimeLine().getTime();
		Bid randomBid;
		double util;
		double opponentUtility;

		List<Bid> randomBidList=new ArrayList<>();
		List<Double> utilList=new ArrayList<>();
		List<Double> opponentUtiList=new ArrayList<>();  //存放对应Bid的对手效用
		//刚开始一直发相同的bid迷惑对手。让对手无法计算自己的模型。
		if(time<0.05){
			return maxBidForMe;
		}

		//尽可能给对手最高的bid,这个时候不需要考虑对手，因为此时的对手模型不准确。
		else if(time<0.4){
			for(int k=0;k<2*numberOfPossibleBids;k++){
				randomBid=generateRandomBid();
				util = predictAdditiveSpace.getUtility(randomBid);
				if(util>concessionValue-0.1&&util<concessionValue+0.05){
					utilList.add(util);
					opponentUtiList.add(iaMap.JBpredict(randomBid));
					randomBidList.add(randomBid);
				}
			}
			//如果找不到满足条件的，就直接生成一个效用最大的
			if(utilList.size()==0){
				for(int t=0;t<2*numberOfPossibleBids;t++){
					randomBid=generateRandomBid();
					randomBidList.add(randomBid);
					utilList.add(predictAdditiveSpace.getUtility(randomBid));
				}
				//这里可以保证，即便上次2000次循环之后没找到自己想要的bid，也能返回一个效用大的bid出来。
				double maxUtility=Collections.max(utilList);
				int indexRandBid=utilList.indexOf(maxUtility);
				Bid suitableBid=randomBidList.get(indexRandBid);
				return suitableBid;
			}
			double maxUtility=Collections.max(utilList);
			int indexRandBid=utilList.indexOf(maxUtility);
			Bid suitableBid=randomBidList.get(indexRandBid);
			return suitableBid;
		}

		//努力给对手最大的bid。
		else{
			for(int k=0;k<3*numberOfPossibleBids;k++){
				randomBid=generateRandomBid();
				util = predictAdditiveSpace.getUtility(randomBid);
				if(util>concessionValue-0.05&&util<concessionValue+0.05){
					utilList.add(util);
					opponentUtiList.add(iaMap.JBpredict(randomBid));
					randomBidList.add(randomBid);
				}
			}
			//如果找不到满足条件的，就直接生成一个自己效用最大的
			if(utilList.size()==0){
				for(int t=0;t<2*numberOfPossibleBids;t++){
					randomBid=generateRandomBid();
					randomBidList.add(randomBid);
					utilList.add(predictAdditiveSpace.getUtility(randomBid));
				}
				//这里可以保证，即便上次2000次循环之后没找到自己想要的bid，也能返回一个效用大的bid出来。
				double maxUtility=Collections.max(utilList);
				int indexRandBid=utilList.indexOf(maxUtility);
				Bid suitableBid=randomBidList.get(indexRandBid);
				return suitableBid;
			}
			else{
				double opponentMax=Collections.max(opponentUtiList);
				System.out.println("对手最大的效用为："+opponentMax);

				int indexOpponent=opponentUtiList.indexOf(opponentMax);
				Bid suitableBid=randomBidList.get(indexOpponent);
				System.out.println("自己的效用为："+predictAdditiveSpace.getUtility(suitableBid));
				return suitableBid;
			}
		}

	}
```


### 如何实现出价的创新？
这里我会说说有哪些点，你们可以创新。

1. 关于如何迷惑对手。现在你们已经知道了，大部分预测你的模型都是用的Johny Black，那么你能不能想出一个出价策略，用来迷惑对手且不暴露自己呢？

2. 关于纳什均衡点，其实我猜的纳什均衡点是在0.8左右。但是这是不准确的。比如有没有方法，通过计算两个offer，一个offer是自己的最高值，对手的最低值，一个offer是自己的最低值，对手的最高值。然后将这两个点连接起来，最后取中点做圆，然后做垂线，垂线与圆的交点会不会更接近纳什均衡点？当然这都是我的想象啦。你们可以试试。

3. 关于时间选择。对手可能在最后短暂的时间妥协，那么我们能不能也死扛到最后呢？当然也不能太后，不然对手一下出个离谱的价格，被你接受了，虽然utility你是高的，但是，你们报价远离纳什均衡点，那个分数会扣的很多噢。


## 接受

### 如何接受
我的接受价格策略就更加暴力了。全分段函数，老方法，不停的调试参数。

为什么我采用这种方式？因为看了往届ANAC的比赛的代码，很多人都采用了这种类似的结构。可能这就是调参的玄学吧🤪。

说实话，我当时的脑袋已经是一团浆糊了，实在想不出什么高端的出价策略。但是，我还是通过最后的比赛结果，反应出了我接受offer的一些问题。

```java
	@Override
	public Action chooseAction(List<Class<? extends Action>> possibleActions)
	{
		double time = getTimeLine().getTime();  //***    #这里的time是正规化[0,1]范围。是相对于round的数目
		System.out.println("时间是："+time);
		calculateConcession(time);  //通过时间来控制自己的阈值，底线

		if(lastOffer!=null){
			//坚决不接受高效用的offer。不然的话离纳什均衡点会很远。
			if(time<0.3){
				if(predictAdditiveSpace.getUtility(lastOffer)>concessionValue||predictAdditiveSpace.getUtility(lastOffer)>0.88){
					return new Accept(getPartyId(), lastOffer);
				}
				return new Offer(getPartyId(), generateRandomBidAboveTarget());
			}
			else if(time<0.5){
				if(predictAdditiveSpace.getUtility(lastOffer)>concessionValue||predictAdditiveSpace.getUtility(lastOffer)>0.87){
					return new Accept(getPartyId(), lastOffer);
				}
				return new Offer(getPartyId(), generateRandomBidAboveTarget());
			}
			else if(time<0.7){
				if(predictAdditiveSpace.getUtility(lastOffer)>concessionValue||predictAdditiveSpace.getUtility(lastOffer)>0.85){
					return new Accept(getPartyId(), lastOffer);
				}
				return new Offer(getPartyId(), generateRandomBidAboveTarget());
			}

			else if(time<0.8){
				if(predictAdditiveSpace.getUtility(lastOffer)>concessionValue||predictAdditiveSpace.getUtility(lastOffer)>0.83){
					return new Accept(getPartyId(), lastOffer);
				}
				return new Offer(getPartyId(), generateRandomBidAboveTarget());
			}
			else if(time<0.85){
				if(predictAdditiveSpace.getUtility(lastOffer)>concessionValue||predictAdditiveSpace.getUtility(lastOffer)>0.8){
					return new Accept(getPartyId(), lastOffer);
				}
				return new Offer(getPartyId(), generateRandomBidAboveTarget());
			}

			else if(time<0.9){
				if(predictAdditiveSpace.getUtility(lastOffer)>concessionValue||predictAdditiveSpace.getUtility(lastOffer)>0.7){
					return new Accept(getPartyId(), lastOffer);
				}
				return new Offer(getPartyId(), generateRandomBidAboveTarget());
			}

			else if(time<0.93){
				if(predictAdditiveSpace.getUtility(lastOffer)>concessionValue||predictAdditiveSpace.getUtility(lastOffer)>iaMap.JBpredict(lastOffer)){
					return new Accept(getPartyId(), lastOffer);
				}
				return new Offer(getPartyId(), generateRandomBidAboveTarget());
			}

			//如果你给的offer满足我的阈值，我就接受。。或者这个效用大于对手的效用（多大于0.1防止计算不准确）
			else if(time<0.98){
				if(predictAdditiveSpace.getUtility(lastOffer)>concessionValue||predictAdditiveSpace.getUtility(lastOffer)>iaMap.JBpredict(lastOffer)){
					System.out.println("对手："+iaMap.JBpredict(lastOffer));
					System.out.println("自己："+predictAdditiveSpace.getUtility(lastOffer));
					return new Accept(getPartyId(), lastOffer);
				}
				return new Offer(getPartyId(), generateRandomBidAboveTarget());
			}
			//最后时刻直接妥协了。
			else {
				return new Accept(getPartyId(), lastOffer);
			}
		}
		//直接结束。
		return new Offer(getPartyId(), generateRandomBidAboveTarget());
```

### 接受offer的创新点
1. 上面我提到了我的agent的问题，那么具体是什么问题呢？就是太软弱，太不aggressive了。为什么这么说呢？这里要给一个干货干货干货🐩！当时我把我的agent调参调的太容易妥协了。要知道，很多时候，不达成协议，就不得分，但是好像最后的比赛，不达成协议是件很正常的事，好像就不会算入平均得分。这也就意味着，你不一定每次都要达成协议。那么我为什么调成这样呢？都是因为参加mini tournament的时候，我18个比赛，经常只能完成17个，因为往年有个叫AgentGP的Agent，他总是跟我死刚到底。而且他的计算非常非常慢(GP的意思是Gaussian Process，你们下学期的FAI会学到，或者有人本科学过随机过程，差不多就是那个意思。)我为了和他完成negotiation，不停的调妥协，但是最后的比赛的数据分析告诉我，没那个必要哦🐌。

2。 注意，你也不能太aggressive，不然你也很多玩不成。现在回想起来，还是把重心放在最后的0.1时间内。为什么呢？因为大部分人都会在这个时间段开始妥协，这也是最容易靠近纳什均衡点和拿高utility的机会(至少我是这么觉得的)🕸


## 总结
这一章节，我还是比较惭愧只能给你们一个思路或者方向，具体的调试还是得靠你们自己啦。凡是要相信自己，要去试，试了不成功，记下来，写在report里，告诉老师你尝试了什么，为什么不成功，可以如何改进啥的，老师一般比较喜欢看这个。

接下来的一章，可能我会讲最后如何打包成老师需要的格式提交给服务器去测试，以及如何写一篇好的report。暂时只能想到这些啦。估计下一章也是最后一章啦，加油。