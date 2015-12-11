title: MurmurHash
date: 2015-09-03 14:55:54
tags:
category: Hash
---

　　MurmurHash 是一种非加密型哈希函数，适用于一般的哈希检索操作。由Austin Appleby在2008年发明，并出现了多个变种，都已经发布到了公有领域(public domain)。与其它流行的哈希函数相比，对于规律性较强的key，MurmurHash的随机分布特征表现更良好。

### 变种

　　当前的版本是MurmurHash3， 能够产生出32-bit或128-bit哈希值。较早的MurmurHash2能产生32-bit或64-bit哈希值。对于大端存储和强制对齐的硬件环境有一个较慢的MurmurHash2可以用。MurmurHash2A 变种增加了Merkle–Damgård 构造，所以能够以增量方式调用。 有两个变种产生64-bit哈希值：MurmurHash64A，为64位处理器做了优化；MurmurHash64B，为32位处理器做了优化。MurmurHash2-160用于产生160-bit 哈希值，而MurmurHash1已经不再使用。

### 实现

　　最初的实现是C++的，但是被移植到了其他的流行语言上，包括 Python, C,C#, Perl, Ruby, PHP,Haskell,、Scala、Java和JavaScript等。这个算法已经被若干开源计划所采纳，最重要的有libstdc++ (4.6版)、Perl、nginx (不早于1.0.1版)、Rubinius、 libmemcached (Memcached的C语言客户端驱动)、maatkit、Hadoop、Kyoto Cabinet以及RaptorDB。

A sample C implementation follows:
```C
uint32_t murmur3_32(const char *key, uint32_t len, uint32_t seed) {
	static const uint32_t c1 = 0xcc9e2d51;
	static const uint32_t c2 = 0x1b873593;
	static const uint32_t r1 = 15;
	static const uint32_t r2 = 13;
	static const uint32_t m = 5;
	static const uint32_t n = 0xe6546b64;
 
	uint32_t hash = seed;
 
	const int nblocks = len / 4;
	const uint32_t *blocks = (const uint32_t *) key;
	int i;
	for (i = 0; i < nblocks; i++) {
		uint32_t k = blocks[i];
		k *= c1;
		k = (k << r1) | (k >> (32 - r1));
		k *= c2;
 
		hash ^= k;
		hash = ((hash << r2) | (hash >> (32 - r2))) * m + n;
	}
 
	const uint8_t *tail = (const uint8_t *) (key + nblocks * 4);
	uint32_t k1 = 0;
 
	switch (len & 3) {
	case 3:
		k1 ^= tail[2] << 16;
	case 2:
		k1 ^= tail[1] << 8;
	case 1:
		k1 ^= tail[0];
 
		k1 *= c1;
		k1 = (k1 << r1) | (k1 >> (32 - r1));
		k1 *= c2;
		hash ^= k1;
	}
 
	hash ^= len;
	hash ^= (hash >> 16);
	hash *= 0x85ebca6b;
	hash ^= (hash >> 13);
	hash *= 0xc2b2ae35;
	hash ^= (hash >> 16);
 
	return hash;
}
```

### 相关资源：
1. http://blog.csdn.net/yfkiss/article/details/7337382
2. https://zh.wikipedia.org/wiki/Murmur%E5%93%88%E5%B8%8C
3. https://en.wikipedia.org/wiki/MurmurHash
4. https://github.com/lijingpeng/java_util/tree/master/src/util/hash
5. https://github.com/huichen/murmur/blob/master/murmur.go
6. https://pypi.python.org/pypi/mmh3/2.0
