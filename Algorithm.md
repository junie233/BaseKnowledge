## Base Structures & Algorithm



## 避免哈希冲突的两种方法

- 拉链法
- 线性探测法

## 字符串

- [把字符串中的每个空格替换成"%20"](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/02.md)

  在禁止使用java array的相关方法前提下，如果从前向后遍历，必然会出现需要移动指针之后的所有字符，这是很复杂的事情，但是如果从后向前遍历，就可以保证每次只移动一个字符串

- [字符串的排列](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/25.md)

  如果采用n个循环是没办法确定循环个数的，因此要采用递归的思想；

  将字符串看成两部分，第一个字符会和之后的字符串，每次都先确定第一个字符的可能性，即通过循环与后面的每一位交换。  得到第一位的可能性之后再来确定后面每一位的可能性，即递归调用。

- [第一个只出现一次的字符](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/32.md)

  首先采用hashmap遍历统计每一个字符出现的次数，接着再遍历一次找到字符出现次数为1的第一个字符。

  这里考虑到的是hashmap.get() 也是O(n),所以不建议去增加它的值，没有必要；所以，只需要改变状态就可以；

  即只有三种状态  没出现；出现一次（存储index）；出现多次；

  ​	

  在实现也有两种方案：

  一种是直接使用hashmap来进行字符和状态的存储；

  一种是单纯的数组，数组的index为字符的ASCII码

  最后进行遍历数组，找到index最小的即为第一次出现的值

- [翻转单词顺序](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/41.md)（但是单词内部不翻转）

  先把全部字符翻转，然后再翻转每一个单词

- [左旋转字符串](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/42.md)（原理同上）

  全部字符串翻转，然后根据n把两部分在进行单独翻转

- [把字符串转换成整数](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/47.md)

  这道题大前提是个整数，所以先不用考虑小数

  首先考虑正负，然后遍历每一个字符，如果是数字，然后*10+字符对应数字；最后再把数字正负加上（需要判断有没有超过最大值）

- [表示数值的字符串](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/52.md)

  需要考虑各种小数

  首先判断正负；接着遍历找到一个不是数字的字符，应该是"."或者E/e;

  如果是.  那就继续向后寻找第一个不是数字的字符，如果找到的有E，判断是不是科学计数；

  如果是E，则直接判断是不是科学计数。

## 数组

- [二维数组中，每一行都按照从左到右递增的顺序排序，查找指定的数字](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/01.md)

  每次比较右上角数字，，一层一层向里面查找

- [旋转数组的最小数字](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/06.md)

  基于数组递增的属性，采用二分法，每次找出数组中间值，如果大于前面的值则说明属于前面的数组，如果小于后面的值，说明属于后面的数组，调整指针，一次while循环

- [调整数组顺序使奇数位于偶数前面](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/11.md)

  类似于快速排序，不断交换前面的偶数和后面的奇数。

- [数组中出现次数超过一半的数字](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/26.md)

  - map保存次数
  - 排序，中间数就是
  - 选取一个数，相同就+1.不同减1，变为0就新选一个数，最后大于0的肯定我们要找的数，看下他的出现次数是否大于一半，是则成功

- [最小的k个数](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/27.md)

  - 一种就是排序

  - 一般涉及到最大或者最小几个  都可以用**最大堆**来实现

    保持一个大小为k的容器，每次判断要放入的数值是否与k中的最大值大，如果大则继续，否则替换掉最大值。

- [连续子数组的最大和](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/28.md)

  前提，数组中有整数也有负数，

  如果和为负数，直接抛弃，从当前数字开始重新算起，；

  如果大于0，则累加当前数字，并判断是否要更新最大值。

- [求从1到n的整数中1出现的次数](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/29.md)

  ？？

- [把数组排成最小的数](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/30.md)

  定义排序规则，如果s1+s2 < s2+s1  那么 s1<s2;

  最后从小到大进行排序。

- [数组中的逆序对](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/33.md)

  ？？？

