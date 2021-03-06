### 概述

一致性hash算法广泛用于缓存服务的路由算法，对于伸缩性分布式缓存架构或数据库迁移，一致性hash有着较好的命中率

#### 1. 余数hash

余数hash的主要思路就是用hash值对服务器数量取余来决定命中那一台服务器，如果不考虑集群的伸缩性，这种算法可以满足大部分需求，但是如果分布式集群需要扩容的时候，就会导致原先的请求无法命中原来的服务器。

如果采用余数hash，那么只有使用rehash，让数据重新分布在新的集群分布中，这样耗时耗力，成本很高

#### 2.一致性hash算法

1). 算法流程：

先构造一个长度为232的整数环（这个环被称为一致性Hash环），根据节点名称的Hash值（其分布为[0, 2^32-1]）将缓存服务器节点放置在这个Hash环上，然后根据需要缓存的数据的Key值计算得到其Hash值（其分布也为[0, 
2^32-1]），然后在Hash环上顺时针查找距离这个Key值的Hash值最近的服务器节点，完成Key到服务器的映射查找。

如果在hash环上新增一个接口，影响的只有hash环上的一小段，并且整个集群下的节点越多，影响的范围越小

![](../images/uniformity_hash.png)


### 实现


#### 1. 确定数据结构
一致性hash算法的数据结构需要构造出一个2^32次方的整数环，我们需要一个尽可能低的时间复杂度的查找算法， 显然树形结构的时间复杂度更低， 所以一般选择 **二叉查找树**, 
避免出现树退化成线性结构，不能直接使用二叉查找树，而是使用红黑树(或平衡二叉树AVL),JDK提供了红黑树的实现：TreeMap和TreeSet

#### 2.Hash值的重新计算

String的hashCode()方法的分散性很差，一段连续的ip分布区间比较固定(所有的节点会分布在一段很小的区间)，这样会导致绝大部分请求都落在第一个节点上(顺时针找到第一个节点)

所以，String重写的hashCode()方法在一致性Hash算法中没有任何实用价值，得找个算法重新计算HashCode。这种重新计算Hash值的算法有很多，比如CRC32_HASH、FNV1_32_HASH、KETAMA_HASH等，其中KETAMA_HASH是默认的MemCache推荐的一致性Hash算法，用别的Hash算法也可以，比如FNV1_32_HASH算法的计算效率就会高一些。

#### 3. 不带虚拟节点的实现

使用FNV1_32_HASH来实现hash

- [不带虚拟节点的实现]()

#### 4. 使用虚拟节点来改善一致性Hash算法

### 参考资料
- [memcache详细解读](http://www.cnblogs.com/xrq730/p/4948707.html)
- [一致性hash的java实现](https://www.cnblogs.com/xrq730/p/5186728.html)