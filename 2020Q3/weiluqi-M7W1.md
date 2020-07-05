## 论文阅读 Generating Labels from Clicks
[论文链接](https://www.microsoft.com/en-us/research/publication/generating-labels-from-clicks/?from=http%3A%2F%2Fresearch.microsoft.com%2Fapps%2Fmobile%2Fpublication.aspx%3Fid%3D77211)

### 面临的问题
搜索引擎排序算法依赖大量训练数据，全靠人工标注无法满足需求。（09年微软的论文，主要关注的还是相关性，而不是电商排序关注的转化率）

人工标注的一般操作：
1. 根据每个（query、urls）pair的相关程度，标注Perfect、Excellent、Good、Fair、Bad五种不同label。
2. 人工标注难免会受主观判断影响，因此一般一份数据会由多人标注，然后结合多人标注作为最终的标注结果

人工标注的缺点：
1. 劳动密集又耗时，太昂贵
2. 搜索内容不断更新变化，要保证样本的完备性和正确性，需要不断进行标注，持续烧钱。

### 前人的工作
1. Thorsten Joachims. Optimizing search engines using clickthrough data. In KDD, pages 133–142, 2002.
2. Thorsten Joachims, Laura A. Granka, Bing Pan, Helene Hembrooke, and Geri Gay. Accurately interpreting clickthrough data as implicit feedback. In SIGIR, pages 154–161, 2005.

前人论文没有看，看描述应该指的是skip-above、skip-next等方法。
> These rules (e.g., clicked documents are better than preceding skipped documents) when applied to a click log produce pairwise preferences between the URLs of a query, which are then used as a training set for a learning algorithm.

前人方法的缺陷：
1. 假设的用户浏览模型被过度简化了，很多用户行为没有捕捉。
2. 假设的搜索场景是对于相同的搜索应该给出相同的结果排序，但真实的搜索引擎对于相同的query不同人呈现的结果也不同，对于相同的query相同的人不同的时间呈现的结果也不同。
3. 这些方法只能构建pairwise的标注数据，但很多排序算法需要的是pointwise的标注数据。
> Third, the goal is to generate pairwise preferences which are then used directly for training. However, several learning algorithms operate on labeled training data. For such algorithms it is important to produce labels for the (query, URL) pairs.

### 本文的工作
#### 核心思路
使用skip-above和skip-next的方法，构造url的偏序对，聚合同一query下的所有偏序对，构造成一个有向图，每个顶点是每个url，边的方向是从点击的url指向未点击的url，目标是给所有顶点标注一个label，使得整个图中正反馈边的权重-负反馈边的权重最大。
#### 什么是正反馈边、负反馈边
两个顶点标注的label之间的偏序与从点击日志中获取的偏序对的偏序相反，则是负反馈边，否则是正反馈边。
#### 构造图时，边的权重如何计算
做了一个用户调查，统计了这样的一个概率P[ij]=P(read i | click j)，这个概率表示的是用户点击了第j个位置的条件下，一共浏览到第i个位置的概率。构造偏序对的两个url，如果其中发生点击行为的url位置为j，未点击的url为i，其对应的边的权重就是P[ij]，多个相同url的偏序对，权重相加
#### 是否可解
文章证明了当label的数量为2时，问题的时间复杂度是O(|E|)，E是边的数量。当label数量无穷时，是个NP-hard问题。对于其它数量的label的情况，他们leave open了。。。。。
证明过程没看

#### 如何求解
TODO



注意力
