# LeetCode

## 数组

1. 删除排序数组中的重复项: 双指针，I 可以为动态增长的length 慢， j快指针遍历，遇到不同的值,赋值给nums[i]

    ```java
    int len = nums.length;
    k %= len;
    if(k == 0 || len == 1) return 0;
    int temp = nums[0];
    int count = 0;
    for (int i = k,cnt = 0; cnt < len; i+=k,cnt++) {
        int t = nums[i%len];
        nums[i%len] = temp;
        temp = t;
        if(i%len == count){
            count++;
            i = count;
            temp = nums[(i)%len];}}
    ```

2. 买卖股票最佳时机2: 只要第二天比第一天大就卖，上升区间的和
3. 旋转数组: k的大小就是值跳的次数 or 双重for：nums1[i] = nums1[i - 1]
4. 重复数字: Set.add 返回添加结果与否
5. 找出唯一的数字: 异或，但只适合只有一个唯一数字
6. 两个数组的交集II: 双指针，sort排序，while循环，arraylist；map getOrDefault,数字做键，次数做值
7. 加1: 条件循环，return直接跳出整个循环，方法，返回结果
8. 移动0: 双指针，单循环 交换位置；for while 分治
9. 两数之和: 双指针或hashmap   (m.get(target - nums[i]) != null) / m.put(nums[i], i)

## String

1. 反转字符串: 双指针
2. 整数反转: %10，用身下的数字去*10 再加上下一次模下的数字再*10
3. 字符串中第一个唯一字符: ``两次遍历,用int[26]去遍历s.toCharArray(),count[chars[i] - 'a']++;if(count[chars[i] - 'a' == 1)return I    foreach hashmap.put(ch, map.getOrDefault(ch, 0) + 1); map.get(chars[i]) == 1``
4. 有效的字母异位词

    ```java
    int[] letterCount = new int[26];
    letterCount[s.charAt(i) - 'a']++;
    if (letterCount[t.charAt(i) - 'a'] == 0)
                    return false;
    letterCount[t.charAt(i) - 'a']--;
    ```