- [在排序数组中出现的次数](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/35.md)

  前提是排序好的；采用二分查找法查找出第一次出现k和最后一次出现k的index，相差即为出现次数。

- [数组中只出现一次的两个数字](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/38.md)

  将所有数字异或，最后得到一个数字，因为有两个数字不同，那么结果肯定不为0，我们找出结果中的bit为不为1的index，然后依据这个index将数组分成两大部分，那么这样会把两个不同数字分到两个不同数组中；

  我们在对每一个数组中的所有数字进行异或，随后得到的就是该数组中的唯一值。

- [和为s的两个数字](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/39.md)

  前后各一个指针，相加来判断结果和移动指针。


- [和为s的连续正数序列](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/40.md)

  初始化两个值，保证小的起点小于s+1/2;  如果队列和小于s，则增大右指针，如果和大于，则删除左指针

- [数组中重复的数字](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/49.md)\

  - map
  - ??

## 数列（特殊数）

- [输入n，求斐波那契数列的第n项值。](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/07.md)

  循环

- [二进制中1的个数](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/08.md)

  ​

- [打印1到最大的n位数](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/09.md)

- [丑数](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/31.md)

  首先是 1 2 3，然后让每个数组中的数* 2 / 3/5  找出大于数组最大值的最小数作为下一个丑数

- 回文数

  从0到中位，判断是都对应相等

- [n个骰子的点数](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/43.md)

  ​

- [不用加减乘除做加法](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/46.md)

## 链表：

- 获取index位置的节点  ->二分查找

- 插入节点  头部/中间/尾部

- 删除

- [从尾到头打印链表](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/03.md)

  入栈，出栈

- [反转链表](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/13.md)

  - 遍历  使用三个指针
  - 先让nextnode 解放出来，然后递归调用，最后nextnode的next指向当前

- [在O(1)时间删除链表节点](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/10.md)

  因为是o（1），不能从head开始遍历，所以需要将当前要删除节点值改为下个节点值，next指向下个节点的下个节点

- [链表中倒数第K个节点](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/12.md)

  维护两个指针，距离是k-1

- [合并两个排序的链表](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/14.md)

  将较小的节点拼接到新联表中，当一个链表结束时，把两外一个链表直接拼接

- [顺时针打印矩阵](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/17.md)

- [两个链表的第一个公共结点](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/34.md)

  ![这里写图片描述](https://camo.githubusercontent.com/07b8cd64bbdec373d6118f13053b9bb126caa44e/687474703a2f2f696d672e626c6f672e6373646e2e6e65742f3230313530373035303635393431353139)

  必然是这种y形状结构；

  可以先算出两个链表差值，然后让较长的链表先走差值，然后开始开始每一个节点的比较

## 树：

- 二叉树/完全二叉树/满二叉树/二叉搜索树->/平衡二叉树（avl）/堆/红黑树//哈夫曼树/B+/B-/B

- 二叉树第i层上的结点数目最多为$2^{i-1}$(i≥1)

- 二叉查找树的 遍历
  - 前序/中序/后序  （递归/非递归）

    - 前序非递归：

      不断深入左节点，同时打印，并将左节点入栈，最后没有做节点可以放入时，就开始出栈，并打印他的右节点

    - 中序非递归：

      同前序，但是打印点变成了在输出右节点之前

    - 后续遍历

      大的while循环

      左子树入栈，判断右节点为空或者已经输出过，则pop；

      pust 当前出战节点，并将当前节点换位右节点（上面已经判过空，所以不为空）

  -  深度和广度遍历

- 二叉查找树的查找
  - 特定值
  - 最大值最小值
  - 前驱和后继

- 插入

- 删除

- [二叉树的深度](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/36.md)

- [判断平衡二叉树](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/37.md)

## 栈和队列

- [用两个栈实现一个队列](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/05.md)
- [栈的压入、弹出序列](https://github.com/LRH1993/android_interview/blob/master/algorithm/For-offer/19.md)





## 海量数据处理



## 排序算法

