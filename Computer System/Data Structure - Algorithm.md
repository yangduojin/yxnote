- [数据结构分类](#数据结构分类)
- [排序算法](#排序算法)
  - [归并排序](#归并排序)
  - [归并迭代](#归并迭代)
  - [快速排序](#快速排序)
- [二叉树(平衡 不平衡 )](#二叉树平衡-不平衡-)
  - [红黑树](#红黑树)
  - [多叉树 2-3-4 树](#多叉树-2-3-4-树)
  - [B树](#b树)
  - [Hash](#hash)


## 数据结构分类

| 名称     | 好处                                                                                                          |
| -------- | ------------------------------------------------------------------------------------------------------------- |
| 数组     | 插入效率低，知道下标就可以非常快的存取,查找慢，删除慢，大小固定(定义: 线性表，连续的内存空间，相同类型的数据) |
| 有序数组 | 比无序数组查找快,删除插入慢，大小固定                                                                         |
| 栈       | 先进后出,存取其他项很慢                                                                                       |
| 队列     | 先进先出,存取其他项很慢                                                                                       |
| 链表     | 插入快，删除快,查找慢                                                                                         |
| 二叉树   | 查找，插入，删除都快(如果树平衡),删除算法复杂                                                                 |
| 红黑树   | 查找，插入，删除都快，树总平衡,算法复杂                                                                       |
| 2-3-4树  | 查找，插入，删除都快 树总平衡，类似的树对磁盘存储有用,算法复杂                                                |
| 哈希表   | 如果关键字已知则存取极快，插入快,删除慢，不知道关键字则存取慢，对存储空间使用不充分                           |
| 堆       | 插入删除快，对最大数据项的存取很快(堆是特殊的二叉树？),对其他数据项存取很慢                                   |
| 图       | 对现实世界建模,有些算法慢且复杂                                                                               |
## 排序算法

| 种类     | 特点                                                                   |
| -------- | ---------------------------------------------------------------------- |
| 冒泡     | 双指针 外层从右--，里层从左++且里层判断要 -1，不然会指针异常           |
| 选择     | 双指针，都是从左侧开始，内选出最小的值与外交换。冒泡变种，冒泡从右开始 |
| 插入排序 | 没懂                                                                   |
| 希尔排序 | 没懂                                                                   |

算法就是纯逻辑 （递归使用不好容易引起系统雪崩，建议替换为非递归）

Log 以2为底数转换到以10为底数 需*3.322 得到大概值

### 归并排序

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

### 归并迭代

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

### 快速排序

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

```
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

- 第一种 颜色变换 插入程序在从根往下寻找时，遇到一个有两个红色子节点的黑色节点时，就必须改变黑色节点为红色，两个子节点改为黑色，提供相同的黑色高度，但黑变红可能违背规则3(他的父是红色)，此问题需在继续向下沿着路径插入新节点之前解决，需要旋转修正
  - 注意：根和他的两个子节点做颜色变换时，根和他的子节点一样都是黑色
  
- 第二种 插入节点的旋转 分三种情况(x 插入点，p: x的父，g: p的父，x的祖父，g不是root)
  1. p是黑色
    - 什么也不做 什么规则也不违背，直接插入x即可
  2. p是红色，x是g的一个外侧子孙节点
    - p红色且x为外侧子孙，需要一次旋转和一些颜色变化(在每次旋转后，下一次旋转之前改变颜色)
    - 50 25 75 12 6 、50 25 75 87 93  这两种对称
  3. p是红色，x是g的一个内侧子孙节点
    - p为红色，x是g的内侧子孙节点，需要两次旋转和一些颜色变化(在每次旋转后，下一次旋转之前改变颜色)
    - 50 25 75 12 18 先要把18左旋到12的位置，然后12 25变色，25再右旋成为18的右子节点
    - 下面是有兄弟节点或叔节点是否会导致别的情况出现? 如果要插入的x有个兄弟节点s，s是p的另一个子节点
      - 如果p是黑色，直接插入x
      - 如果p是红色，他的两个子节点都比为黑色，但是x作为新插入的节点为红色，且p不能单独有个黑色的子节点s，因为这样导致s和空兄弟节点的黑色高度不一样。因此x是红色，x不可能有一个兄弟节点，除非p是红色
    - 如p的父节点g有一个子节点u，p u 为兄弟，u 是x的叔节点
      - 如p是黑色，插入x不需要旋转
      - p是红色，u也必为红色，但是有两个红色节点的黑色父节点在沿路径向下的时候颜色变换了，所以这种情况也是不存在的，因此前三种可能性就是全部可能存在的情况；
- 在下行的路途中 使用颜色变换已经消除了旋转造成树的上方任何规则的违规情况；
- 颜色变换使红黑树的插入效率比其他平衡树高，且保证了下行在路途中仅在树上行走了一遍
	- 第三种 在向下路途上的旋转
	  - 违规节点是外侧子孙节点
		- 该节点为父子红红冲突中的子节点，需要两次颜色改变和一次旋转 
		  - 50 25 75 12 37 6 18 在插入 12 和 6(俩都是root外侧子孙) 时需要颜色变换。现在插入3的值，25就会旋转为root，25右子节点变为50左左子节点
		- 违规节点是内侧子孙节点
		  - 下行途中红红冲突，x是内侧子孙节点，则需要两次旋转改正
		  - 50 25 75 12 37 31 43 插入新值28红色，会从下网上改变 最后达到树平衡
- 在插入过程中，保持树的红黑正确性，因此可以取得树的平衡；
- 删除节点：太复杂 有种逃避方式就是设置一个删除标识，但不是物理删除，所有查到该节点的例程都知道不用报告已找到该节点，适用于不经常执行删除操作

### 多叉树 2-3-4 树

- 非叶节点的子节点数总是比它含有的数据项+1
- 叶节点没有子节点，但是含有1、2、3个数据项，空节点是不会存在的
- 除叶节点外，其余子节点不能出现子节点指向一个链接，链接为null，有一个数据项的节点必须总是保持两个链接
一般不允许出现重复关键字值

### B树

- 磁盘的树和内存的树不一样 内存链接引用是地址值 磁盘链接引用是文件中块的编号
- 磁盘中数据块(读取头最小可读取范围)作为树节点，并保存节点间的链接(链接到其它块去，因为一个节点对应一个块)
- 
### Hash

arrayIndex = hugeNumber % arraysize