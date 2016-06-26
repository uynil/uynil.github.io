---
layout: post
title: Bloom filter 布隆过滤
tags: [爬虫]
---
1. Bloom Filter 概述
2. 算法概述
3. 参数选择

#### 1. 实例
Bloom Filter 在爬虫应用广泛。由于网络错综复杂，蜘蛛爬行很有可能会闭环。为了避免这种可能，需要知道哪些URL是已经访问的。直观地想Bloom Filter 是判重算法中比较高效的一种。

一个URL判重问题的特点是， 『允许小概率的出错，不一定要100%准确！也就是说少量url实际上没有没网络蜘蛛访问，而将它们错判为已访问的代价是很小的——大不了少抓几个网页呗』。

#### 2. Bloom Filter 算法
为了降低出错的概率，Bloom Filter 建一个Bitset, 引入了多个哈希函数，而不是一个。

算法具体如下：

创建一个m位BitSet, 把所有位初始为0。选个K个不同的哈希函数，标记哈希位对应的值为一。第i个哈希函数对字符串str哈希的结果为BF(i, str), 0 < SF(i, str) <= m-1。 细分方法共分两步：

1. 判断字符串
对于字符串str 分别计算 BF(0, str), BF(1, str), BF(2, str) ... BF(k-1, str)；然后把BitSet的BF(0, str), BF(1, str), BF(2, str) ... BF(k-1, str)位标记为1。
这样就把str 映射到 BitSet的K个值中。

2. 检查字符串str是否存在

"""
　　对于字符串str，分别计算h（1，str），h（2，str）…… h（k，str）。然后检查BitSet的第h（1，str）、h（2，str）…… h（k，str）位是否为1，若其中任何一位不为1则可以判定str一定没有被记录过。若全部位都是1，则“认为”字符串str存在。

　　若一个字符串对应的Bit不全为1，则可以肯定该字符串一定没有被Bloom Filter记录过。（这是显然的，因为字符串被记录过，其对应的二进制位肯定全部被设为1了）

　　但是若一个字符串对应的Bit全为1，实际上是不能100%的肯定该字符串被Bloom Filter记录过的。（因为有可能该字符串的所有位都刚好是被其他字符串所对应）这种将该字符串划分错的情况，称为false positive 。
"""

(3) 删除字符串过程 
"""
   字符串加入了就被不能删除了，因为删除会影响到其他字符串。实在需要删除字符串的可以使用Counting bloomfilter(CBF)，这是一种基本Bloom Filter的变体，CBF将基本Bloom Filter每一个Bit改为一个计数器，这样就可以实现删除字符串的功能了。

　　Bloom Filter跟单哈希函数Bit-Map不同之处在于：Bloom Filter使用了k个哈希函数，每个字符串跟k个bit对应。从而降低了冲突的概率。
"""
#### 3.  Bloom Filter参数选择

"""
   (1)哈希函数选择

   　　哈希函数的选择对性能的影响应该是很大的，一个好的哈希函数要能近似等概率的将字符串映射到各个Bit。选择k个不同的哈希函数比较麻烦，一 种简单的方法是选择一个哈希函数，然后送入k个不同的参数。

    在 Bloom Filter Tutorial的关于哈希函数的章节中,提到：
        The hash functions used in a Bloom filter should be independent and uniformly distributed. They should also be as fast as possible (cryptographic hashes such as sha1, though widely used therefore are not very good choices).

        Examples of fast, simple hashes that are independent enough3 include murmur, the fnv series of hashes, and Jenkins Hashes.

        In a short survey of bloom filter implementations:
        Cassandra uses Murmur hashes
        Hadoop includes default implementations of Jenkins and Murmur hashes
        python-bloomfilter uses cryptographic hashes
        Plan9 uses a simple hash as proposed in Mitzenmacher 2005
        Sdroege Bloom filter uses fnv1a (included just because I wanted to show one that uses fnv.)
        Squid uses MD5

   (2)Bit数组大小选择 

   　　哈希函数个数k、位数组大小m、加入的字符串数量n的关系可以参考 [参考文献1](http://pages.cs.wisc.edu/~cao/papers/summary-cache/node8.html)。该文献证明了对于给定的m、n，当 k = ln(2)* m/n 时出错的概率是最小的。

   　　同时该文献还给出特定的k，m，n的出错概率。例如：根据参考文献1，哈希函数个数k取10，位数组大小m设为字符串个数n的20倍时，false positive发生的概率是0.0000889 ，这个概率基本能满足网络爬虫的需求了。  

"""

### 参考文章
1. [BloomFilter——大规模数据处理利器](http://www.cnblogs.com/heaad/archive/2011/01/02/1924195.html)
1. [BloomFilter——搜狗百科](http://baike.sogou.com/v63706797.htm?fromTitle=bloom+filter)
3. [BloomFilter Tutorial](http://billmill.org/bloomfilter-tutorial/)

