---
layout: post
title: "Hash Algorithms"
date: 2017-02-26 18:15:06 
description: "Hash Algorithms"
tag: Algorithm
---

### 1. String Hash
- BKDRHash
- APHash
- Murmur2
- CityHash
- MD5 (Message Digest)
- SHA1(Secure Hash Algorithm)
- SHA125

实现：

	// BKDR Hash Function
	/* 核心的思想是通过一个选择器，我们也成为种子(seed)，
	 * 通常选择种子应该是一个质数，如31， 131，1313等，
	 * 这样的好处在于减少不同字符串映射为相同整数的冲突。*/
	unsigned int BKDRHash(char *str)
	{
	    unsigned int seed = 131; // 31 131 1313 13131 131313 etc..
	    unsigned int hash = 0;
	    while (*str) {
	        hash = hash * seed + (*str++);
	    }
	    return (hash & 0x7FFFFFFF);
	}

	// AP Hash Function
	unsigned int APHash(char *str)
	{
	    unsigned int hash = 0;
	    int i;
	    for (i=0; *str; i++) {
	        if ((i & 1) == 0) {
	            hash ^= ((hash << 7) ^ (*str++) ^ (hash >> 3));
	        }
	        else {
	            hash ^= (~((hash << 11) ^ (*str++) ^ (hash >> 5)));
	        }
	    }
	    return (hash & 0x7FFFFFFF);
	}




### 2. int Hash
黄金分割比素数(Golden ratio prime)常用于哈希(Knuth,第三卷,排序与搜索,1973).

	/* 2^31 + 2^29 - 2^25 + 2^22 - 2^19 - 2^16 + 1 */
	#define GOLDEN_RATIO_PRIME_32       0x9e370001UL
	
	/* *
	 * hash32 - generate a hash value in the range [0, 2^@bits - 1]
	 * @val:    the input value
	 * @bits:   the number of bits in a return value
	 *
	 * High bits are more random, so we use them.
	 * */
	uint32_t
	hash32(uint32_t val, unsigned int bits) {
	    uint32_t hash = val * GOLDEN_RATIO_PRIME_32;
	    return (hash >> (32 - bits));
	}

hash的方式是，让key乘以一个大数，于是结果溢出，就把留在32位变量中的值作为hash值，又由于散列表的索引长度有限，我们就取这hash值的高几为作为索引值，之所以取高几位，是因为高位的数更具有随机性，能够减少冲突。那么，乘以的这个大数应该是多少呢？从上面的代码来看，32位系统中这个数是0x9e370001UL，64位系统中这个数是0x9e37fffffffc0001UL。这个数是怎么得到的呢？ Knuth建议，要得到满意的结果，对于32位机器，2^32做黄金分割，这个大树是最接近黄金分割点的素数，0x9e370001UL就是接近 2^32*(sqrt(5)-1)/2 的一个素数，且这个数可以很方便地通过加运算和位移运算得到，因为它等于2^31 + 2^29 - 2^25 + 2^22 - 2^19 - 2^16 + 1。对于64位系统，这个数是0x9e37fffffffc0001UL，同样有2^63 + 2^61 - 2^57 + 2^54 - 2^51 - 2^18 + 1。


### 3. 一致性hash
普通的哈希算法（也称硬哈希）采用简单取模的方式，将机器进行散列，这在cache环境不变的情况下能取得让人满意的结果，但是当cache环境动态变化时，这种静态取模的方式显然就不满足单调性的要求（当增加或减少一台机子时，几乎所有的存储内容都要被重新散列到别的缓冲区中）。

在分布式环境中，评价hash算法的四个方面：

1、平衡性(Balance)：平衡性是指哈希的结果能够尽可能分布到所有的缓冲中去，这样可以使得所有的缓冲空间都得到利用。很多哈希算法都能够满足这一条件。
2、单调性(Monotonicity)：单调性是指如果已经有一些内容通过哈希分派到了相应的缓冲中，又有新的缓冲加入到系统中。哈希的结果应能够保证原有已分配的内容可以被映射到原有的或者新的缓冲中去，而不会被映射到旧的缓冲集合中的其他缓冲区。 
3、分散性(Spread)：在分布式环境中，终端有可能看不到所有的缓冲，而是只能看到其中的一部分。当终端希望通过哈希过程将内容映射到缓冲上时，由于不同终端所见的缓冲范围有可能不同，从而导致哈希的结果不一致，最终的结果是相同的内容被不同的终端映射到不同的缓冲区中。这种情况显然是应该避免的，因为它导致相同内容被存储到不同缓冲中去，降低了系统存储的效率。分散性的定义就是上述情况发生的严重程度。好的哈希算法应能够尽量避免不一致的情况发生，也就是尽量降低分散性。 
4、负载(Load)：负载问题实际上是从另一个角度看待分散性问题。既然不同的终端可能将相同的内容映射到不同的缓冲区中，那么对于一个特定的缓冲区而言，也可能被不同的用户映射为不同 的内容。与分散性一样，这种情况也是应当避免的，因此好的哈希算法应能够尽量降低缓冲的负荷。


对于分布式环境中，增删节点，需要保证单调性和平衡性：

- 单调性：通过调整节点所属的hash值，达到增删的单调性。
- 平衡性：通过引入虚节点解决。相当于进行二级映射：第一级hashkey映射到虚节点，第二级再映射到正式的物理节点。



### References

[1] [Hash表算法](http://blog.csdn.net/v_july_v/article/details/6256463),csdn.net.

[2] [Hash & Rabin-Karp字符串查找算法](http://novoland.github.io/%E7%AE%97%E6%B3%95/2014/07/26/Hash%20&%20Rabin-Karp%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%9F%A5%E6%89%BE%E7%AE%97%E6%B3%95.html),github.io.

[3] http://sfsrealm.hopto.org/inside_mopaq/chapter2.htm

[4] [各种字符串Hash函数比较](https://www.byvoid.com/zhs/blog/string-hash-compare),byvoid.com.

[5] [一致性哈希算法](http://blog.csdn.net/cywosp/article/details/23397179),csdn.net.

[6] [一致性hash算法简介](http://blog.huanghao.me/?p=14),huanghao.me.

[7] [Consistent hashing](https://www.codeproject.com/Articles/56138/Consistent-hashing),codeproject.com.

[8] [一致性哈希算法的理解与实践](http://yikun.github.io/2016/06/09/%E4%B8%80%E8%87%B4%E6%80%A7%E5%93%88%E5%B8%8C%E7%AE%97%E6%B3%95%E7%9A%84%E7%90%86%E8%A7%A3%E4%B8%8E%E5%AE%9E%E8%B7%B5/),github.io.

[9] [consistent hash原理，优化及实现](http://oserror.com/distributed/consistent-hashing/),oserror.com.