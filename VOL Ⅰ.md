# Overview

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220512105424688.png" alt="image-20220512105424688" style="zoom: 80%;" />

# Cha1. 在基因组数据中找到DNA复制起始位点

基因组复制事件在生物体中的重要性

​	DNA复制过程，保证DNA复制所需的全部分子保障（DNA聚合酶等）

​	需要将复制这一过程转换为计算机问题

​	复制起点：replication origin（OriC）

​	==重要科学问题：如何找到OriC==

## 问题描述

<img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205121112256.png" alt="image-20220512111238171" style="zoom:80%;" />

​	生物学问题与计算生物学问题的区别

​		面对上述的问题描述，生物学家使用实验技术来确定OriC：删除多个基因组小片段以确定哪个片段的删除会使得片段无法进行复制	

​		计算机科学家会认为需要更多的信息

**——即 Hidden Messages in the Replication Origin**  

> 以寻找细菌基因组中OriC为例
>
> 已有的研究：知道细菌基因组中OriC一般为几百bp的片段
>
> 目的：根据已知的细菌OriC，推断未知OriC序列的细菌中的OriC

书中以霍乱弧菌为例，已知其OriC序列为

> atcaatgatcaacgtaagcttctaagcatgatcaaggtgctcacacagtttatccacaac
> ctgagtggatgacatcaagataggtcgttgtatctccttcctctcgtactctcatgacca
> cggaaagatgatcaagagaggatgatttcttggccatatcgcaatgaatacttgtgactt
> gtgcttccaattgacatcttcagcgccatattgcgctggccaaggtgacggagcgggatt
> acgaaagcatgatcatggctgttgttctgtttatcttgttttgactgagacttgttagga
> tagacggtttttcatcactgactagccaaagccttactctgcctgacatcgaccgtaaat
> tgataatgaatttacatgcttccgcgacgatttacctcttgatcatcgatccgattgaag
> atcttcaattgttaattctcttgcctcgactcatagccatgatgagctcttgatcatgtt
> tccttaaccctctattttttacggaagaatgatcaagctgctgctcttgatcatcgtttc  

隐藏的信息：细胞是怎样在一个较大的基因组中识别出来对应的OriC之后开始进行复制的呢？==在OriC上存在有DnaA box，DnaA会结合到这一区域之后介导复制的起始==

那么问题就转变称为：怎样在不提前知道DnaA box构成的情况下，寻找到这一结构？

<img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205121131310.png" alt="image-20220512113112235" style="zoom:80%;" />

但是在上述的问题描述中 hidden message仍然没有被很好地描述

这时候又给了一个密码学的例子，羊皮纸上的密语解密

> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205121313061.png" alt="image-20220512131326974" style="zoom:80%;" />
>
> 发现在这段文字中 ;48出现的频率最高，又基于源码是英文，所以认为；48是英文中出现最多的单词 the，之后将对应的密码进行转换
>
> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205121314612.png" alt="image-20220512131421547" style="zoom:80%;" />

从这个例子类比：假设DNA也具有自身语言，尝试在霍乱弧菌的OriC中找到类似上文“；48”一样出现频率较高的序列，==并且在生物学上，多种生物学功能相关的区域都会出现对应的特定序列==

对这个计数过程进行问题抽象：COUNT(Text，Pattern)

