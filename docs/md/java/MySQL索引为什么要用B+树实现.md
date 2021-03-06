---
layout: post
title: MySQL索引为什么要用B+树实现？
lock: need
---

# 面试官：MySQL索引为什么要用B+树实现？

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200901232617212.png?)

## 原因如下
1. B+树能显著减少IO次数，提高效率
2. B+树的查询效率更加稳定，因为数据放在叶子节点
3. B+树能提高范围查询的效率，因为叶子节点指向下一个叶子节点
## B+树是怎么来的？
在从一堆数据中查找指定的数据时，我们常用的数据结构是哈希表和二叉查找树，表本质上就是一堆数据的集合，所以MySQL数据库用了哈希表和B+树来实现索引

B+树是通过二叉查找树，再由平衡二叉树，B树（又名B-树）演化而来的，B+树中的B不是代表二叉（binary），而是代表平衡（balance），因为B+树是从最早的平衡二叉树演化而来，但是B+树不是一个二叉树。

## 二叉查找树和平衡二叉树
二叉查找树的效率和平衡二叉树的查找效率已经很高了，为什么不用这两种数据结构来实现索引呢？慢慢来分析

二叉查找树是带有特殊属性的二叉树，需要满足以下属性

 1. 非叶子节点最多拥有两个子节点
 2. 非叶子节值大于左边子节点、小于右边子节点
 3. 没有值相等重复的节点;

