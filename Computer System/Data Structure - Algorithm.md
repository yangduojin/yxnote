# 数据结构和算法

- [数据结构和算法](#数据结构和算法)
  - [数据结构分类](#数据结构分类)
  - [算法](#算法)
    - [排序算法](#排序算法)
      - [归并排序](#归并排序)
      - [归并迭代](#归并迭代)
      - [快速排序](#快速排序)
    - [LRU算法原理及其实现](#lru算法原理及其实现)
      - [LRU算法实现思路](#lru算法实现思路)
      - [LRU的简单实现](#lru的简单实现)
  - [二叉树(平衡 不平衡 )](#二叉树平衡-不平衡-)
    - [红黑树](#红黑树)
    - [多叉树 2-3-4 树](#多叉树-2-3-4-树)
    - [B树](#b树)
    - [Hash](#hash)
  - [主键算法](#主键算法)
    - [生成主键方案有哪些](#生成主键方案有哪些)
      - [UUID的优缺点](#uuid的优缺点)
      - [数据库自增主键的优缺点](#数据库自增主键的优缺点)
      - [基于Redis生成全局ID策略优缺点](#基于redis生成全局id策略优缺点)
      - [雪花算法，Twitter的分布式自增ID算snowflake优缺点](#雪花算法twitter的分布式自增id算snowflake优缺点)
    - [雪花算法 (算出来有19位，只能用String接收)](#雪花算法-算出来有19位只能用string接收)

## 数据结构分类

常用的4种数据结构有

1. 集合：只有同属于一个集合的关系，没有其他关系
2. 线性结构：结构中的数据元素之间存在一个对一个的关系
3. 树形结构：结构中的数据元素之间存在一个对多个的关系
4. 图状结构或者网状结构：图状结构或者网状结构

| 名称    | 好处                                                                                                  |
| ------- | ----------------------------------------------------------------------------------------------------- |
| 位(bit) | 计算机二进制,正码,反码,补码                                                                           |
| 数组    | 插入效率低，知道下标就可以非常快的存取,删除慢，大小固定(定义: 线性表，连续的内存空间，相同类型的数据) |
| 栈      | 先进后出,存取其他项很慢                                                                               |
| 队列    | 先进先出,存取其他项很慢                                                                               |
| 链表    | 插入快，删除快,查找慢(单向,双向,跳表)                                                                 |
| 树      | 查找，插入，删除都快(如果树平衡),删除算法复杂   (二叉树,红黑树,2-3-4树)                               |
| 哈希表  | 如果关键字已知则存取极快，插入快,删除慢，不知道关键字则存取慢，对存储空间使用不充分                   |
| 堆      | 插入删除快，对最大数据项的存取很快(堆是特殊的二叉树？),对其他数据项存取很慢                           |
| 图      | 对现实世界建模,有些算法慢且复杂                                                                       |

## 算法

### 排序算法

| 种类     | 特点                                                                   |
| -------- | ---------------------------------------------------------------------- |
| 冒泡     | 双指针 外层从右--，里层从左++且里层判断要 -1，不然会指针异常           |
| 选择     | 双指针，都是从左侧开始，内选出最小的值与外交换。冒泡变种，冒泡从右开始 |
| 插入排序 | 没懂                                                                   |
| 希尔排序 | 没懂                                                                   |

算法就是纯逻辑 （递归使用不好容易引起系统雪崩，建议替换为非递归）

Log 以2为底数转换到以10为底数 需*3.322 得到大概值

#### 归并排序

i = 0 ; arr2[i++] = arr[i++]; 从左到右开始计算， arr2[0] = arr[1],最后i = 2;

分为递归和迭代两种，先分 再治，最后合并，不考虑太多

```java
public static void sort2(int []arr){
        int[] temp = new int[arr.length];
        sort2(arr,0,arr.length -1 ,temp);
    }
    private static void sort2(int[] arr, int left, int right, int[] temp) {
        if(left < right){
            int mid = (left + right) / 2;
            sort2(arr, left, mid, temp);
            sort2(arr, mid + 1, right, temp);
            merge2(arr, left, mid, right, temp);
        }
    }
    private static void merge2(int[] arr, int left, int mid, int right, int[] temp) {
        int i = left;
        int j = mid + 1;
        int t = 0;
        while (i<= mid && j<=right) {
            if(arr[i] <= arr[j]){
                temp[t++] = arr[i++];
            }else{
                temp[t++] = arr[j++];
            }
        }
        while(i <= mid){
            temp[t++] = arr[i++];
        }
        while(j <= right){
            temp[t++] = arr[j++];
        }
            t=0;
        while(left <= right){
            arr[left++] = temp[t++];
        }
    }
```

#### 归并迭代

```java
public static void merge_sort(int[] arr) {
    int len = arr.length;
    int[] result = new int[len];
    int block, start;

    // 原版代码的迭代次数少了一次，没有考虑到奇数列数组的情况
    for(block = 1; block < len*2; block *= 2) {
        for(start = 0; start <len; start += 2 * block) {
            int low = start;
            int mid = (start + block) < len ? (start + block) : len;
            int high = (start + 2 * block) < len ? (start + 2 * block) : len;
            //两个块的起始下标及结束下标
            int start1 = low, end1 = mid;
            int start2 = mid, end2 = high;
            //开始对两个block进行归并排序
            while (start1 < end1 && start2 < end2) {
            result[low++] = arr[start1] < arr[start2] ? arr[start1++] : arr[start2++];
            }
            while(start1 < end1) {
            result[low++] = arr[start1++];
            }
            while(start2 < end2) {
            result[low++] = arr[start2++];
            }
        }
    int[] temp = arr;
    arr = result;
    result = temp;
    }
    result = arr;       
}
```

```java
static void merge_sort_recursive(int[] arr, int[] result, int start, int end) {
    if (start >= end)
        return;
    int len = end - start, mid = (len >> 1) + start;
    int start1 = start, end1 = mid;
    int start2 = mid + 1, end2 = end;
    merge_sort_recursive(arr, result, start1, end1);
    merge_sort_recursive(arr, result, start2, end2);
    int k = start;
    while (start1 <= end1 && start2 <= end2)
        result[k++] = arr[start1] < arr[start2] ? arr[start1++] : arr[start2++];
    while (start1 <= end1)
        result[k++] = arr[start1++];
    while (start2 <= end2)
        result[k++] = arr[start2++];
    for (k = start; k <= end; k++)
        arr[k] = result[k];
}
public static void merge_sort(int[] arr) {
    int len = arr.length;
    int[] result = new int[len];
    merge_sort_recursive(arr, result, 0, len - 1);
}
```

#### 快速排序

找一个基准值，所有比基准小的放基准前面，大的放基准后面

递归地把"基准值前面的子数列"和"基准值后面的子数列"进行排序。

```java
public static void quickSort(int[] a, int l, int r) {
       if (l < r) {
           int i,j,x;
           i = l;
           j = r;
           x = a[i];
           while (i < j) {
               while(i < j && a[j] > x)
                   j--; // 从右向左找第一个小于x的数
               if(i < j)
                   a[i++] = a[j];
               while(i < j && a[i] < x)
                   i++; // 从左向右找第一个大于x的数
               if(i < j)
                   a[j--] = a[i];   }
           a[i] = x;
           quickSort(a, l, i-1); /* 递归调用 */
           quickSort(a, i+1, r); /* 递归调用 */
       } }
```

```java
private static void quickSort2(int[] arr, int low, int high) {
    if (low < high) {
        // 找寻基准数据的正确索引
        int index = getIndex(arr, low, high);
//quickSort(arr, 0, index - 1); 之前的版本，这种姿势有很大的性能问题
        quickSort2(arr, low, index - 1);
        quickSort2(arr, index + 1, high);
    } }
    private static int getIndex(int[] arr, int low, int high) {
        // 基准数据
        int tmp = arr[low];
        while (low < high) {//双指针 前后移动
            while (low < high && arr[high] >= tmp) {
                high--;
            }// 该while不满足触发下面的语句
            arr[low] = arr[high];
            while (low < high && arr[low] <= tmp) {
                low++;
            }// 该while不满足触发下面的语句
            arr[high] = arr[low];
        }// 此时low = high
        arr[low] = tmp;// 将基准值放入正确的位置
        return low; // 返回tmp的正确位置}}
```

### LRU算法原理及其实现

LRU(Least Recently Used) 即最近最少使用，属于典型的内存淘汰机制。

#### LRU算法实现思路

根据LRU算法的理念，我们需要：

1. 一个参数cap来作为最大容量
2. 一种数据结构来存储数据，并且需要1. 轻易地更新最新的访问的数据。2. 轻易地找出最近最少被使用的数据，当到达cap时，清理掉。

在这里，我们用到的数据结构是：hashmap+双向链表。

1. 利用hashmap的get、put方法O(1)的时间复杂度，快速取、存数据。
2. 利用doublelinkedlist的特征（可以访问到某个节点之前和之后的节点），实现O(1)的新增和删除数据。

![当key2再次被使用时，它所对应的node3被更新到链表头部。](./img/LRUFirst.png)

当key2再次被使用时，它所对应的node3被更新到链表头部。

![假设cap=3，当key4创建、被访问后，处于链表尾部的node2将被淘汰，key1将被清楚。](./img/LRUsecond.png)

假设cap=3，当key4创建、被访问后，处于链表尾部的node2将被淘汰，key1将被清楚。

#### LRU的简单实现

1. 节点node,存放key、val值、前节点、后节点

    ```java
    class Node{
        public int key;
        public int val;
        public Node next;
        public Node previous;

        public Node() {}

        public Node(int key, int val) {
            this.key = key;
            this.val = val;}}
    ```

2. 双向链表，属性有size、头节点、尾节点。

    addFirst(): 头插法入链表
    remove(): 删除最后一个节点
    remove(Node node):删除特定节点
    size()：获取链表长度

    ```java
    class DoubleList{
        private int size;
        private Node head;
        private Node tail;

        public DoubleList() {
            this.head = new Node();
            this.tail = new Node();
            size = 0;
            head.next = tail;
            tail.previous = head;}

        public void addFirst(Node node){
            Node temp = head.next;
            head.next = node;
            node.previous = head;
            node.next = temp;
            temp.previous = node;
            size++;}

        public void remove(Node node){
            if(null==node|| node.previous==null|| node.next==null){
                return;}

            node.previous.next = node.next;
            node.next.previous = node.previous;
            node.next=null;
            node.previous=null;
            size--;}

        public void remove(){
            if(size<=0) return;
            Node temp = tail.previous;
            temp.previous.next = temp.next;
            tail.previous = temp.previous;
            temp.next = null;
            temp.previous=null;
            size--;}

        public int size(){
            return size;}}
    ```

3. LRU算法实现类

    get(int key): 为null返回-1
    put(int key, int value)

    若map中有，删除原节点，增加新节点
    若map中没有，map和链表中新增新数据。

    ```java
    public class LRUCache {

        Map<Integer,Node> map;
        DoubleList cache;
        int cap;


        public LRUCache(int cap) {
            map = new HashMap<>();
            cache = new DoubleList();
            this.cap = cap;}

        public int get(int key){
            Node node = map.get(key);
            return  node==null? -1:node.val;}

        public void put(int key, int val){
            Node node = new Node(key,val);
            if(map.get(key)!=null){
                cache.remove(map.get(key));
                cache.addFirst(node);
                map.put(key,node);
                return;}

            map.put(key,node);
            cache.addFirst(node);
            if(cache.size()>cap){
                cache.remove();}}

        public static void main(String[] args) {
            //test, cap = 3
            LRUCache lruCache = new LRUCache(3);
            lruCache.put(1,1);
            lruCache.put(2,2);
            lruCache.put(3,3);
            //<1,1>来到链表头部
            lruCache.put(1,1);
            //<4,4>来到链表头部， <2,2>被淘汰。
            lruCache.put(4,4);}}
    ```

4. LRU应用场景

- 底层的内存管理，页面置换算法
- 一般的缓存服务，memcache\redis之类
- 部分业务场景

[参考文档地址](http://doumaomao.github.io/blog/LRU%E5%8E%9F%E7%90%86%E4%BB%A5%E5%8F%8A%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF.html)

## 二叉树(平衡 不平衡 )

二叉搜索树 只有两个节点或者没有节点
删除节点 继承节点successor 不是current右节点的话就一定是 的右节点的最左节点

avl树 不常用

### 红黑树

4原则

1. 每个节点都必须有颜色(新插入的节点总是红色)
2. 根是黑色,所有NIL节点都是黑色
3. 如节点是红色，他的子节点必为黑（红红不行，黑黑行）(一条路径不能出现两个相邻的红色节点)
4. 从根到叶节点或空子节点(该叶结点的兄弟节点，只不过为空)的每条路径，必须包含相同数目的黑色节点，包含叶节点本身。(黑色高度)

总结"有红必有黑，红红不相连"

增加了某些特点的二叉搜索树

插入节点的三个部分，复杂度排列为

1. 在下行路途中的颜色变换
2. 插入节点之后的旋转
3. 在向下路途上的旋转

如果按照时间来排，应该是 1 3 2，但是在树底旋转比树中间旋转容易，且1 2 比3常用的多

1. 第一种 颜色变换 插入程序在从根往下寻找时，遇到一个有两个红色子节点的黑色节点时，就必须改变黑色节点为红色，两个子节点改为黑色，提供相同的黑色高度，但黑变红可能违背规则3(他的父是红色)，此问题需在继续向下沿着路径插入新节点之前解决，需要旋转修正
     - 注意：根和他的两个子节点做颜色变换时，根和他的子节点一样都是黑色
  
2. 第二种 插入节点的旋转 分三种情况(x 插入点，p: x的父，g: p的父，x的祖父，g不是root)
   1. p是黑色
       - 什么也不做 什么规则也不违背，直接插入x即可
   2. p是红色，x是g的一个外侧子孙节点
       - p红色且x为外侧子孙，需要一次旋转和一些颜色变化(在每次旋转后，下一次旋转之前改变颜色)
       - 50 25 75 12 6 、50 25 75 87 93  这两种对称
   3. p是红色，x是g的一个内侧子孙节点
       1. p为红色，x是g的内侧子孙节点，需要两次旋转和一些颜色变化(在每次旋转后，下一次旋转之前改变颜色)
       2. 50 25 75 12 18 先要把18左旋到12的位置，然后12 25变色，25再右旋成为18的右子节点
       3. 下面是有兄弟节点或叔节点是否会导致别的情况出现? 如果要插入的x有个兄弟节点s，s是p的另一个子节点
            - 如果p是黑色，直接插入x
            - 如果p是红色，他的两个子节点都比为黑色，但是x作为新插入的节点为红色，且p不能单独有个黑色的子节点s，因为这样导致s和空兄弟节点的黑色高度不一样。因此x是红色，x不可能有一个兄弟节点，除非p是红色
       4. 如p的父节点g有一个子节点u，p u 为兄弟，u 是x的叔节点
             - 如p是黑色，插入x不需要旋转
             - p是红色，u也必为红色，但是有两个红色节点的黑色父节点在沿路径向下的时候颜色变换了，所以这种情况也是不存在的，因此前三种可能性就是全部可能存在的情况；
3. 在下行的路途中 使用颜色变换已经消除了旋转造成树的上方任何规则的违规情况；
4. 颜色变换使红黑树的插入效率比其他平衡树高，且保证了下行在路途中仅在树上行走了一遍
     - 第三种 在向下路途上的旋转
       - 违规节点是外侧子孙节点
     - 该节点为父子红红冲突中的子节点，需要两次颜色改变和一次旋转
       - 50 25 75 12 37 6 18 在插入 12 和 6(俩都是root外侧子孙) 时需要颜色变换。现在插入3的值，25就会旋转为root，25右子节点变为50左左子节点
     - 违规节点是内侧子孙节点
       - 下行途中红红冲突，x是内侧子孙节点，则需要两次旋转改正
       - 50 25 75 12 37 31 43 插入新值28红色，会从下网上改变 最后达到树平衡
5. 在插入过程中，保持树的红黑正确性，因此可以取得树的平衡；
6. 删除节点：太复杂 有种逃避方式就是设置一个删除标识，但不是物理删除，所有查到该节点的例程都知道不用报告已找到该节点，适用于不经常执行删除操作

### 多叉树 2-3-4 树

- 非叶节点的子节点数总是比它含有的数据项+1
- 叶节点没有子节点，但是含有1、2、3个数据项，空节点是不会存在的
- 除叶节点外，其余子节点不能出现子节点指向一个链接，链接为null，有一个数据项的节点必须总是保持两个链接
一般不允许出现重复关键字值

### B树

- 磁盘的树和内存的树不一样 内存链接引用是地址值 磁盘链接引用是文件中块的编号
- 磁盘中数据块(读取头最小可读取范围)作为树节点，并保存节点间的链接(链接到其它块去，因为一个节点对应一个块)

### Hash

arrayIndex = hugeNumber % arraysize

## 主键算法

### 生成主键方案有哪些

1. UUID
2. 数据库自增主键。
3. 基于Redis生成全局ID策略。
4. 雪花算法，Twitter的分布式自增ID算法snowflake。
5. 百度UidGenerator算法(基于雪花算法实现自定义时间戳)。
6. 美团Leaf算法(依赖于数据库，ZK)。

#### UUID的优缺点

- 优点：性能非常高，JDK自带本地生成，无网络消耗。
- 缺点：
  - 只保证了唯一性，趋势递增。
  - 无序，无法预测他的生成规则，不能生成递增有序的数字。(字符储存，占用空间大)
  - mysql官方推荐主键越短越好，UUID包含32个16位进制的字母数字，每一个都很长。
  - B+树索引的分裂。主键是包含索引的，mysql的索引是通过B+树来实现的，每一次新的UUID数据插入，为了查询优化，因为UUID是无序的，都会对索引底层的B+树进行修改。插入无序，不但会导致一些中间节点产生分裂，也会白白创造很多不饱和的节点，大大降低了数据库插入的性能。

#### 数据库自增主键的优缺点

- 优点：简单方便易用。
- 缺点：
  - 要设置增长步长，系统水平扩展比较困难。
  - 每次获取ID都得读写一次数据库，数据库压力大，非常影响性能，不符合分布式ID里低延迟和高QPS的规则。

#### 基于Redis生成全局ID策略优缺点

- 优点：满足分布式ID生成要求，并且已有成功落地案例。
- 缺点：
  - 要设置增长步长，同时key一定要设置有效期。
  - 为了一个分布式ID，要搞一个Redis集群，维护成本大。

#### 雪花算法，Twitter的分布式自增ID算snowflake优缺点

- 优点：
  - 经测试snowflake每秒能生成26万个自增可排序的ID。
  - snowflake生成的ID结果是一个64bit大小的整数，为一个Long型 （转换成字符串后长度最多19）。
  - 分布式系统内不会产生ID碰撞(datacenter和workerId作区分)且效率高。
  - 不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成ID的性能也非常高，可以根据自身业务分配bit位，非常灵活。
- 缺点：依赖机器时钟，如果机器时钟回拨，会导致id重复。由于是部署到分布式环境，每台机器上的时钟不可能完全同步，有时候出现不是全局递增的情况。（一般分布式ID只要求趋势递增，并不会严格要求递增，90%的需求只要求趋势递增，可以忽略这个缺点，或者按实际情况进行改进，如下代码demo）

### 雪花算法 (算出来有19位，只能用String接收)

- 1bit 表示正负，现固定为0
- 41bit-时间戳
- 10bit-工作机器id
- 12bit-序列号
- 所有生成的id按时间趋势递增
- 整个分布式系统内不会产生重复id（因为有datacenterId和workerId来做区分）

- 优点
  1. 毫秒数在高位，自增序列在低位，整个Id都是趋势递增的
  2. 不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成id的性能也是非常高的
  3. 可以根据自身业务特性分配bit位，非常灵活
- 缺点：
  1. 依赖机器时钟，如果机器时钟回拨，会导致重复id生成
  2. 在单机上是递增的，但是由于设计到分布式环境，每台机器上的时钟不可能完全同步，有时会出现不是全局递增的情况(此缺点可以无视，一般分布式id只要求趋势递增，不会严格要求递增，90%的需求只要求趋势递增)

对于时钟回拨导致重复id可以用 UidGenerator(百度) / Leaf(美团)