> ![image-20220512131940684](https://gitee.com/wangjx68/notebook_pic/raw/master/202205121319743.png)
>
> 注意要允许模式匹配对象可以部分重叠
>
> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205121320795.png" alt="image-20220512132005696" style="zoom:80%;" />

解决问题的计数策略：滑窗扫一遍序列，确定滑窗内的子序列是否与提供的模式匹配

> 0-based
>
> 记在位置 i 开始的k-mer为 Text(i, k)，得到对应的伪代码
>
> ![image-20220512132453332](https://gitee.com/wangjx68/notebook_pic/raw/master/202205121324407.png)

这样就对之前的问题有了一个更确切/严格的描述

> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205121351728.png" alt="image-20220512135146592" style="zoom:80%;" />
>
> 在书中作者使用了一个数组计算从各个position开始的k-mer出现的次数
>
> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205121516546.png" alt="image-20220512151613322" style="zoom:80%;" />
>
> 伪代码如下
>
> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205121517135.png" alt="image-20220512151719960" style="zoom:80%;" />
>
> 但是这个算法的复杂度比较高 |Text|^2^*k

根据上述的算法对霍乱弧菌OriC进行分析，获得对应的k-mer

> ![image-20220512153545020](https://gitee.com/wangjx68/notebook_pic/raw/master/202205121535151.png)

在实际研究中发现细菌的DnaA boxes通常为9 Nts长度

> The probability that there exists a 9-mer appearing three or more times in a randomly generated DNA string of length 500 is approximately 1/1300 .

这一低概率会使得我们倾向于认为这4个9-mer可能是我们要寻找的DnaAbox



现在要在这4个待选的9-mer中挑选出最可能的一个

在分析生物序列的时候需要明确生物序列的读取方向为5' → 3'， 并且在实际复制的过程中存在A-T G-C的碱基互补配对原则

这个时候需要解决一个新的问题：处理输出反向互补序列

> ![image-20220512154858780](https://gitee.com/wangjx68/notebook_pic/raw/master/202205121548911.png)

解决这个问题之后发现，对于上文中挑选出来的k-mer存在有一对反向互补序列：

> ATGATCAAG and CTTGATCAT are reverse complements of each other, resulting in the following six occurrences of these strings.  

并且在Dna A结合的过程中不存在正负链特异性，所以这两个序列都可以作为结合box

那么这时候有较大可能这两端序列就是我们要寻找的霍乱弧菌OriC DnaA box

==但是在作出最后结论之前，我们需要对这个结论进行验证：这两段序列并不是在霍乱弧菌中广泛存在的，而是只在OriC中重复出现==

这时候会遇到一个新问题：模式匹配

> ![image-20220512160826970](https://gitee.com/wangjx68/notebook_pic/raw/master/202205121608263.png)

继续往后走，发现在整个基因组中这段序列会出现17次，并且在整个基因组中没有其他成簇存在的情况，那么可以下一个比较确切的结论

> We now have strong statistical evidence that ATGATCAAG/CTTGATCAT may represent the hidden message to DnaA to start replication.  

但是这一结论仅仅在霍乱弧菌中适用，在推广到别的细菌研究中需要进行验证

> 甚至可能在霍乱弧菌中这一结论也是一种统计学巧合
>
> 并且不同细菌存在不同的DnaA BOXes

给了另外一个例子：*Thermotoga petrophila*  

------

现在需要找到一个可以进行推广的方法：在基因组中找到每一个成簇存在的k-mer

> 在一个限定的L 长度内寻找多次出现的 k-mer：设定为 t 次
>
> 书中对应的L = 500
>
> 标记为(L, t) -slump

所以描述为如下的簇搜索问题

> ![image-20220512173239120](https://gitee.com/wangjx68/notebook_pic/raw/master/202205121732231.png)
>

将这一方法应用在E.coli基因组中发现会有多个9-mer满足（500，3）-clumps；那么接下来需要怎么做去确认在Ecoli中存在的DNAAbox呢

==寻找更多的有助于帮助鉴定DnaA box的隐藏信息==

详细研究整个复制过程

> 在复制过程中发生解旋，然后解旋后的链沿着不同方向进行复制，从而形成两个复制叉，在环形染色体上复制直到两个复制叉相遇终止于 replication terminus，所以terC应该是在oriC的对面
>
> ![image-20220513150839996](https://gitee.com/wangjx68/notebook_pic/raw/master/202205131508132.png)
>
> 并且DNA聚合酶在双链初步解旋之后就开始结合上DNA链进行复制，而不是等到双链完全解旋之后才开始工作（边解旋边复制）
>
> 而在图中红色所示的部位，存在primer，DNA聚合酶需要结合到primer上之后才能开始进行复制
>
> （粗略地）总的来说：在DNA双链解旋之后，DNA聚合酶结合到primer上，然后双链上一共4个聚合酶由oriC开始进行复制（添加核苷酸到新链上），4个聚合酶沿着顺时针或逆时针方向复制到terC结束复制，最后产生一个新的环状染色体，以备细胞进行分裂。

==上面的描述只是为了简化：假设了DNA聚合酶可以从两个方向进行复制==：**需要对模型进行修正**

> 但是事实上DNA聚合酶只能在模版链上以3' → 5' 的方向移动
>
> 所以作者对上述的简略模型进行了修改：复制具有不对称性
>
> ​	正义链与反义链
>
> ![image-20220513153126096](https://gitee.com/wangjx68/notebook_pic/raw/master/202205131531188.png)
>
> 这里描述的都是OriC到terC的方向
>
> ​	5' → 3' 的forward strand 需要不同的复制机制；
>
> ​	3' → 5'  DNA聚合酶可以正常地从oriC到terC移动
>
> ![image-20220513153735293](https://gitee.com/wangjx68/notebook_pic/raw/master/202205131537371.png)
>
> 在这种情况下，需要等到复制叉打开足够大的双链（~2000bp），在复制叉的末端产生新的primer之后新的DNA聚合酶才能够结合然后在末端开始复制（依然是沿着3' → 5'在DNA链条上进行移动）
>
> 这样就会产生存在间隙的冈崎片段
>
> ![image-20220513154102661](https://gitee.com/wangjx68/notebook_pic/raw/master/202205131541737.png)
>
> 并且在复制完成之后/ 过程中需要对不连续的冈崎片段进行连接：DNA连接酶，最终完成复制
>
> ![image-20220513154331213](https://gitee.com/wangjx68/notebook_pic/raw/master/202205131543299.png)
>
> 所以在生物学上（根据复制的快慢），反向链条又被称为引导链，正向链条又被称为滞后链

那么讨论完整个复制过程在正向和反向链条上的不对称性，开始着手将这一特性应用在算法中

> 依据上文表述，那么在正向链的整个存在过程中，它会相较反向链更多地以单链的形式存在
>
> 而单链相比双链更不稳定：容易存在突变并且无法修复
>
> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205131605519.png" alt="image-20220513160511443" style="zoom:80%;" />
>
> ==也就是说：4种碱基中更容易发生突变的碱基在正向链中存在的比例更低==
>
> 希望使用这一差异来确定一个未知的oriC的位置

==实际上：C 容易通过脱氨基反应突变为T，所以在正向链上C 的含量会更少，在单链的形式下脱氨基反应发生的概率为双链的100倍==

> C - G → T - G → 在复制中修复为T - A， 从而导致在反向链上G的减少

利用上述特性进行oriC的定位

> 在正向链上，#G - #C 为正数，在负向链上#G - #C 为负数
>
> ==那么假设我们在基因组上移动，#G - #C增加的时候我们应该是在正向链上，减少的时候我们应该是在负向链上==

这时候定义 SKEW~i~(Genome)

> We define SKEWi(Genome) as the difference between the total number of occurrences of G and the total number of occurrences of C in the first i nucleotides of Genome.
>
> 并且可以据此定义在全基因组上作图 skew diagram：skewi 与 skew i+1具有传递关系
>
> <img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220513161253209.png" alt="image-20220513161253209" style="zoom:80%;" />

可以根据skew diagram确定oriC的位置

> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205131616206.png" alt="image-20220513161628140" style="zoom:80%;" />
>
> Thus, the skew should achieve a minimum at the position where the reverse half-strand ends and the forward
> half-strand begins, which is exactly the location of oriC!  
>
> 转化为如下问题
>
> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205131643033.png" alt="image-20220513164312958" style="zoom:80%;" />

使用这个算法找到了Ecoli中oriC的位置，但是在500bp的窗口中没有发现有重复出现的9-mer

那需要继续找隐含的信息

> 不同9-mer之间差别仅仅为1个点突变
>
> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205131650129.png" alt="image-20220513165047029" style="zoom:80%;" />
>
> 这些在*Vibrio cholerae*    OriC中出现的情况更具统计学显著性
>
> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205131653004.png" alt="image-20220513165333931" style="zoom:80%;" />
>
> ==并且在实际研究中发现DnaA可以与DnaA box 或者与DnaAbox序列相近的区域结合==

由此引发出一个问题：计算两条序列之间的汉明距离，HammingDistance(p, q)

> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205131655188.png" alt="image-20220513165531123" style="zoom:80%;" />

然后使用汉明距离定义的相近k-mer方法去进行模式匹配，寻找到可能的成簇存在（相近）序列

> <img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220513170205362.png" alt="image-20220513170205362" style="zoom:80%;" />

现在算法的目的已经变成了确定在一个区域内出现次数最频繁的k-mer，并且允许存在d个错配

> <img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220513171800094.png" alt="image-20220513171800094" style="zoom:80%;" />
>
> ==需要注意的是，课本中给了一个例子说明不出现在text k-mer列表中的k-mer也可以作为most frequenct k-mer==
>
> 我的想法是做表：但是怎样在做表的过程中减少程序的复杂度
>
> 降低复杂度：只考虑k-mer的邻接k-mer
>
> 所以问题被转化为怎么去创建邻接k-mer列表

并且我们在前面提到还有反向序列的存在，所以需要对反向序列也进行处理

> <img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220515113506429.png" alt="image-20220515113506429" style="zoom:80%;" />

做最后的尝试

> We now make a final attempt to find DnaA boxes in E. coli by finding the most frequent9-mers with mismatches and reverse complements in the region suggested by the minimum skew as oriC.

在使用skew 最小化的方法中找到对应的位置，并且在对应位置旁边找到对应的9-mer 重复度较高（正向序列与反向互补序列）

并且通过实验验证该位点即为oriC位点

> 但是在这里出现一个问题，除了被证实的DnaAbox之外，还存在有其他4个序列也在500-nt的滑窗中出现较多次数

==这些high frequent的序列可能也执行特定的功能==

这也表明当前的技术在预测DnaA box方面可能是不准确的：但是提供给生物学家一些候选DnaA box序列已经是很大的帮助

==两个改进==

1. 与生物学家一起合作证明预测结果的正确性
2. 研究怎样在比较基因组学中预测oriC

后记：预测oriC在实际情况下更加复杂

1. 多个DnaA box
2. terC不一定在oriC对面：正向链和反向链长度不一
3. skew diagram的预测oriC位点只是粗略的估计，一般需要使用更大的滑窗在其周围进行检测，但是可能会导致引入重复序列
4. 并且skew diagram可能存在多个峰，呈现波动型，无法确定oriC

开放性问题

> <img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220516131808875.png" alt="image-20220516131808875" style="zoom:80%;" />

细菌中可能存在有多个oriC：在进化上讲得通，如果基因组足够长并且复制足够慢，那么具有多个oriC可以降低复制所需的时间，从而提高适应性

> skew diagram 有多个局部峰可能预示有多个oriC
>
> ![image-20220516135514402](https://gitee.com/wangjx68/notebook_pic/raw/master/202205161355489.png)
>
> 但是也可能是基因重组的结果
>
> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205161355246.png" alt="image-20220516135540160" style="zoom:80%;" />
>
> 水平基因转移也可能导致上述图的产生：一个物种的正向半链基因转移到另一个物种的负向半链上
>
> **解决上述问题：在不同oriC上是否能够找到对应的DnaA box 簇**

## 另外一个问题：古菌中的oriC

寻找古菌中的oriC：古菌更类似真核生物

<img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205161415050.png" alt="image-20220516141532961" style="zoom:80%;" />

其中一种古菌的skew图：显示可能有三个oriC

并且有实验证明确实该古菌存在有三个oriC

> Lundgren et al., 2004 demonstrated experimentally that Sulfolobus solfataricus indeed has three oriCs.  

在后续的古菌研究中也发现在很多种古菌中存在有多个oriC：==但是目前没有针对古菌oriC进行准确预测的工具==

留下来的问题：

 	1. However, if you could demonstrate that there exist two sets of identical DnaA boxes in the vicinity of two local minima in the skew diagram of Wigglesworthia glossinidia,then you would have the first solid evidence in favor of multiple bacterial oriCs.
 	2. *Methanococcus jannaschii*  的oriC位置



## 问题：在酵母中发现oriC

酵母菌具有大约400个oriC

并且真核生物基因组中存在有大量的重复片段

在酵母菌的oriC中发现存在有ARS Con'sensus Sequence（ACS）：作为origin recognition complex结合位点，并依此招募更多复制起始所需的蛋白

许多ACSs对应以下的经典T-rich片段：11nt

<img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205161436388.png" alt="image-20220516143634325" style="zoom:80%;" />

但是也存在有多于这一长度的其他片段

并且这一识别过程序列定义严格程度较低，使得仅仅通过序列分析很难对oriC进行定位

## 还有一个问题：特定pattern在一个string中出现的概率

问题描述

> We start by asking a question: what is the probability that a specific k-mer Pattern will appear (at least once) as a substring of a random string of length N?  

存在重叠词悖论

We are interested in computing the following probabilities for a random N-letter string in an A-letter alphabet:  

> <img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220516144846660.png" alt="image-20220516144846660" style="zoom: 67%;" />

Note that the above two probabilities are relatively straightforward to compute. Several variants of these questions are open:  

> <img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220516144910953.png" alt="image-20220516144910953" style="zoom: 67%;" />

# Cha1. 补充

字母表顺序定义字符，从而使用数列代替字典

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220516160922189.png" alt="image-20220516160922189" style="zoom:80%;" />

递归算法可以进行pattern to number 的计算

<img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205161620491.png" alt="image-20220516162000408" style="zoom:80%;" />

而number to pattern算法的思想可以从下面的公式中得到

<img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205161630124.png" alt="image-20220516163046050" style="zoom:80%;" />

这个公式意味着我们可以直接将最后取得的数整除4，余数即为最后一位对应的symbol_number

<img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205161645792.png" alt="image-20220516164512710" style="zoom:80%;" />

使用排序的方法减少输出freq kmer时的操作数：想法是将index记录并排序，然后连续相同的index即为出现的次数

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220516165213603.png" alt="image-20220516165213603" style="zoom:80%;" />

同样的使用index的方法可以较快地进行计数与转换：应用于clump的寻找中

> For each value of i between 0 and 4k ! 1, we will set CLUMP(i) equal to 1 if NUMBERTOPATTERN(i, k) forms an (L, t)-clump in Genome.

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220516165805050.png" alt="image-20220516165805050" style="zoom:80%;" />

## 滑窗解法的优化

注意在滑窗的过程中邻接的两个窗口滑动可以通过边缘计数的增减进行优化，从而缩减操作复杂度

并且每次滑窗移动时，只需要在增减后判断增加的pattern 出现次数是否超过 t

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220516171630070.png" alt="image-20220516171630070" style="zoom:80%;" />

## 错配匹配问题优化

在之前的问题解决思路中，仍然是先产生一个对应的4^k^ 个key-value的字典记录pattern出现的次数，并且在每次滑窗移动之后都要对所有的4^k^ key进行判断

在这个优化里面，只产生对应的邻域k-mer：blast算法中的一部分，这个邻域kmer集合包含的元素个数为3^k^ + 1

他这里写的算法也比较复杂：相较没有优化的算法多项式中少了n的一次幂

1. CLOSE FrequencyArray分别用于记录一个kmer是否是序列上模式的邻域、所有邻域的出现次数
2. 做滑窗，每个滑窗都对滑窗内的序列取得对应的邻域集合，对应邻域的CLOSE标记为1（出现过）
3. 之后对每个出现的邻域再在整个序列中做错配搜索
4. 再扫一遍，输出出现次数最高的pattern

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517095353115.png" alt="image-20220517095353115" style="zoom:80%;" />



## 邻域k-mer集合产生方法

> Our goal is to generate the d-neighborhood NEIGHBORS(Pattern, d), the set of all k-mers whose Hamming distance from Pattern does not exceed d.  

最开始先做1-neigborhood k-mer set

==最简单的做法，将所有位置上的符号都替换一遍即可==

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517095922654.png" alt="image-20220517095922654" style="zoom:80%;" />

递归做法：要求neighborhood(pattern, d) 先求 neighborhood(suffix(pattern), d)，然后在其基础上根据d的情况添加首字符		

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517101709778.png" alt="image-20220517101709778" style="zoom:80%;" />

## 大O 计数法

关系算法在最差情况下的表现

并且只关心增长最快的项

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517104235287.png" alt="image-20220517104235287" style="zoom:80%;" />

## :triangular_flag_on_post: 在一个string中出现特定pattern的概率

想象一个简单的模型

> Specifically, we can generate a random string modeling a DNA strand by choosing each nucleotide for any position with probability 1/4.  

先解决一个相对简单的问题

> We now ask a simple question: what is the probability that a **specific k-mer Pattern** will appear (at least once) **as a substring of a random string of length N?**   

以01 和 11 在一个4-mer的01序列中出现的概率为例：分别为11/16 与 8/16

简记符号

> Let **Pr(N, A,Pattern, t)** denote the probability that a string Pattern appears t or more times in a random string of length N formed from an alphabet of A letters.   

并且存在一个称为==重叠词悖论==的问题

> 不同k-mer在相同长度的随机序列中出现的概率不同，因为有的k-mer匹配之间能够重叠而有的不能

==这使得对于pattern出现概率的计算更加复杂==

> 因为概率取决于特定选择的pattern

所以做法是趋近而非精确计算概率

简化：不考虑pattern之间的重叠问题

并且将可能的组合看做是pattern插入到另外的随机序列中

> If we select exactly t of these occurrences, then we can think about Text as a sequence of n = N ! t · k symbols interrupted by t insertions of the k-mer Pattern.   

假设是在将两个“01”插入到222序列中，有一下情况：可以看做是在5个位置中选择2个位置，C（5，2）=10=5！/ 2! *3! 

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517110224530.png" alt="image-20220517110224530" style="zoom:80%;" />

这样就可以得到对应的pattern出现 t 次概率：在n+t个位置中选择t个位置 * 其他位置随机序列个数/ 所有位置随机序列总个数

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517110657192.png" alt="image-20220517110657192" style="zoom:80%;" />

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517110647914.png" alt="image-20220517110647914" style="zoom:80%;" />

而我们的目标是计算一个kmer在N 长的随机序列中出现至少 t 次的概率：Pr(N, A, k, t).  

==**感觉书中这里的推导存在问题，前面估计的是刚好出现 t 次的概率而不是至少出现 t 吃的概率**==

按照他的逻辑走下去

> 那么这时候一个kmer不出现 t 次及以上次数的概率为 1-p
>
> 所有kmer都不出现的概率为 （1-p）^A^^^k^
>
> 出现的概率则为
>
> <img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205171121970.png" alt="image-20220517112134890" style="zoom:80%;" />
>
> 这里的计算可能会导致round-off溢出
>
> ==所以化简计算：假设对所有的kmer p都相等==
>
> <img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517113218515.png" alt="image-20220517113218515" style="zoom:80%;" />

这个估计由于忽略了重叠的pattern，所以会偏小



## DNA链的方向

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517153148100.png" alt="image-20220517153148100" style="zoom: 67%;" />

5‘ 连接磷酸基团，3' 连接-OH， 1' 连接 含氮碱基

其中3' -OH 会与下一个核糖核苷酸的5' -磷酸基团连接

任何一个DNA链条的两端都分别是 5'- 磷酸基团 3'-OH

并且一般都是5' → 3' 的顺序表示

## 递归：汉诺塔问题

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517153716255.png" alt="image-20220517153716255" style="zoom:80%;" />

汉诺塔递推公式：需要将n-1 个dish先移动到中间的柱子，之后将最大的dish移动到最右边，再将n-1个柱子移动到最右边

所以有

<img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205171550337.png" alt="image-20220517155051268" style="zoom: 80%;" />

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517155506099.png" alt="image-20220517155506099" style="zoom:80%;" />



## 重叠词悖论

Best bet for simpletons 

>  However, an intriguing feature of Best Bet for Simpletons is that if k > 2, then Player 2 can always choose a k-mer B that beats A, regardless of Player 1’s choice of A.

相关多项式

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517160306308.png" alt="image-20220517160306308" style="zoom:80%;" />

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517160414187.png" alt="image-20220517160414187" style="zoom:80%;" />







# Cha2. 在DNA序列中找到执行功能的基序

## 问题导入

是否存在生物钟相关的基因？是否有相关的分子机制，比如用于解释一些疾病的发生时间？

现在的研究发现有少量的生物钟基因，并且这些基因能够调节大量的其他基因的表达，具有高度的进化保守性



最开始书里先介绍在植物中进行研究的例子

首先想下，在植物中有很多基因的表达调节需要依赖于太阳的升降：光合、光接收、开花

>  just three plant genes, called LCY, CCA1, and TOC1, are the clock’s master timekeepers  

这三个基因及其编码的调控蛋白一般受到外界因素的影响，从而使得生物体能够在不同环境中调整自身的基因表达情况；并且这三个基因之间存在负反馈的循环

这些基因转录之后的产物一般作为转录因子调控其他基因的表达，并且这些转录因子是结合在基因上游区域（600-1000bp）的调控基序上发挥作用的（regulatory motif）

在实际研究中发现，这些调控基序并不是保守的

> CCA1 binds to the AAAAAATCT   and also binds to AAGAACTCT instead.

==怎样在序列中定位这些基序的位置呢？==

使用MicroArray寻找所有会发生周期性表达变化的基因，并且这些基因的upsteam region出现AAAATATCT  的次数极高（46 / 500genes）

突变掉一个基因中这一序列之后，发现对应的基因不存在周期性的表达变化

==上述为发现一个保守的基序的例子==

发现保守的调控基序较为简单，但是具有许多突变的基序发现起来会更为困难



另外一个例子：果蝇中免疫相关基因的调控，NF-KappaB基因对应motif具有多个突变，下图展示了不同免疫基因上游结合NF-kappaB motif的序列情况，其中用彩色标出了每个位点上占多数的SNP，全部为12-mer

<img src="https://gitee.com/wangjx68/notebook_pic/raw/master/202205171646831.png" alt="image-20220517164633742" style="zoom:80%;" />

==同样的，现在的目标是将生物学问题描述转变为计算问题的描述==

## 问题描述

书中模拟了这样一件事情

> 将一个15-mer的序列插入到随机选择的10个DNA序列中去：模拟了在10个基因的上游区域存在有相同的一个转录因子结合位点

寻找这个插入的15-mer

> 将这个基因联结之后寻找在里面出现最多次数的15-mer即可，并且由于是随机产生的序列，所以理论上不存在其他的most frequent 15-mer

现在将这个问题转变为：插入的15-mer中，每次插入都随机选择4个位置进行突变后再插入

![image-20220517165617514](https://gitee.com/wangjx68/notebook_pic/raw/master/202205171656607.png)

在第一章中我们写得算法应用于这个15-mer的寻找来说太慢了：k过大，d过大；并且将10个序列联结起来并不符合我们寻找的motif的生物学意义

> 对于DnaA box来说：成簇出现
>
> 对于调控motif来说：至少出现一次，在基因组上是分散的

## 问题解决

### 暴力解法/ 穷举法

首先先考虑暴力解法：列举所有可能答案并一一验证

> 一定能得到最优解，但是时间消耗巨大

上述的问题可以表述为：在一组 Dna序列中寻找一个k-mer 以 d 编辑距离的形式出现在每一个序列中

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220517170152939.png" alt="image-20220517170152939" style="zoom:80%;" />

暴力解法的逻辑

> that any (k, d)-motif must be at most d mismatches apart from some k-mer appearing in one of the strings of Dna.   

所以就是将所有的邻域k-mer都打出来，然后看哪个邻域kmer在所有的string中都存在对应的d-kmer

暴力解法使用的时间过长，需要想出另一个办法化简算法

### 评分矩阵构建

我们思考，既然知道在所有的序列中都插入了对应的15mer，那么只需要将两个任意的string配对之后，寻找最相似的两个15mer即可获得对应的信息

当我们把这个想法用于我们的模拟数据中，我们发现事情没有那么简单

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220518112903384.png" alt="image-20220518112903384" style="zoom:80%;" />

可以看到虽然都是具有4个突变，但是两个序列kmer之间就会存在有8个突变，这使得配对寻找非常困难

> 在配对的比较中肯定会出现两个15mer之间存在的差异小于8

==整个章节的目的：寻找在10个随机产生的，每个600bp长度中插入的一个15mer，并且每次插入对应存在有4个突变==

并且在实际的研究中会存在这个一个问题：==在结果数据集中不一定所有基因的上游区域都存在有对应的调控motif（如microarray中存在噪声）==，对于这样个别序列不存在motif的数据集，我们的算法永远无法找到对应的motif

所以书中认为这个问题更恰当的描述是：对一个motif例子进行评分，衡量其与一个理想motif的相似程度

但是又因为这个理想的motif 事先不知道，所以我们的策略是在每条序列中选择1条kmer，然后评分评价其与其他kmer的相似程度

**接下来看这个评分的定义过程**

考虑 t 个DNA序列，每个序列长为 n，从每个DNA序列中选择一个k-mer组成一个Motifs矩阵（t × k）

在下面的图中使用图标大小表示一个位置上碱基出现的次数：越大代表这个位置的碱基越保守

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220518145828317.png" alt="image-20220518145828317" style="zoom:80%;" />

==通过不断的改变在每个序列中选择的kmer，会产生多种Motif矩阵组合，我们的目的是找到使得Consensus中upper letters数目最多的组合方式：也就是上图中Score（unpopular 字符出现次数总和）最小的情况==

为了计算这个Score，我们需要创建一个Count矩阵，记录每个位置上ACGT四个碱基出现的次数，进而得到Profile矩阵（记录出现频率）

最后可以产生对应更多Consensus序列，只要选择的Motif矩阵是正确的，那么对应的Consensus序列就是我们需要寻找的motif序列

**==对Score评分的讨论==**

并且我们知道motif中不同位置的序列保守程度不同

> 在上文中的第2列（6C2A2T）与第12列（6C4T）的位点保守程度不一
>
> 对应的Score评分都增加4

当我们将对应的出现频率为0.4以上的核苷酸也记录到motif 序列中，会获得如下的结果

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220518152016980.png" alt="image-20220518152016980" style="zoom:80%;" />

==使用熵 H 来衡量这些位点的核苷酸出现频率分布不确定性：取值范围为0~2，数值越小位点保守程度越高==

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220518152314777.png" alt="image-20220518152314777" style="zoom:80%;" />

> 实际计算中认为
>
> <img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220518152351182.png" alt="image-20220518152351182" style="zoom:80%;" />

**也就是说熵H 是比Score更好的一个评价标准，但是在本文后续仍然简单地将Score作为评价标准**

------

**==Motif Logo==**

上图中最后的图：相对大小表示在每列中的频率，总的高度对应为2-H，字符越大代表这个位点越保守

------

经过上面的论述，我们可以将motif finding问题进行进一步的陈述

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220518153657143.png" alt="image-20220518153657143" style="zoom:80%;" />

这个问题的暴力解法复杂度为 O(n^t^ · k · t)，所以需要寻找更快速的算法解决

> <img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220518153908731.png" alt="image-20220518153908731" style="zoom:80%;" />



前面的算法是通过组合不同的motif之后计算出对应的consensus，在组合的过程中会涉及到较大的计算量，如果我们反过来：假设一个可能的consensus，然后选取最可能的Motif计算score，可能可以减少操作次数

并且我们发现：在上文中我们计算Score的过程都是逐列计算，现在可以逐行计算以实现我们反向搜索的算法目的

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220518154741145.png" alt="image-20220518154741145" style="zoom:80%;" />

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220518154830342.png" alt="image-20220518154830342" style="zoom:80%;" />

这样一行一行计算的方式是我们可以得到

<img src="C:\Users\94732\AppData\Roaming\Typora\typora-user-images\image-20220518155158160.png" alt="image-20220518155158160" style="zoom:80%;" />

==这样我们就可以选择一个可能的pattern：然后从每个序列中找到一个kmer组成Motif，使得 d 总和最小==

> <img src="VOL%20%E2%85%A0.assets/image-20220518161624913.png" alt="image-20220518161624913" style="zoom:80%;" />

但是这更加复杂化了原问题：不仅仅要搜索所有的可能组合，还需要搜索所有的可能kmer

**但是基于一个观察可以缩小我们的算法复杂度：给定了pattern之后，我们并不要搜索所有可能的kmer组合**

> 在下面这个矩阵中我们发现
>
> <img src="VOL%20%E2%85%A0.assets/image-20220518162647641.png" alt="image-20220518162647641" style="zoom:80%;" />
>
> 我们可以在Dna的每条序列中都找到对应使得 d（pattern，kmer）最小的kmer
>
> 每个都取最小组合起来就是我们期望获得的最小d 总和

### Median String（Consensus String）：全kmer搜索

那么现在的目标就是寻找一个k-mer pattern，最小化d 总和：寻找median string

> <img src="VOL%20%E2%85%A0.assets/image-20220518163059996.png" alt="image-20220518163059996" style="zoom:80%;" />



==可以得到在重新定义了这个问题之后我们显著减少了实际中使用这一算法的操作数==

> <img src="VOL%20%E2%85%A0.assets/image-20220518172037733.png" alt="image-20220518172037733" style="zoom:80%;" />

==在问题的解决过程中，反向思维有时候能够创造更好的解决方式==：并且要尽可能减少输入数据的计算规模

------

继续优化这个问题的解决算法



### 贪婪算法

在每次循环中都选择最优的选项：启发式算法，以运算的精确程度的降低换来运算的快速

从上面的Profile矩阵中我们可以计算一个随机的序列是由Profile对应产生的概率

<img src="VOL%20%E2%85%A0.assets/image-20220518173952665.png" alt="image-20220518173952665" style="zoom:80%;" />

==这个概率越大，对应的序列就与Profile对应的Consensus序列越相似==

> 如果给定了Profile，我们就可以直接计算每个Text中的kmer出现的概率，并找到Profile矩阵最可能对应的kmer（相同的一般选取第一个）

贪婪算法：Dna1中的每个kmer作为起始kmer，然后构建对应的profile，接下来基于对应的profile选择Dna2中最可能的kmer选取最可能的kmer，再更新profile，最后得到Dna1对应选择的kmer不断最优选择的kmer矩阵作为Motif矩阵，最后选择所有Dna1-kmer中score最低的作为对应的贪婪解

> <img src="VOL%20%E2%85%A0.assets/image-20220519140208969.png" alt="image-20220519140208969" style="zoom:80%;" />

**对这个贪婪算法进行分析**

这个算法可以处理k=15的问题，但是最后的结果准确性较差

> <img src="VOL%20%E2%85%A0.assets/image-20220520152800343.png" alt="image-20220520152800343" style="zoom:80%;" />

现在来分析导致贪婪算法准确性差的原因：局部最优推不出全局最优

> 在选定了第一条插入motif之后，尽管假设我们选定的是正确的motif，也会导致根据最开始的profile计算出来的dna~2~中除非与dna~1~中选择的motif相同，其他的motif都为0
>
> 这样就会大大干扰我们选择motif的过程：因为算法在这种情况下只是选择最早出现的那个motif

也就是通过贪婪算法我们只能获得一个粗略的结果：除非改进profile的计算规则



文中使用 Cromwell's rules引入一个观点

> 出现是在描述只有0 1 两个状态的逻辑事件，否则对于其他事件的发生概率，我们一般不能使用0或1描述

==在小样本中或者是大样本中的小概率事件，一件发生概率不为0的事件没有发生==，但是不能简单的将该事件的发生概率描述为0，==需要人为的调整这一事件的概率以避免出现错误==



**拉普拉斯连续法则**

看到下面的例子

> 只在第4个位置与consensus 序列不同的序列会与具有较大差异的序列一样概率都为0：需要对这个问题进行人为的修正
>
> <img src="VOL%20%E2%85%A0.assets/image-20220520155532659.png" alt="image-20220520155532659" style="zoom:80%;" />
>
> 也就是使得对应的0 对于结果的影响不要是决定性的：适当增加小概率时间的发生概率

拉普拉斯修正：伪计数(pseudocounts )，在原有的频数基础上+1

> <img src="VOL%20%E2%85%A0.assets/image-20220520155853094.png" alt="image-20220520155853094" style="zoom:80%;" />

### 优化贪婪算法：pseudocounts

将Laplace’s Rule of Succession（pseudocounts）引入到贪婪算法中

优化贪婪算法在实际应用中的表现

> <img src="VOL%20%E2%85%A0.assets/image-20220520160929270.png" alt="image-20220520160929270" style="zoom:80%;" />

15mer中正确预测了10个位置

------



### 随机化 motif 搜索

随机化算法最早证明可以应用于实际问题的解决：18世纪数学家啊，Buffon’s Needle 使用随机打点的方式成功估计π

随机化算法一般是反直觉的：无法像传统算法一样进行精确的控制，可以分为

​	Las Vegas algorithms ：保证可以提供精确的解答

​	Monte Carlo algorithms  ：大部分随机算法，不保证可以获得精确的解答，但是能够快速寻找到大致的答案；==并且由于运行速度快，所以可以进行多次运行寻找最佳逼近==

现在模拟一下随机化搜索最佳motif的过程：

> 首先，还是有Dna Profile Motif的定义，定义Motif（Dna， Profile）是在Profile下每个Dna都选择最佳kmer组成的列表
>
> 算法一般从随机选择构成一个Motif矩阵开始，由这个Motif矩阵获得对应的Profile矩阵
>
> 之后再根据这个Profile矩阵选择最优的Motif，如此往复直到Score不再减小
>
> ==不断重复这个过程的目的是使得Score不断减小==
>
> 在算法最开始要根据生成的随机数对应产生随机的kmer组合Motif

==一般这种随机算法会独立运行上千次从而筛选出相对最佳结果==

> <img src="VOL%20%E2%85%A0.assets/image-20220520164241967.png" alt="image-20220520164241967" style="zoom:80%;" />

------

接下来作者在书中论证了随机搜索算法的有效性：以一个5序列Dna矩阵，(4,1)-motif ACGT插入为例

假设最开始随机挑选的矩阵为

<img src="VOL%20%E2%85%A0.assets/image-20220521145513368.png" alt="image-20220521145513368" style="zoom:80%;" />

根据初始的4mer矩阵构建profile

<img src="VOL%20%E2%85%A0.assets/image-20220521145607799.png" alt="image-20220521145607799" style="zoom:80%;" />

之后根据对应的Profile对每行进行计算，获得新的Motif

<img src="VOL%20%E2%85%A0.assets/image-20220521145840041.png" alt="image-20220521145840041" style="zoom:80%;" />

这时候我们就发现挑选出来的Motif恰好就是我们选择插入的序列

<img src="VOL%20%E2%85%A0.assets/image-20220521145907204.png" alt="image-20220521145907204" style="zoom:80%;" />

并且在实际搜索（15，4）插入的时候获得的分数也仅仅比优化后的贪婪算法稍微不保守而已：但是需要时间更短

而且在不断增加随机算法的运行次数之后期望是能够得到不断优化的结果：而优化后的贪婪算法并没有这一特性，这在实际研究中非常重要

==这里只是用两个例子举例说明，接下来书里进一步阐明随机算法在搜索上返回准确结果的理论基础==

如果Dna序列是随机的，那么就会导致我们计算出来的Profile任一条目都为0.25，但是由于存在对应插入的motif，这会使得这样的均一分布被打破，从而使得我们计算出来的profile朝向Motif的方向偏移

> 这些在Dna矩阵中不断出现的具有相似序列的motif会导致计算出来的profile逐渐向motif偏移
>
> 虽然随机挑选到所有插入kmer的几率很低，但是随机挑选到其中一条kmer的几率较高，这样在不断的随机挑选过程中肯定会存在有逼近到真实结果的结果出现

==但是如果开始选择的kmer位置十分多，那么不一定挑选到一个kmer就可以使得随机搜索算法找到最优解：而且找到最优解的概率较小==

### Gibbs抽样：谨慎/保守的随机化算法

在之前的随机化算法中，我们每次运行算法都是随机选取Motif，这样的更优解的来源是完全重新开始选择的随机Motif，而不是对现有的Motif进行优化，现在尝试使用一个更为谨慎的策略：每次循环考虑Motif中的一条序列，更新/ 保留

与之前策略相比：这是一个更随机的算法，之前的随机搜索策略是在随机选择起始序列之后就进行定向的更新直到Score不再下降

==而现在的抽样方法是在Motif中随机选择一个序列进行更新：这个序列的更新方法也是根据profile的概率随机挑选的==

> 使用Random函数进行更新Motif i 中 i 的选择
>
> <img src="VOL%20%E2%85%A0.assets/image-20220521154220984.png" alt="image-20220521154220984" style="zoom:80%;" />



注意在这里我的程序输出的最开始随机选取Motif及其评分与最后输出的最优Motif及其评分如下

~~~python
['GGGGGACTCGTTGAA', 'GGAAAAAACGGCGAG', 'GGGTCTTGCATTGGA', 'CTCGGGCGCTGTGCA', 'CGTCAGCAGGACGTA', 'GCTTAGCGACGCTTG', 'AAATAGTTATCCTAG', 'ATTCCCACTAGTTAT', 'GATACTTTATTAAGG', 'CATAGACCAAGCACG', 'TTCTACCTAGTTAGC', 'TAAGATCCGGCTTAA', 'TCGGACCTGTTAAAC', 'CTCGGTCGTGTCGGG', 'ATAGTATTACTATGG', 'TAATCACCAACGCGT', 'CTCACCTTCACTTAA', 'GGCGACTGGCTCCCT', 'GTTCCATAGTAGTTA', 'TAAGTTTTCGTGGGT'] 
189
['TCGTTGAAATTGACA', 'TCGATTTTAAGGACC', 'TCGTACTTATTTACC', 'TCGTACTTAAGGGTG', 'ACGTACTTAAGGATA', 'ACCTCCCTAGGGACC', 'TCGTATGAAAGGACC', 'TCGTACTTAATTCCC', 'TTCGACTTAAGGACC', 'TCGTATCAAAGGACC', 'TCGTACTTAAGCGAC', 'TCCGGCTTAAGGACC', 'TCGTACTATCGGACC', 'TCGTGTGTAAGGACC', 'TCGTCAATAAGGACC', 'TCGTACGAGAGGACC', 'TCGTACCACAGGACC', 'TCGTACTTGGTGACC', 'TCGTCCTTAGGCGCC', 'TCGACATTAAGGACC']
63
~~~

并且我在程序中进行Gibbs抽样的运行次数是伪代码中规定的10倍：以期获得尽可能的最优解

这时候意识到可能最开始随机起始对程序的影响较大，所以又独立运行了10次随机起始之后Gibbs 抽样的过程

但是在实际的Gibbs抽样中按照书中展示的过程，应该可以很好的cover到插入的motif中

- [x] 所以程序还在debug中，先往下走

注意到题目描述中有这句话

>  The strings *BestMotifs* resulting from running *GibbsSampler*(*Dna*, *k*, *t*, *N*) with 20 random starts. Remember to use pseudocounts!

==所以需要20个random start去跑这个东西==

==结果发现确实需要20个random start去跑这个东西==：以免结果只是局部最优

------

所以正如我在调试程序中遇到的，GibbsSampler极容易在搜索的过程中陷入局部最优中

> 这里的最优解指的是在一个较小的邻域解集中最优，而不是全部解集的最优
>
> 会导致局部最优的原因是我们的算法只是处理了所有解集中的一小部分，所以需要在实际应用中多次运行这一程序以期获得我们所需的全局最优解法

- [ ] ==另外，随机化算法还存在有其他缺陷==

## 后记：结核分枝杆菌如何休眠以躲避抗生素？

全世界有1/3的人感染结核分枝杆菌

但是结核分枝杆菌大部分会在人体内休眠，这使得对于结核病疫情的防控较为困难

生物学家想要研究结核分枝杆菌是怎样在宿主体内休眠并怎样激活自身的：孢子期，将绝大部分基因的表达显著降低并且进入休眠状态

> 缺氧环境容易导致结核分枝杆菌休眠：使得宿主肺部得以恢复，以进行病菌传播
>
> 并且该病菌可以在缺氧条件下生存多年

所以鉴定出使得病菌进入低氧环境休眠状态的基因十分重要

> 即鉴定出对应的感应环境缺氧的转录因子

2003年，发现DosR → DnaMicroArray发现被调控的基因→ 找到调控区域

然后我们使用这一章节中提到的不同程序去处理这个问题，会发现不同程序获得的结果我出入：==选择哪一种方法呢==

> 我的想法是通过不同的程序预测之后结合湿实验进行验证：双荧光素酶报告基因系统

最后的终极问题

> <img src="VOL%20%E2%85%A0.assets/image-20220523133410162.png" alt="image-20220523133410162" style="zoom:80%;" />
>
> 序列如下
>
> ~~~txt
> GCGCCCCGCCCGGACAGCCATGCGCTAACCCTGGCTTCGATGGCGCCGGCTCAGTTAGGGCCGGAAGTCCCCAATGTGGCAGACCTTTCGCCCCTGGCGGACGAATGACCCCAGTGGCCGGGACTTCAGGCCCTATCGGAGGGCTCCGGCGCGGTGGTCGGATTTGTCTGTGGAGGTTACACCCCAATCGCAAGGATGCATTATGACCAGCGAGCTGAGCCTGGTCGCCACTGGAAAGGGGAGCAACATC
> CCGATCGGCATCACTATCGGTCCTGCGGCCGCCCATAGCGCTATATCCGGCTGGTGAAATCAATTGACAACCTTCGACTTTGAGGTGGCCTACGGCGAGGACAAGCCAGGCAAGCCAGCTGCCTCAACGCGCGCCAGTACGGGTCCATCGACCCGCGGCCCACGGGTCAAACGACCCTAGTGTTCGCTACGACGTGGTCGTACCTTCGGCAGCAGATCAGCAATAGCACCCCGACTCGAGGAGGATCCCG
> ACCGTCGATGTGCCCGGTCGCGCCGCGTCCACCTCGGTCATCGACCCCACGATGAGGACGCCATCGGCCGCGACCAAGCCCCGTGAAACTCTGACGGCGTGCTGGCCGGGCTGCGGCACCTGATCACCTTAGGGCACTTGGGCCACCACAACGGGCCGCCGGTCTCGACAGTGGCCACCACCACACAGGTGACTTCCGGCGGGACGTAAGTCCCTAACGCGTCGTTCCGCACGCGGTTAGCTTTGCTGCC
> GGGTCAGGTATATTTATCGCACACTTGGGCACATGACACACAAGCGCCAGAATCCCGGACCGAACCGAGCACCGTGGGTGGGCAGCCTCCATACAGCGATGACCTGATCGATCATCGGCCAGGGCGCCGGGCTTCCAACCGTGGCCGTCTCAGTACCCAGCCTCATTGACCCTTCGACGCATCCACTGCGCGTAAGTCGGCTCAACCCTTTCAAACCGCTGGATTACCGACCGCAGAAAGGGGGCAGGAC
> GTAGGTCAAACCGGGTGTACATACCCGCTCAATCGCCCAGCACTTCGGGCAGATCACCGGGTTTCCCCGGTATCACCAATACTGCCACCAAACACAGCAGGCGGGAAGGGGCGAAAGTCCCTTATCCGACAATAAAACTTCGCTTGTTCGACGCCCGGTTCACCCGATATGCACGGCGCCCAGCCATTCGTGACCGACGTCCCCAGCCCCAAGGCCGAACGACCCTAGGAGCCACGAGCAATTCACAGCG
> CCGCTGGCGACGCTGTTCGCCGGCAGCGTGCGTGACGACTTCGAGCTGCCCGACTACACCTGGTGACCACCGCCGACGGGCACCTCTCCGCCAGGTAGGCACGGTTTGTCGCCGGCAATGTGACCTTTGGGCGCGGTCTTGAGGACCTTCGGCCCCACCCACGAGGCCGCCGCCGGCCGATCGTATGACGTGCAATGTACGCCATAGGGTGCGTGTTACGGCGATTACCTGAAGGCGGCGGTGGTCCGGA
> GGCCAACTGCACCGCGCTCTTGATGACATCGGTGGTCACCATGGTGTCCGGCATGATCAACCTCCGCTGTTCGATATCACCCCGATCTTTCTGAACGGCGGTTGGCAGACAACAGGGTCAATGGTCCCCAAGTGGATCACCGACGGGCGCGGACAAATGGCCCGCGCTTCGGGGACTTCTGTCCCTAGCCCTGGCCACGATGGGCTGGTCGGATCAAAGGCATCCGTTTCCATCGATTAGGAGGCATCAA
> GTACATGTCCAGAGCGAGCCTCAGCTTCTGCGCAGCGACGGAAACTGCCACACTCAAAGCCTACTGGGCGCACGTGTGGCAACGAGTCGATCCACACGAAATGCCGCCGTTGGGCCGCGGACTAGCCGAATTTTCCGGGTGGTGACACAGCCCACATTTGGCATGGGACTTTCGGCCCTGTCCGCGTCCGTGTCGGCCAGACAAGCTTTGGGCATTGGCCACAATCGGGCCACAATCGAAAGCCGAGCAG
> GGCAGCTGTCGGCAACTGTAAGCCATTTCTGGGACTTTGCTGTGAAAAGCTGGGCGATGGTTGTGGACCTGGACGAGCCACCCGTGCGATAGGTGAGATTCATTCTCGCCCTGACGGGTTGCGTCTGTCATCGGTCGATAAGGACTAACGGCCCTCAGGTGGGGACCAACGCCCCTGGGAGATAGCGGTCCCCGCCAGTAACGTACCGCTGAACCGACGGGATGTATCCGCCCCAGCGAAGGAGACGGCG
> TCAGCACCATGACCGCCTGGCCACCAATCGCCCGTAACAAGCGGGACGTCCGCGACGACGCGTGCGCTAGCGCCGTGGCGGTGACAACGACCAGATATGGTCCGAGCACGCGGGCGAACCTCGTGTTCTGGCCTCGGCCAGTTGTGTAGAGCTCATCGCTGTCATCGAGCGATATCCGACCACTGATCCAAGTCGGGGGCTCTGGGGACCGAAGTCCCCGGGCTCGGAGCTATCGGACCTCACGATCACC
> ~~~

# Cha2. 补充

## Median String问题解决

书中展现了计算每个pattern到DNA中所有dna序列的d的算法

我在实际的计算中没有使用数字与pattern互换的做法：而是直接创建了对应的kmer列表，itertools.product

节省的存储和时间需要进行比较计算

## 基因表达

基因表达在不同环境下不同

细胞通过中心法则对表达进行调控

转录开始：RNA聚合酶结合到启动子序列，最为简便的控制节点（基因表达的最开始）

可以通过多种调控因子进行调控

## DNA微阵列

固定有设计好的探针的固体表面，探针会与靶标DNA结合，结合的DNA上具有用于检测的荧光标记，荧光标记的强度与靶标DNA的表达量成正比

## 布冯针问题

<img src="VOL%20%E2%85%A0.assets/image-20220523140620642.png" alt="image-20220523140620642" style="zoom:80%;" />

抛掷一枚硬币，当其完全落于棋盘中的一个方格内则记为1，否则记为0，问期望

假设落点均一分布，并且棋盘上只有右图所示的一个方块，那么可以得到期望为：（1-2r）^2^

这对于具有多个方块的棋盘也适用

**更新问题描述：现在是对一个竖直方向上为木长条的棋盘进行投掷**，buffon needle

<img src="VOL%20%E2%85%A0.assets/image-20220523141452654.png" alt="image-20220523141452654" style="zoom: 67%;" />

也是同样的条件与问题：这时候就涉及到位置与向量的关系，还是先确定圆心的位置

解决这个问题首先处理下落在特定位置取到的01概率：使用坐标平面进行标记，并且y轴作为panel的中线，假定宽度与直径都为2

<img src="VOL%20%E2%85%A0.assets/image-20220523143541734.png" alt="image-20220523143541734" style="zoom:80%;" />

<img src="VOL%20%E2%85%A0.assets/image-20220523143639227.png" alt="image-20220523143639227" style="zoom:80%;" />

## :triangular_flag_on_post: Motif Finding问题的复杂性

具有较大偏向性的序列也会导致我们的算法导向一个不具有生物学意义的motif

<img src="VOL%20%E2%85%A0.assets/image-20220523144703260.png" alt="image-20220523144703260" style="zoom:80%;" />

需要使用相对熵

另外一个问题：存在多义位点，并且我们在这章节中使用的方法无法确定多义位点（概率都极小）

<img src="VOL%20%E2%85%A0.assets/image-20220523144940997.png" alt="image-20220523144940997" style="zoom:80%;" />

在实际应用中优化是使得相对熵最大



# Cha3. 基因组组装问题

## 问题引入

报纸问题：收集100份某天的环球时报，然后捆在一起炸开成碎片，问怎么从碎片中还原出当前的报纸

> 这是比拼图更难的问题：具有多份重复，并且在爆炸的过程中肯定会导致部分信息的丢失
>
> 所以需要通过不同碎片中的重叠信息重建当天的新闻

基因测序问题

> 现在的技术仍然无法将基因组从头到尾测序，所以需要将基因组切碎为小片段之后进行测序（得到reads）之后再组装
>
> 我们无法标记reads在基因组中的来源，所以需要使用reads的重叠信息进行组装

基因组装问题：与报纸问题类似

> 组装问题成为现在基因组测序的主要阻碍：已经能够产生足够的数据

<img src="VOL%20%E2%85%A0.assets/image-20220523151616241.png" alt="image-20220523151616241" style="zoom:80%;" />

## 问题描述：序列重构问题

基因组组装问题的难点

1. 基因组序列为双链：所以在使用reads时无法确定是使用reads还是使用其反向互补序列
2. 测序技术存在缺陷，会产生少量测序错误，从而导致重叠信息检测可能出现问题
3. 部分基因组区域可能无法进行测序，无reads覆盖，从而使得无法对全基因组进行测序

首先先考虑所有的reads来源于同一条链，没有错误，具有完美覆盖程度：之后在考虑怎样放低假设要求

### 从k-mers中重建strings

先产生一个序列的字母表排序kmer集合

<img src="VOL%20%E2%85%A0.assets/image-20220523152432355.png" alt="image-20220523152432355" style="zoom: 67%;" />

那么在组装问题中我们需要考虑的是这个问题的反向问题

> 给定k与kmer列表，返回一个具有该kmer列表的序列
>
> <img src="VOL%20%E2%85%A0.assets/image-20220523153240775.png" alt="image-20220523153240775" style="zoom: 67%;" />

在书中首先给了一个可以简单的连接k-1 重叠kmer直到完成组装的例子

<img src="VOL%20%E2%85%A0.assets/image-20220523153426755.png" alt="image-20220523153426755" style="zoom:67%;" />

之后给了一个例子：**无法使用全部的kmer进行string的构建**

在这个例子中会导致这一现象的原因是其中一个kmer出现了三次，那么在选择连接的kmer时则有三种可能，所以给拼接问题带来复杂度

> 在具有多个kmer的数据集中（多个reads）这一问题影响巨大
>
> 而在人类基因组中存在这样具有高重复度的序列：Alu sequence
>
> e.g., the approximately 300 nucleotide-long Alu sequence is repeated over a million times, with only a few nucleotides inserted/deleted/substituted each time   

## 问题解决：重叠图问题

### 将序列转换为图

> 上述重复序列问题使得需要在组装中“向前看”，试着将kmer整理成图的形式
>
> <img src="VOL%20%E2%85%A0.assets/image-20220523154431988.png" alt="image-20220523154431988" style="zoom:80%;" />

从图得到序列十分简单，但是得到对应的图需要我们事先知道对应的图

### 重叠图问题

接下来的描述中使用前缀和后缀分别指kmer的前k-1和后k-1个字符：在path上相连的两个节点后缀与前缀相等

> 所以得到构建path的方法：将前缀后缀相等的kmer连接
>
> <img src="VOL%20%E2%85%A0.assets/image-20220523155601372.png" alt="image-20220523155601372" style="zoom:80%;" />
>
> 但是在事先不知道genome组成的情况下我们得到的path是混乱的有向图

当我们将genome产生的kmer按照字母表顺序排列：模拟事先不知道genome序列的情况，这时候会难以直观看到genome

> <img src="VOL%20%E2%85%A0.assets/image-20220523161002746.png" alt="image-20220523161002746" style="zoom:67%;" />

但是我们发现：genome的路线是存在的，**==当我们找到一条path途径每一个节点一次，则为我们需要的genome路径==**

> 并且这样的路径可能不止一条
>
> <img src="VOL%20%E2%85%A0.assets/image-20220523163015248.png" alt="image-20220523163015248" style="zoom: 67%;" />

简单介绍一下图的存储形式：连接矩阵/ 连接表

> <img src="VOL%20%E2%85%A0.assets/image-20220523163427836.png" alt="image-20220523163427836" style="zoom:67%;" />
>
> 连接矩阵的定义：For a directed graph with n nodes, the n ⇥ n adjacency matrix (Ai,j) is defined by the following rule: Ai,j = 1 if a directed edge connects node i to node j, and Ai,j = 0 otherwise.  
>
> 连接表：节省存储空间

### 哈密顿路径和通用序列

重构序列问题在上文的描述中已经被转化为了寻找一条路径，通过每个节点并且只通过节点一次——==哈密顿路径==

并且每个重叠图可能具有不止一条哈密顿路径

<img src="VOL%20%E2%85%A0.assets/image-20220523164152637.png" alt="image-20220523164152637" style="zoom:67%;" />

讨论de Bruijn

> 解决一个问题寻找k-universal string：a binary string is k-universal if it contains every binary k-mer exactly once  
>
> <img src="VOL%20%E2%85%A0.assets/image-20220523164641458.png" alt="image-20220523164641458" style="zoom: 67%;" />
>
> 如果我们上面讨论的kmer合集为所有的binary kmer，那么寻找对应的k-universal string类似做序列重构
>
> de Bruijn 希望能够找到对于任意k值，接触k-universal string 的方法：当k较大的时候，会存在极多的节点（2^k^），寻找哈密顿路径的复杂度极高

###  De Bruijn图与节点黏合

另外一种图的表示方法：在前面我们使用kmer作为节点，现在我们使用kmer作为边，前缀和后缀作为节点——节点表示连接的边上序列的重叠序列

> <img src="VOL%20%E2%85%A0.assets/image-20220523165550588.png" alt="image-20220523165550588" style="zoom:67%;" />

当开始进行节点黏合（合并）

> 1. 第一行：将三个相同的AT节点进行合并
> 2. 第二行：将三个相同的TG节点进行合并
> 3. 第三行：将三个相同的GG节点进行合并
>
> <img src="VOL%20%E2%85%A0.assets/image-20220523170524088.png" alt="image-20220523170524088" style="zoom:50%;" />
>
> 这样从最开始的16个节点，我们将图缩减为11个节点：得到de Bruijn图
>
> 并且我们可以通过AT TG节点之间存在3条连线保留ATG拷贝为3的信息

对于de Bruijn图的一般定义

> 产生一般图之后黏合节点
>
> <img src="VOL%20%E2%85%A0.assets/image-20220523171330706.png" alt="image-20220523171330706" style="zoom: 80%;" />

好了，现在我们可以根据text来构建de Bruijn 图，那么需要反过来处理问题

==从de Bruijn图构建text==

### 欧拉路径问题

寻找通过de Bruijn图中的每条边一次的路径













































