![在这里插入图片描述](https://img-blog.csdnimg.cn/a771953a52924653b6d88802419f4827.png)

对上图这个二叉树进行查找，如查键值为5的记录，先找到根，其值时6，大于5，查找6的左子树，找到3，5大于3，再找其右子树，一共找了3次。同理，查找键值为8的记录，用了3次。所有键值平均查找次数为(1+2+2+3+3+3)/6=2.3次，假如对这些键值进行顺序查找，平均查找次数为(1+2+3+4+5+6)/6=3.3（查找顺序摆放的数，第一个数肯定是1次，而第2个数是2次，以此类推），显然二叉查找树的平均查找速度比顺序查找更快

二叉查找树可以任意的构造，假如二叉查找树按照如下方式构造

![在这里插入图片描述](https://img-blog.csdnimg.cn/44ea953c53944541bdc43289b246e7a5.png)

平均查找速度为(1+2+3+4+5+5)/6=3.16次，和顺序查找差不多。为了提高二叉查找树的查询效率，需要二叉查找数是平衡的，这就引出了平衡二叉树。

平衡二叉树除了满足上面3个属性，还要满足如下1个属性，**树的左右两边的层级数相差不会大于1**

平衡二叉树的查找效率确实很快，但维护一颗平衡二叉树的代价是非常大的，需要1次或多次左旋和右旋来得到插入或更新后树的平衡性。简单举个例子。

初始平衡二叉树

![在这里插入图片描述](https://img-blog.csdnimg.cn/a22a3dfcdab4444da6a2dd2c21968e84.png)

插入3

![在这里插入图片描述](https://img-blog.csdnimg.cn/f01e4508ab5447c3941f453f3ffd5e98.png)

右旋一次

![在这里插入图片描述](https://img-blog.csdnimg.cn/fe5bd645aad2497b98b0ec6b566cfaa8.png)

再左旋一次

![在这里插入图片描述](https://img-blog.csdnimg.cn/75a811827f844792aa9bda45bd4b222b.png)

作为一个科普性的文章，这里不对左旋的右旋的细节进行分析，放几个图片能理解左旋和右旋即可

对y进行右旋，意味着将y变为一个右节点

![在这里插入图片描述](https://img-blog.csdnimg.cn/46bcaf4cf8b64281a772707562ff3d03.png)

对x进行左旋，意味着将x变为一个左结点

![在这里插入图片描述](https://img-blog.csdnimg.cn/2f921c8793ae4bdfb747c3718c347e35.png)

回头看上面例子的左旋和右旋，是不是很清楚了？
## B树和B+树
B树和B-树是同一种树，假如用平衡二叉树实现索引效率已经很高了，查找一个节点所做的IO次数是这个节点所处的树的高度，因为我们无法把整个索引都加载到内存，并且节点数据在磁盘中不是顺序排放的。所以最快情况下，磁盘的IO次数为数的高度。

虽然平衡二叉树查找效率确实很高，但是频繁的IO才是阻碍提高性能的瓶颈，怎样减少IO次数呢？前辈们很聪明的提出了局部性原理，分为时间局部性原理，即加入你查询id为1的用户数据，过一段时间你还会查询id为1的数据，所以会将这部分数据缓存下来。空间局部性原理，当你查询id为1的用户数据的时候，你有很大的概率会去查询id为2，3，4的用户的数据，所以会一次性的把id为1，2，3，4的数据都读到内存中去，这个最小的单位就是页。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190827233646941.jpg?)

简单来说CPU进行运算是电子运动，计算速度很快。而将数据从硬盘读取到内存中是机械运动，很慢。我们在买硬盘的时候经常问这个硬盘是多少转（每分钟转动的圈数），7200转，5400转。所以说转动的越快加载数据越快，但是和CPU比起来差的还很远，所以说要减低IO次数。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190827233627885.png?)

B树和B+树的概念比较复杂，有兴趣的小伙伴可以点原文链接看看知乎上写的一篇文章，这里只做一个宏观的介绍

**前文已经提到树高决定着IO的次数，那么降低树高不就能减少IO的次数吗，怎么减少呢，每个节点的数据多放一点不就行了，并且这个数据是存放在一块的，对应的是数据库中的读取的最小单位页，一次IO就可以将这些数据读取出来，虽然比较的次数有可能会增加，但是在内存中的比较和磁盘IO相比差几个数量级，整体上效率还是提高了**

所以你看到的B树是这样的

![在这里插入图片描述](https://img-blog.csdnimg.cn/d7477eb37846425f841a22584b20b96d.png)

B+树是这样的

![在这里插入图片描述](https://img-blog.csdnimg.cn/bc6a16f5ba63430c89008db2b122783c.png)

那么B树和B+树的区别在哪呢？

 1. B+跟B树不同**B+树的非叶子节点不保存键值对应的数据，这样使得B+树每个节点所能保存的键值大大增加；**
 2. B+树叶子节点保存了父节点的所有键值和键值对应的数据，每个叶子节点的关键字从小到大链接（**这个特性对范围查找特别有利，范围查找只需要遍历链表即可，并不用像b树一样回旋查找，例如先查找5，再查找6，再查找7**）
 3. B+树的根节点键值数量和其子节点个数相等;
 4. B+的非叶子节点只进行数据索引，不会存实际的键值对应的数据，所有数据必须要到叶子节点才能获取到，所以每次数据查询的次数都一样；

放个图理解的更清楚一点

B树

![在这里插入图片描述](https://img-blog.csdnimg.cn/c420be951d1549f9813d5cd4b2a87aff.png)

B+树

![在这里插入图片描述](https://img-blog.csdnimg.cn/9411fe1b38074932aa7879a7c8cd3b8a.png)

**在B树的基础上每个节点存储的关键字数更多，树的层级更少所以查询数据更快，所有关键字指针都存在叶子节点，所以每次查找的次数都相同所以查询速度更稳定。**

除此之外，B+树的叶子节点是跟后序节点相连接的，这对范围查找是非常有用的。

**看到没B+树的非叶子节点是主键，主键占用的空间越小，每个节点能放的主键就能更多，这就是为什么我们的主键一般不设置太大的原因。主键占用的空间小，能降低树高，减少IO次数**。

## 聚集索引和联合索引
在InnoDB存储引擎中，是以主键为索引来组织数据的。在InnoDB存储引擎中，每张表都有个主键，如果再创建表时没有显示的定义主键，则InnoDB存储引擎会按如下方式选择或创建主键。

 1. 首先判断表中是否有非空的唯一索引，如果有，则该列即为主键
 2. 如果不符合上述条件，InnoDB存储引擎自动创建一个6字节大小的指正作为索引
 3. 如果有多个非空唯一索引时，InnoDB存储引擎将选择建表时第一个定义的非空唯一索引作为主键

假如说有如下数据，用户id为主键（1， tom），（2，mike），（3，sam），（4，lisa），（5，li）则数据是这样存储的，图1

![在这里插入图片描述](https://img-blog.csdnimg.cn/783b6de0e28b4dd5a920c2eb67afd3e6.png)

假如说我们现在对用户名建索引，用户名索引是怎么存的呢？图2
![在这里插入图片描述](https://img-blog.csdnimg.cn/c16be15c032e4b53a0eac2a03402702f.png)

用户名索引主键存储的是主键，所以当我们运行如下sql语句时

```sql
select * from table where name ="sam"
```
过程是这样的，先在name索引上找到对应的主键，在根据对应的主键去建表时建立的B+树上找到对应的记录，即先在图2上找，再到图1上找。

聚集索引：数据行的物理顺序与列值（一般是主键的那一列）的逻辑顺序相同，一个表中只能拥有一个聚集索引。图1用的就是聚集索引

非聚集索引：定义：该索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同，一个表中可以拥有多个非聚集索引。图2用的就是非聚集索引

最后再说一个联合索引，联合索引是指对表上的多个列进行索引。创建方式如下：

```sql
CREATE TABLE `t` (
  `a` int(10),
  `b` int(10),
  PRIMARY KEY (`a`),
  KEY `idx_a_b` (`a`,`b`)
) ENGINE=InnoDB;
```
多个键值得B+树是如下存储的
![在这里插入图片描述](https://img-blog.csdnimg.cn/61c10389914c40c0adc7b867e862288c.png)

可以看到键值都是排序的，就上面的例子来说（1，1）（1，2）（2，1）（2，4）（3，1）（3，2），数据按照（a，b）的顺序进行了存放。

因此对于查询select * from table where a = xxx and b = xxx，显然是可以使用（a，b）这个联合索引的。对于单个的a列查询select * from table where  a = xxx，也可以使用（a，b）这个索引。但对于b列的查询select * from table where b = xxx，则不可以使用这颗B+树索引。可以发现叶子节点上的b值为1，2，1，4，1，2，显然不是排序的，因此对于b列的查询使用不到（a，b）的索引

### 哈希表
InnoDB存储引擎会监控对表上各项索引页的查询。如果观察到建立哈希索引可以带来速度提升，则建立哈希索引，称之为自适应哈希索引，DBA不能对建立哈希索引的过程进行干预，只能启动或禁用自适应哈希索引

数据库一般采用除法散列的方法，即取k除以m的余数，将关键词k映射到m个槽的某一个去，即哈希函数为h(k) = k mod m，当发生冲突时，即两个关键字可能映射到同一个槽上，采用链接法，即以链表的形式保存冲突的关键字，和HashMap类似

当对热点数据建立了哈希索引以后，省去在B+树上进行查找，可以极大地提高服务的性能

![在这里插入图片描述](https://img-blog.csdnimg.cn/9cf571b91e5f43168cdc95dc6c699aa9.png)