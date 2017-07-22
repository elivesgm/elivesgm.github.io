---
layout: post
title: "Hash Algorithms"
date: 2017-02-26 18:15:06 
description: "Hash Algorithms"
tag: Algorithm
---

### String Hash

1. Hash表算法, http://blog.csdn.net/v_july_v/article/details/6256463
2. Hash & Rabin-Karp字符串查找算法, http://novoland.github.io/%E7%AE%97%E6%B3%95/2014/07/26/Hash%20&%20Rabin-Karp%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%9F%A5%E6%89%BE%E7%AE%97%E6%B3%95.html
3. http://sfsrealm.hopto.org/inside_mopaq/chapter2.htm

### int Hash
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