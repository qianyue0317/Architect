##  HashMap 面试

#### 1.说说你对hash算法的理解

hash的基本概念就是把任意长度的输入通过一个hash算法之后，映射成固定长度的输出

   ##### 1.1 hash算法任意长度的输入 转化为了 固定长度的输出，会不会有问题呢？

肯定会有问题的。在程序中（可能）碰到两个value值经过hash算法之后，算出同样的hash值，也就是会发生hash冲突

 ##### 1.2 hash冲突能避免么？

理论上是没有办法避免的，就类比“抽屉原理”，比如说一共有10个苹果，但是咱一共有9个抽屉，最终一定会有一个抽屉里的数量是大于1的，所以hash冲突没有办法避免，只能尽量避免。

#### 2.你认为好的hash算法，应该考虑点有哪些呢？

1. 首先这个hash算法，它一定效率得高，要做到长文本也能高效计算出hash值嘛，
2. hash值不能让它逆推出原文吧；两次输入，只要有一点不同，它也得保证这个hash值是不同的。
3. 其次，就是尽可能的要分散吧，因为，在table中slot中slot大总会发都处于空闲状的，要尽可能降低hash冲突。

#### 3.HashMap中存储数据的结构是什么样的呢？

JDK1.7 是 数组 + 链表；
JDK1.8是 数组 + 链表 + 红黑树，每个数据单元都是一个Node结构，Node结构中有key字段、有value字段、还有next字段、还有hash字段。next字段就是发生hash冲突的时候，当前桶位中node与冲突的node连成一个链表要用的字段。

#### 4.创建HashMap时，不指定散列表数组长度，初始长度是多少呢？

初始长度默认是16

   ##### 4.1 散列表是new HashMap() 时创建的么？

散列表是懒加载机制，只有第一次put数据的时候，它才创建的

#### 5.默认负载因子是多少呢，并且这个负载因子有什么作用？

默认负载因子0.75，就是75%，负载因子它的作用就是计算扩容阈值用的，比如使用无参构造方法创建的hashmap对象，它默认情况下扩容阈值就 16*0.75 = 12

#### 6.链表转化为红黑树，需要达到什么条件呢？

链表转红黑树，主要是有两个指标。

1. 其中一个就是链表长度达到8，
2. 这有个指标就是当前散列表数组长度它已经达到64。否则的话，就算slot内部链表长度到了8，它也不会链转树，它仅仅会发生一次resize，散列表扩容。

#### 7.Node对象内部的hash字段，这个hash值是key对象的hashcode()返回值么？

不是的

   #####  7.1 这个hash值是怎么得到呢？

这个hash值是key.hashcode二次加工得到的。
加工原则是：key的hashcode 高16位 ^ 低16位，得到的一个新值。

 ##### 7.2hash字段为什么采用高低位异或？

主要为了优化hash算法，因为hashmap内部散列表，它大多数场景下，它不会特别大。也就是说这个[table.length - 1] 得到的这个二进制数，实际有效位很有限，一般都在（低）16位以内，这样的话，key的hash值高16位就等于完全浪费了，没起到作用。所以，node的hash字段才采用了 高16位 异或 低16位 这种方式来搞它。

#### 8.HashMap put 写数据的具体流程，尽可能的详细点！

主要为4种情况：
前面这个，寻址算法是一样的，都是根据key的hashcode 经过 高低位 异或 之后的值，然后再 按位与 & （table.length -1)，得到一个槽位下标，然后根据这个槽内状况，状况不同，情况也不同，大概就是4种状态，
第一种是slot == null，直接占用slot就可以了，然后把当前put方法传进来的key和value包状成一个Node 对象，放到这个slot中就可以了
第二种是slot != null 并且 它引用的node 还没有链化；需要对比一下，node的key 与当前put 对象的key 是否完全相等；如果完全相等的话，这个操作就是replace操作，就是替换操作，把那个新的value替换当前slot -> node.value 就可以了；否则的话，这次put操作就是一个正儿八经的hash冲突了，slot->node 后面追加一个node就可以了，采用尾插法。
第三种就是slot 内的node已经链化了；这种情况和第二种情况处理很相似，首先也是迭代查找node，看看链表上的元素的key，与当前传来的key是不是完全一致。如果一致的话，还是repleace操作，替换当前node.value，否则的话就是我们迭代到链表尾节点也没有匹配到完全一致的node，把put数据包装成node追加到链表尾部；这块还没完，还需要再检查一下当前链表长度，有没有达到树化阈值，如果达到阈值的话，就调用一个树化方法，树化操作都在这个方法里完成
第四种就是冲突很严重的情况下，就是那个链已经转化成红黑树了


#### 9.红黑树的写入操作，是怎么找到父节点的，找父节点流程？

#### 10.TreeNode数据结构，简单说下。

#### 11.红黑树的原则有哪些呢？

#### 12.JDK8 hashmap为什么引入红黑树？解决什么问题？

    追问：为什么hash冲突后性能变低了？【送分题】

#### 13.hashmap 什么情况下会触发扩容呢？
​    追问：触发扩容后，会扩容多大呢？算法是什么？
​    追问：为什么采用位移运算，不是直接*2？

#### 14.hashmap扩容后，老表的数据怎么迁移到扩容后的表的呢？

#### 15.hashmap扩容后，迁移数据发现该slot是颗红黑树，怎么处理呢？



参考：

1. https://blog.csdn.net/weixin_32265569/article/details/108481327
2. https://www.bilibili.com/video/BV1Qk4y1672n?spm_id_from=333.999.0.0

