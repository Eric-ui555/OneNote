# 数组

## 121 买卖股票的最佳时机

[121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

**题目描述**

给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0` 。

**示例**

> 输入：[7,1,5,3,6,4]
> 输出：5
> 解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
>      注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。

> 输入：prices = [7,6,4,3,1]
> 输出：0
> 解释：在这种情况下, 没有交易完成, 所以最大利润为 0。

**解题思路**

需要找出给定数组中两个数字之间的最大差值（即，最大利润）。此外，第二个数字（卖出价格）必须大于第一个数字（买入价格）

一次遍历

- 时间复杂度：O(n)，只需要遍历一次。
- 空间复杂度：O(1)，只使用了常数个变量。

**参考代码**

```java
public int maxProfit(int prices[]) {
    // 历史中的最小值
    int minprice = Integer.MAX_VALUE;
    int maxprofit = 0;
    for (int i = 0; i < prices.length; i++) {
        if (prices[i] < minprice) {
            minprice = prices[i];
        } else if (prices[i] - minprice > maxprofit) {
            maxprofit = prices[i] - minprice;
        }
    }
    return maxprofit;
}
```

------

------

# 字符串

## 151.翻转字符串里的单词

[151. 反转字符串中的单词](https://leetcode.cn/problems/reverse-words-in-a-string/)

**题目描述**

给你一个字符串 `s` ，请你反转字符串中 **单词** 的顺序。

**单词** 是由非空格字符组成的字符串。`s` 中使用至少一个空格将字符串中的 **单词** 分隔开。

返回 **单词** 顺序颠倒且 **单词** 之间用单个空格连接的结果字符串。

**注意：**输入字符串 `s`中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。

**示例**

> 输入：s = "the sky is blue"  输出："blue is sky the"
>

> 输入：s = "  hello world  "  输出："world hello"  解释：反转后的字符串中不能存在前导空格和尾随空格。

> 输入：s = "a good   example"  输出："example good a"  解释：如果两个单词间有多余的空格，反转后的字符串需要将单词间的空格减少到仅有一个。

**解题思路**

思路一：

- 去除首尾以及中间多余空格
- 反转整个字符串
- 反转各个单词

思路二：双指针法

**参考代码**

```Java
/**
 * 不使用Java内置方法实现
 * <p>
 * 1.去除首尾以及中间多余空格
 * 2.反转整个字符串
 * 3.反转各个单词
 */
public String reverseWords(String s) {
    // System.out.println("ReverseWords.reverseWords2() called with: s = [" + s + "]");
    // 1.去除首尾以及中间多余空格
    StringBuilder sb = removeSpace(s);
    // 2.反转整个字符串
    reverseString(sb, 0, sb.length() - 1);
    // 3.反转各个单词
    reverseEachWord(sb);
    return sb.toString();
}

private StringBuilder removeSpace(String s) {
    // System.out.println("ReverseWords.removeSpace() called with: s = [" + s + "]");
    int start = 0;
    int end = s.length() - 1;
    while (s.charAt(start) == ' ') start++;
    while (s.charAt(end) == ' ') end--;
    StringBuilder sb = new StringBuilder();
    while (start <= end) {
        char c = s.charAt(start);
        if (c != ' ' || sb.charAt(sb.length() - 1) != ' ') {
            sb.append(c);
        }
        start++;
    }
    // System.out.println("ReverseWords.removeSpace returned: sb = [" + sb + "]");
    return sb;
}

/**
 * 反转字符串指定区间[start, end]的字符
 */
public void reverseString(StringBuilder sb, int start, int end) {
    // System.out.println("ReverseWords.reverseString() called with: sb = [" + sb + "], start = [" + start + "], end = [" + end + "]");
    while (start < end) {
        char temp = sb.charAt(start);
        sb.setCharAt(start, sb.charAt(end));
        sb.setCharAt(end, temp);
        start++;
        end--;
    }
    // System.out.println("ReverseWords.reverseString returned: sb = [" + sb + "]");
}

private void reverseEachWord(StringBuilder sb) {
    int start = 0;
    int end = 1;
    int n = sb.length();
    while (start < n) {
        while (end < n && sb.charAt(end) != ' ') {
            end++;
        }
        reverseString(sb, start, end - 1);
        start = end + 1;
        end = start + 1;
    }
}
```

```java
public String reverseWords(String s) {
    s = s.trim();                                     // 删除首尾空格
    // i指向单词的首字母的前一个字符，j指向单词最后一个字符
    int j = s.length() - 1, i = j;
    StringBuilder res = new StringBuilder();
    // 从后向前遍历
    // 找到第一个空格，然后将单词插入res中
    // 跳过单词间空格，j 指向下个单词的尾字符
    while (i >= 0) {
        while (i >= 0 && s.charAt(i) != ' ') i--;     // 搜索首个空格
        res.append(s, i + 1, j + 1).append(" ");      // 添加单词
        while (i >= 0 && s.charAt(i) == ' ') i--;     // 跳过单词间空格
        j = i;                                        // j 指向下个单词的尾字符
    }
    return res.toString().trim();                     // 转化为字符串并返回
}
```

## 28. 找出字符串中第一个匹配项的下标

[28. 找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/)

**题目描述**

给你两个字符串 `haystack` 和 `needle` ，请你在 `haystack` 字符串中找出 `needle` 字符串的第一个匹配项的下标（下标从 0 开始）。如果 `needle` 不是 `haystack` 的一部分，则返回 `-1` ****。

**示例**

> 示例 1：输入：haystack = "sadbutsad", needle = "sad" 输出：0 解释："sad" 在下标 0 和 6 处匹配。 第一个匹配项的下标是 0 ，所以返回 0 。

> 示例 2：输入：haystack = "leetcode", needle = "leeto" 输出：-1 解释："leeto" 没有在 "leetcode" 中出现，所以返回 -1 。

**解题思路**

KMP算法：字符串匹配问题

前缀表：next数组表示，记录模式串与子串（文本串）不匹配的时候，模式串应该从哪里考试重新匹配，其值为**最长相等前后缀的长度**

- **前缀是指不包含最后一个字符的所有以第一个字符开头的连续子串**。
- **后缀是指不包含第一个字符的所有以最后一个字符结尾的连续子串**。

字符串匹配过程：

**参考代码**

```Java
public int strStr(String haystack, String needle) {
    return kmp(haystack, needle);
}

/**
 * KMP数组
 *
 * @param haystack 文本串
 * @param needle   模式串
 * @return 数组索引
 */
public int kmp(String haystack, String needle) {
    // 边界条件
    if (needle.isEmpty()) return 0;
    int[] next = new int[needle.length()];
    // 获取前缀表
    getNext(next, needle);
    // 记录模式串的索引
    int j = 0;
    // 遍历文本串，i记录模式串在文本串中匹配的最后一个字符索引
    for (int i = 0; i < haystack.length(); i++) {
        // 当前字符不一致
        while (j > 0 && haystack.charAt(i) != needle.charAt(j)) {
            j = next[j - 1];
        }
        // 当前字符一致
        if (haystack.charAt(i) == needle.charAt(j)) {
            j++;
        }
        // 模式串全部匹配
        if (j == needle.length()) {
            return (i - needle.length() + 1);
        }
    }
    return -1;
}

/**
 * 计算前缀表：最长相等前后缀的长度
 *
 * @param next   前缀表
 * @param target 目标字符串
 */
public void getNext(int[] next, String target) {
    // 初始化，i指向后缀末尾位置，j指向前缀末尾位置（代表i之前包括i子串的最长相等前后缀长度）
    int j = 0;
    next[0] = 0;
    for (int i = 1; i < target.length(); i++) {  // 注意i从1开始
        // 前后缀不相同，向前回退
        while (j > 0 && target.charAt(i) != target.charAt(j)) {
            j = next[j - 1];  // 向前回退
        }
        // 找到相同的前后缀
        if (target.charAt(i) == target.charAt(j)) {
            j++;
        }
        // 将j(前缀的长度)赋给next[i]
        next[i] = j;
    }
}
```

------

# 双指针

## 15 三数之和

[15. 三数之和](https://leetcode.cn/problems/3sum/)

**题目描述**

给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请

你返回所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

**示例**

> 输入：nums = [-1,0,1,2,-1,-4]  输出：[[-1,-1,2],[-1,0,1]]  解释：  nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。  nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。  nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。  不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。  注意，输出的顺序和三元组的顺序并不重要。

**具体思路**

**Hash解法**：两层for循环可以确定a和b的值，可以使用哈希法来确定 0-(a+b) 是否在数组里出现，使用Set进行去重操作

- 时间复杂度: O(n^2)
- 空间复杂度: O(n)，额外的 set 开销

**双指针法**

- 以nums数组为例，首先将数组排序，然后有一层for循环，i从下标0的地方开始，同时定一个下标left 定义在i+1的位置上，定义下标right 在数组结尾的位置上。
- 依然还是在数组中找到 abc 使得a + b +c =0，我们这里相当于 a = nums[i]，b = nums[left]，c = nums[right]。
- 如何移动left 和right呢， 如果nums[i] + nums[left] + nums[right] > 0 就说明 此时三数之和大了，因为数组是排序后了，所以right下标就应该向左移动，这样才能让三数之和小一些。
- 如果 nums[i] + nums[left] + nums[right] < 0 说明此时 三数之和小了，left 就向右移动，才能让三数之和大一些，直到left与right相遇为止。
- 时间复杂度：O(n^2)

**参考代码**

```Java
public List<List<Integer>> threeSum(int[] nums) {
    // 边界条件
    if (nums == null || nums.length < 3)
        return null;
    // 存放结果数组
    List<List<Integer>> res = new ArrayList<>();
    int len = nums.length;
    // 数组排序
    Arrays.sort(nums);
    for (int i = 0; i < len; i++) {
        // 数组最小值大于0
        if (nums[i] > 0)
            break;
        // 去重
        if (i > 0 && nums[i] == nums[i - 1])
            continue;
        int left = i + 1;
        int right = len - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                res.add(Arrays.asList(nums[i], nums[left], nums[right]));
                // 左区间去重
                while (left < right && nums[left] == nums[left + 1]) {
                    left++;
                }
                // 右区间去重
                while (left < right && nums[right] == nums[right - 1]) {
                    right--;
                }
                left++;
                right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }
    return res;
}
```

## 16 最接近的三数之和

[16. 最接近的三数之和](https://leetcode.cn/problems/3sum-closest/)

**题目描述**

给你一个长度为 `n` 的整数数组 `nums` 和 一个目标值 `target`。请你从 `nums` 中选出三个整数，使它们的和与 `target` 最接近。

返回这三个数的和。

假定每组输入只存在恰好一个解。

**示例**

> 输入：nums = [-1,2,1,-4], target = 1
> 输出：2
> 解释：与 target 最接近的和是 2 (-1 + 2 + 1 = 2) 。

> 输入：nums = [0,0,0], target = 1
> 输出：0

**解题思路**

- 同样依照上述三数之和的解法，依靠双指针进行枚举

**参考代码**

```java
public int threeSumClosest(int[] nums, int target) {
    int n = nums.length;
    int closeSum = 10000000;
    Arrays.sort(nums);

    for (int i = 0; i < n; i++) {
        // 保证和上一次枚举的元素不相等，排除重复元素
        if (i > 0 && nums[i] == nums[i - 1]) {
            continue;
        }
        // 使用双指针进行枚举
        int left = i + 1, right = n - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            // 如果和为 target 直接返回答案
            if (sum == target) {
                return target;
            }
            // 根据差值的绝对值来更新答案
            if (Math.abs(sum - target) < Math.abs(closeSum - target)) {
                closeSum = sum;
            }
            if (sum > target) {
                // 右区间去重
                while (left < right && nums[right] == nums[right - 1]) {
                    --right;
                }
                --right;
            } else {
                // 左区间去重
                while (left < right && nums[left] == nums[left + 1]) {
                    ++left;
                }
                ++left;
            }
        }
    }
    return closeSum;
}
```

## 18 四数之和

[18. 四数之和](https://leetcode.cn/problems/4sum/)

**题目描述**

给你一个由 `n` 个整数组成的数组 `nums` ，和一个目标值 `target` 。请你找出并返回满足下述全部条件且**不重复**的四元组 `[nums[a], nums[b], nums[c], nums[d]]` （若两个四元组元素一一对应，则认为两个四元组重复）：

- `0 <= a, b, c, d < n`
- `a`、`b`、`c` 和 `d` **互不相同**
- `nums[a] + nums[b] + nums[c] + nums[d] == target`

你可以按 **任意顺序** 返回答案 。

**示例**

> 输入：nums = [1,0,-1,0,-2,2], target = 0
> 输出：[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]

> 输入：nums = [2,2,2,2,2], target = 8
> 输出：[[2,2,2,2]]

**解题思路**

双指针：继续延续三树之和双指针策略，采用两次循环遍历

剪枝操作：提出重复值

- 在确定第一个数之后，如果 `nums[i]+nums[i+1]+nums[i+2]+nums[i+3]>target`，说明此时剩下的三个数无论取什么值，四数之和一定大于 target，因此退出第一重循环；
- 在确定第一个数之后，如果 `nums[i]+nums[n−3]+nums[n−2]+nums[n−1]<target`，说明此时剩下的三个数无论取什么值，四数之和一定小于 target，因此第一重循环直接进入下一轮，枚举 `nums[i+1]`；
- 在确定前两个数之后，如果 `nums[i]+nums[j]+nums[j+1]+nums[j+2]>target`，说明此时剩下的两个数无论取什么值，四数之和一定大于 target，因此退出第二重循环；
- 在确定前两个数之后，如果 `nums[i]+nums[j]+nums[n−2]+nums[n−1]<target`，说明此时剩下的两个数无论取什么值，四数之和一定小于 target，因此第二重循环直接进入下一轮，枚举 `nums[j+1]`。

**参考代码**

```java
public static List<List<Integer>> fourSum(int[] nums, int target) {
    // 存放结果数组
    List<List<Integer>> res = new ArrayList<>();
    int len = nums.length;
    // 边界条件
    if (nums == null || nums.length < 4) return res;
    Arrays.sort(nums);
    for (int i = 0; i < len - 3; i++) {
        // 数组最小值大于0
        if (i > 0 && nums[i] == nums[i - 1]) {
            continue;
        }
        if ((long) nums[i] + nums[i + 1] + nums[i + 2] + nums[i + 3] > target) {
            break;
        }
        if ((long) nums[i] + nums[len - 3] + nums[len - 2] + nums[len - 1] < target) {
            continue;
        }
        for (int j = i + 1; j < len - 2; j++) {
            int left = j + 1;
            int right = len - 1;
            if (j > i + 1 && nums[j] == nums[j - 1]) {
                continue;
            }
            if ((long) nums[i] + nums[j] + nums[j + 1] + nums[j + 2] > target) {
                break;
            }
            if ((long) nums[i] + nums[j] + nums[len - 2] + nums[len - 1] < target) {
                continue;
            }
            while (left < right) {
                int num = nums[i] + nums[j] + nums[left] + nums[right];
                if (num == target) {
                    res.add(Arrays.asList(nums[i], nums[j], nums[left], nums[right]));
                    while (left < right && nums[left] == nums[left + 1]) left++;
                    while (left < right && nums[right] == nums[right - 1]) right--;
                    left++;
                    right--;
                } else if (num > target) {
                    right--;
                } else {
                    left++;
                }
            }
        }
    }

    return res;
}
```



# 滑动窗口

## 209. 长度最小的子数组

[209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

**题目描述**

给定一个含有 `n` ****个正整数的数组和一个正整数 `target` **。**

找出该数组中满足其总和大于等于 ****`target` ****的长度最小的 **连续子数组** `[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度**。**如果不存在符合条件的子数组，返回 `0` 。

**示例**

> 输入：target = 7, nums = [2,3,1,2,4,3]  输出：2  解释：子数组 [4,3] 是该条件下的长度最小的子数组。
>
> 输入：target = 4, nums = [1,4,4]  输出：1
>
> 输入：target = 11, nums = [1,1,1,1,1,1,1,1]  输出：0

**解题思路**

辅助队列

滑动窗口

**参考代码**

```Java
/**
 * 辅助队列
 */
public int minSubArrayLen(int target, int[] nums) {
    ArrayDeque<Integer> deque = new ArrayDeque<>();
    int sum = 0, minLen = Integer.MAX_VALUE;
    for (int num : nums) {
        deque.offerLast(num);
        sum += num;
        if (sum >= target) {
            while (sum >= target) {
                Integer first = deque.peekFirst();
                if(sum-first>=target){
                    deque.pollFirst();
                    sum -= first;
                }else{
                    break;
                }
            }
            minLen = Math.min(minLen, deque.size());
        }
    }
    if(sum < target) return 0;
    return minLen;
}
/**
 * 滑动窗口
 */
public int minSubArrayLen(int target, int[] nums) {
    int sum = 0, minLen = Integer.MAX_VALUE;
    if (nums.length == 0) {
        return 0;
    }
    int start = 0, end = 0;
    for (; end < nums.length; end++) {
        sum += nums[end];
        while (sum >= target) {
            minLen = Math.min(minLen, end - start + 1);
            sum -= nums[start];
            start++;
        }
    }
    return minLen == Integer.MAX_VALUE ? 0 : minLen;
}
```

------

# 栈与队列

## 239. 滑动窗口的最大值

[239. 滑动窗口最大值 - 力扣（LeetCode）](https://leetcode.cn/problems/sliding-window-maximum/description/)

**题目描述**：给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。返回 *滑动窗口中的最大值* 。

**示例**

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
 
输入：nums = [1], k = 1
输出：[1]
```

**解题思路**

暴力方法：遍历一遍的过程中每次从窗口中再找到最大的数值，其时间复杂度为O(n × k)，会超时；

单调队列：

- 自定义单调递减队列，会随着滑动窗口的移动动态增减队列数据，其中实现poll、add、peek三个方法；
- 分析题意可知，队列中不需要存储滑动窗口内的所有元素，最多存储最大的前两个元素即可；
- 入队时，需对队列中的元素进行遍历，将队列中小于val的元素出队；
- 弹出元素时，比较当前要弹出的数值是否等于队列出口的数值，如果相等则弹出；

**参考代码**

```java
class MyQueue {
    Deque<Integer> deque = new LinkedList<>();

    // 弹出元素时，比较当前要弹出的数值是否等于队列出口的数值，如果相等则弹出
    // 同时判断队列当前是否为空
    void poll(int val) {
        if (!deque.isEmpty() && val == deque.peek()) {
            deque.poll();
        }
    }

    // 添加元素时，如果要添加的元素大于入口处的元素，就将入口元素弹出
    // 保证队列元素单调递减
    // 比如此时队列元素3,1，2将要入队，比1大，所以1弹出，此时队列：3,2
    void add(int val) {
        while (!deque.isEmpty() && val > deque.getLast()) {
            deque.removeLast();
        }
        deque.add(val);
    }

    // 队列队顶元素始终为最大值
    int peek() {
        return deque.peek();
    }
}

/**
     * 解法一
     */
public int[] maxSlidingWindow(int[] nums, int k) {
    if (nums.length == 1) {
        return nums;
    }
    int len = nums.length - k + 1;
    // 存放结果元素的数组
    int[] res = new int[len];
    int num = 0;
    // 自定义队列
    MyQueue myQueue = new MyQueue();
    // 先将前k的元素放入队列
    for (int i = 0; i < k; i++) {
        myQueue.add(nums[i]);
    }
    res[num++] = myQueue.peek();
    for (int i = k; i < nums.length; i++) {
        // 滑动窗口移除最前面的元素，移除是判断该元素是否放入队列
        myQueue.poll(nums[i - k]);
        // 滑动窗口加入最后面的元素
        myQueue.add(nums[i]);
        // 记录对应的最大值
        res[num++] = myQueue.peek();
    }
    return res;
}
```

------

# 二叉树

## 102 二叉树的层次遍历

[102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)

**题目描述**

给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

**示例**

![https://assets.leetcode.com/uploads/2021/02/19/tree1.jpg](leetcode题解.assets/tree1.jpg)

> 输入：root = [3,9,20,null,null,15,7]  输出：[[3],[9,20],[15,7]]

**解题思路**

递归法：借助辅助数组

迭代法：借助队列，实现层次遍历

**同类型题目**

[107. 二叉树的层序遍历 II](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/)

[199. 二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)

[637. 二叉树的层平均值](https://leetcode.cn/problems/average-of-levels-in-binary-tree/)

[103. 二叉树的锯齿形层序遍历](https://leetcode.cn/problems/binary-tree-zigzag-level-order-traversal/)

**参考代码**

```Java
public List<List<Integer>> levelOrder(TreeNode root) {
    if (root == null) return new LinkedList<>();
    // 定义双端队列
    Queue<TreeNode> queue = new ArrayDeque<>();
    // 结果数组
    List<List<Integer>> res = new ArrayList<>();
    // 根节点入队
    queue.offer(root);
    // 判断队列是否为空，不为空则执行循环体
    while (!queue.isEmpty()) {
        // 获取队列长度
        int n = queue.size();
        List<Integer> layer = new ArrayList<>();
        while(n > 0){
            TreeNode p = queue.poll();
            layer.add(p.val);
            if (p.left != null) queue.offer(p.left);
            if (p.right != null) queue.offer(p.right);
            n--;
        }
        res.add(layer);
    }
    return res;
}
public List<List<Integer>> resList = new ArrayList<List<Integer>>();

public List<List<Integer>> levelOrder2(TreeNode root) {
    traversal(root,0);
    return resList;
}

/**
 * DFS--递归方式
 * @param node 结点
 * @param deep 深度
 */
public void traversal(TreeNode node, Integer deep) {
    if (node == null) return;
    deep++;

    if (resList.size() < deep) {
        //当层级增加时，list的Item也增加，利用list的索引值进行层级界定
        List<Integer> item = new ArrayList<Integer>();
        resList.add(item);
    }
    resList.get(deep - 1).add(node.val);

    traversal(node.left, deep);
    traversal(node.right, deep);
}
```

## 701.二叉搜索树中的插入操作

[701. 二叉搜索树中的插入操作](https://leetcode.cn/problems/insert-into-a-binary-search-tree/)

**题目描述**

给定二叉搜索树（BST）的根节点 `root` 和要插入树中的值 `value` ，将值插入二叉搜索树。 返回插入后二叉搜索树的根节点。 输入数据 **保证** ，新值和原始二叉搜索树中的任意节点值都不同。

**注意**，可能存在多种有效的插入方式，只要树在插入后仍保持为二叉搜索树即可。 你可以返回 **任意有效的结果** 。

**示例**

> 输入：root = [4,2,7,1,3], val = 5 
> 输出：[4,2,7,1,3,5] 

**解题思路**

递归法：

- 确定**递归函数的参数以及返回值：**遍历到待插入的位置（一定是个空节点），返回该节点
- 确定**终止条件：**如果当前节点不存在，创建节点
- 确定**单层递归的逻辑：**依托二叉搜索树的特性，找到待插入的位置

迭代法：

```java
public TreeNode insertIntoBST(TreeNode root, int val) {
    if (root == null) {
        return new TreeNode(val);
    }
    if (val < root.val) {
        root.left = insertIntoBST(root.left, val);
    } else {
        root.right = insertIntoBST(root.right, val);
    }
    return root;
}
```

```Java
public TreeNode insertIntoBST2(TreeNode root, int val) {
    if (root == null) {
        return new TreeNode(val);
    }
    TreeNode pre = null, cur = root;
    while (cur != null) {
        pre = cur;
        cur = cur.val > val ? cur.left : cur.right;
    }
    TreeNode treeNode = new TreeNode(val);
    if (pre.val > val) {
        pre.left = treeNode;
    } else {
        pre.right = treeNode;
    }
    return root;
}
```

## 669. 修剪二叉搜索树

[669. 修剪二叉搜索树 - 力扣（LeetCode）](https://leetcode.cn/problems/trim-a-binary-search-tree/description/)

**题目描述**

给你二叉搜索树的根节点 `root` ，同时给定最小边界`low` 和最大边界 `high`。通过修剪二叉搜索树，使得所有节点的值在`[low, high]`中。修剪树 **不应该** 改变保留在树中的元素的相对结构 (即，如果没有被移除，原有的父代子代关系都应当保留)。 可以证明，存在 **唯一的答案** 。

所以结果应当返回修剪好的二叉搜索树的新的根节点。注意，根节点可能会根据给定的边界发生改变。

**示例**

![img](leetcode题解.assets/trim1.jpg)

```
输入：root = [1,0,2], low = 1, high = 2
输出：[1,null,2]
```

**解题思路**

**参考代码**

## 538. 把二叉搜索树转换为累加树

[538. 把二叉搜索树转换为累加树](https://leetcode.cn/problems/convert-bst-to-greater-tree/)

**题目描述**

给出二叉 **搜索** 树的根节点，该树的节点值各不相同，请你将其转换为累加树（Greater Sum Tree），使每个节点 `node` 的新值等于原树中大于或等于 `node.val` 的值之和。

**示例1**

<img src="leetcode题解.assets/tree.png" alt="img" style="zoom:33%;" />

```
输入：[4,1,6,0,2,5,7,null,null,null,3,null,null,null,8]
输出：[30,36,21,36,35,26,15,null,null,null,33,null,null,null,8]
```

**解题思路**

遍历需要技巧，利用二叉搜索树的特性，可以看出，大于或等于 `node.val` 的值都在该节点的右子树上，因此使用递归法（RNL）进行遍历：

递归法（RNL）：

- 先遍历右节点
- 计算当前根节点的新值
- 在遍历左节点

**参考代码**

```java
class Solution {
    private int sum = 0;

    public TreeNode convertBST(TreeNode root) {
        traversal(root);
        return root;
    }

    public void traversal(TreeNode node) {
        if (node == null) return;
        traversal(node.right);
        sum += node.val;
        node.val = sum;
        traversal(node.left);
    }
}
```

## 912 排序数据

[912. 排序数组](https://leetcode.cn/problems/sort-an-array/)

**题目描述**

给你一个整数数组 `nums`，请你将该数组升序排列。

> 示例 1：
> 输入：nums = [5,2,3,1]
> 输出：[1,2,3,5]
>
> 示例 2：
> 输入：nums = [5,1,1,2,0,0]
> 输出：[0,0,1,1,2,5]

**解题思路**

**解题思路**

**参考代码**

```java
/**
 * 快速排序
 */

public int[] sortArray(int[] nums) {
    randomizedQuicksort(nums, 0, nums.length - 1);
    return nums;
}

public void randomizedQuicksort(int[] nums, int l, int r) {
    if (l < r) {
        int pos = randomizedPartition(nums, l, r);
        randomizedQuicksort(nums, l, pos - 1);
        randomizedQuicksort(nums, pos + 1, r);
    }
}

public int randomizedPartition(int[] nums, int l, int r) {
    int i = new Random().nextInt(r - l + 1) + l; // 随机选一个作为我们的主元
    swap(nums, r, i);
    return partition(nums, l, r);
}

public int partition(int[] nums, int l, int r) {
    int pivot = nums[r];
    int i = l - 1;
    for (int j = l; j <= r - 1; ++j) {
        if (nums[j] <= pivot) {
            i = i + 1;
            swap(nums, i, j);
        }
    }
    swap(nums, i + 1, r);
    return i + 1;
}
```



```Java
/**
 * 堆排序
 * 一般用数组来表示堆，下标为 i 的结点的父结点下标为(i-1)/2；其左右子结点分别为 (2i + 1)、(2i + 2)
 */

public void heapSort(int[] nums) {
    int len = nums.length - 1;
    buildMaxHeap(nums, len);
    for (int i = len; i >= 1; --i) {
        swap(nums, i, 0);
        len -= 1;
        maxHeapify(nums, 0, len);
    }
}

/**
 * 创建最大堆
 */
public void buildMaxHeap(int[] nums, int len) {
    for (int i = len / 2; i >= 0; --i) {
        maxHeapify(nums, i, len);
    }
}

/**
 * 最大堆调整
 */
public void maxHeapify(int[] nums, int i, int len) {
    // 找到第一个非叶子节点
    while ((i << 1) + 1 <= len) {
        int lSon = (i << 1) + 1;  // 左孩子节点
        int rSon = (i << 1) + 2;  // 右孩子节点
        int large;
        // 对比左孩子节点
        if (lSon <= len && nums[lSon] > nums[i]) {
            large = lSon;
        } else {
            large = i;
        }
        // 对比右孩子节点
        if (rSon <= len && nums[rSon] > nums[large]) {
            large = rSon;
        }
        // 交换
        if (large != i) {
            swap(nums, i, large);
            i = large;
        } else {
            break;
        }
    }
}

```

```java
/**
 * 选择排序
 * 从待排序序列中找到最小的值，与排好序的序列最后的元素进行置换
 */
public int[] selectSort(int[] nums) {
    int n = nums.length;
    int minIndex;
    for (int i = 0; i < n - 1; i++) {
        minIndex = i;
        // 找到最小值索引
        for (int j = i + 1; j < n; j++) {
            if (nums[minIndex] > nums[j]) {
                minIndex = j;
            }
        }
        swap(nums, minIndex, i);
    }
    return nums;
}
```

```java
/**
 * 冒泡排序
 */
public int[] bubbleSort(int[] nums) {
    int n = nums.length;
    for (int i = 0; i < n - 1; i++) {
        boolean flag = true;
        for (int j = 0; j < n - 1; j++) {
            if (nums[j] > nums[j + 1]) {
                swap(nums, j, j + 1);
                flag = false;
            }
        }
        if (flag) break;
    }
    return nums;
}

public void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

------

# 动态规划

动态规划，英文：Dynamic Programming，简称DP，如果某一问题有很多重叠子问题，使用动态规划是最有效的。

所以动态规划中每一个状态一定是由上一个状态推导出来的，**这一点就区分于贪心**，贪心没有状态推导，而是从局部直接选最优的;

解题步骤：

1. 确定dp数组（dp table）以及下标的含义
2. 确定递推公式
3. dp数组如何初始化
4. 确定遍历顺序
5. 举例推导dp数组

------

## 122 买卖股票的最佳时机2

[122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

**题目描述**

给你一个整数数组 `prices` ，其中 `prices[i]` 表示某支股票第 `i` 天的价格。

在每一天，你可以决定是否购买和/或出售股票。你在任何时候 **最多** 只能持有 **一股** 股票。你也可以先购买，然后在 **同一天** 出售。

返回 *你能获得的 **最大** 利润* 。

**示例**

> 输入：prices = [7,1,5,3,6,4]
> 输出：7
> 解释：在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
>      随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6 - 3 = 3 。
>      总利润为 4 + 3 = 7 。

**解题思路**

暴力解法：超时

动态规划法

1. 确定dp数组以及下标的含义
   - `dp[i][0]` 表示第i天持有股票所得现金。
   - `dp[i][1]` 表示第i天不持有股票所得最多现金
2. 确定递推公式
   - `dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);`
   - `dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i]);` 
3. dp数组如何初始化：`dp[0][0] = 0;` `dp[0][1] = -prices[0];`
4. 确定遍历顺序
5. 举例推导dp数组

**参考代码**

```java
/**
 * 优化空间
 * 时间复杂度：O(n)
*  空间复杂度：O(n)
 */
public int maxProfit(int[] prices) {
    int len = prices.length;
    if (len < 2) {
        return 0;
    }

    // 0：持有现金，dp[i][0] 表示第i天持有股票所得现金
    // 1：持有股票，dp[i][1] 表示第i天不持有股票所得最多现金
    // 状态转移：0 → 1 → 0 → 1 → 0 → 1 → 0
    int[][] dp = new int[len][2];

    dp[0][0] = 0;
    dp[0][1] = -prices[0];

    for (int i = 1; i < len; i++) {
        // 这两行调换顺序也是可以的
        dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
        dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
    }
    return dp[len - 1][0];
}
```

```java
/**
 * 优化空间
 * 时间复杂度：O(n)
 *  空间复杂度：O(1)
 */
public int maxProfit2(int[] prices) {
    int[] dp = new int[2];
    // 0表示持有，1表示卖出
    dp[0] = -prices[0];
    dp[1] = 0;
    for(int i = 1; i <= prices.length; i++){
        // 前一天持有; 既然不限制交易次数，那么再次买股票时，要加上之前的收益
        dp[0] = Math.max(dp[0], dp[1] - prices[i-1]);
        // 前一天卖出; 或当天卖出，当天卖出，得先持有
        dp[1] = Math.max(dp[1], dp[0] + prices[i-1]);
    }
    return dp[1];
}
```

## 123 买卖股票的最佳时机3

[123. 买卖股票的最佳时机 III](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)

**题目描述**

给定一个数组，它的第 `i` 个元素是一支给定的股票在第 `i` 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 **两笔** 交易。

**注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例**

> 输入：prices = [3,3,5,0,0,3,1,4]
> 输出：6
> 解释：在第 4 天（股票价格 = 0）的时候买入，在第 6 天（股票价格 = 3）的时候卖出，这笔交易所能获得利润 = 3-0 = 3 。
>      随后，在第 7 天（股票价格 = 1）的时候买入，在第 8 天 （股票价格 = 4）的时候卖出，这笔交易所能获得利润 = 4-1 = 3 。

**解题思路**

动态规划法

1. 确定dp数组以及下标的含义
   - `dp[i][j]`中 i表示第i天，j为 `[0 - 4]` 五个状态，`dp[i][j]`表示第i天状态j所剩最大现金。
   - 五个状态分别是：没有操作 （其实我们也可以不设置这个状态）、第一次持有股票、第一次不持有股票、第二次持有股票、第二次不持有股票
2. 确定递推公式
   - 达到`dp[i][1]`状态，有两个具体操作：
     - 操作一：第i天买入股票了，那么`dp[i][1] = dp[i-1][0] - prices[i]`
     - 操作二：第i天没有操作，而是沿用前一天买入的状态，即：`dp[i][1] = dp[i - 1][1]`
   - 其它同理
3. dp数组如何初始化：`dp[0][0] = 0;` `dp[0][1] = -prices[0];`
4. 确定遍历顺序
   - 从递归公式其实已经可以看出，一定是从前向后遍历，因为dp[i]，依靠dp[i - 1]的数值。
5. 举例推导dp数组

**参考代码**

```java
public int maxProfit(int[] prices) {
    int len = prices.length;
    // 边界判断, 题目中 length >= 1, 所以可省去
    if (prices.length == 0) return 0;

   /*
    * 定义 5 种状态:
    * 0: 没有操作, 1: 第一次买入, 2: 第一次卖出, 3: 第二次买入, 4: 第二次卖出
    */
    int[][] dp = new int[len][5];
    dp[0][1] = -prices[0];
    // 初始化第二次买入的状态是确保 最后结果是最多两次买卖的最大利润
    dp[0][3] = -prices[0];

    for (int i = 1; i < len; i++) {
        dp[i][1] = Math.max(dp[i - 1][1], -prices[i]);
        dp[i][2] = Math.max(dp[i - 1][2], dp[i - 1][1] + prices[i]);
        dp[i][3] = Math.max(dp[i - 1][3], dp[i - 1][2] - prices[i]);
        dp[i][4] = Math.max(dp[i - 1][4], dp[i - 1][3] + prices[i]);
    }

    return dp[len - 1][4];
}
```

```java
/**
 * 空间优化
 */
public int maxProfit(int[] prices) {
    int[] dp = new int[4];
    // 存储两次交易的状态就行了
    // dp[0]代表第一次交易的买入
    dp[0] = -prices[0];
    // dp[1]代表第一次交易的卖出
    dp[1] = 0;
    // dp[2]代表第二次交易的买入
    dp[2] = -prices[0];
    // dp[3]代表第二次交易的卖出
    dp[3] = 0;
    for (int i = 1; i <= prices.length; i++) {
        // 要么保持不变，要么没有就买，有了就卖
        dp[0] = Math.max(dp[0], -prices[i - 1]);
        dp[1] = Math.max(dp[1], dp[0] + prices[i - 1]);
        // 这已经是第二次交易了，所以得加上前一次交易卖出去的收获
        dp[2] = Math.max(dp[2], dp[1] - prices[i - 1]);
        dp[3] = Math.max(dp[3], dp[2] + prices[i - 1]);
    }
    return dp[3];
}
```

## 139 单词拆分

[139. 单词拆分](https://leetcode.cn/problems/word-break/)

**题目描述**

给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。

**注意：**不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。

**示例**

> 输入: s = "leetcode", wordDict = ["leet", "code"]
> 输出: true
> 解释: 返回 true 因为 "leetcode" 可以由 "leet" 和 "code" 拼接成。

> 输入: s = "applepenapple", wordDict = ["apple", "pen"]
> 输出: true
> 解释: 返回 true 因为 "applepenapple" 可以由 "apple" "pen" "apple" 拼接成。
>      注意，你可以重复使用字典中的单词。

> **提示：**
>
> - `1 <= s.length <= 300`
> - `1 <= wordDict.length <= 1000`
> - `1 <= wordDict[i].length <= 20`
> - `s` 和 `wordDict[i]` 仅由小写英文字母组成
> - `wordDict` 中的所有字符串 **互不相同**

**解题思路**

动态规划:

1、确定dp数组以及下标的含义

定义 `dp[i]`表示字符串 s 前 i个字符组成的字符串 `s[0..i−1]` 是否能被空格拆分成若干个字典中出现的单词

2、确定递推公式

`dp[i]=dp[j]&& check(s[j..i−1])`

从前往后计算考虑转移方程，**每次转移的时候我们需要枚举包含位置 i−1 的最后一个单词，看它是否出现在字典中以及除去这部分的字符串是否合法即可**。

公式化来说，我们需要枚举 `s[0..i−1]`中的分割点 j ，看 `s[0..j−1]` 组成的字符串 `s1` （默认 j=0 时 `s1`为空串）和 s[j..i−1] 组成的字符串 `s2`是否都合法，如果两个字符串均合法，那么按照定义 `s1` 和 `s2` 拼接成的字符串也同样合法。

**时间复杂度：O(n^2)**

**空间复杂度：O(n)**

**参考代码**

```java
public boolean wordBreak(String s, List<String> wordDict) {
    // 数组去重
    Set<String> wordDictSet = new HashSet<>(wordDict);
    // dp数组
    boolean[] dp = new boolean[s.length() + 1];
    // dp数组初始化
    dp[0] = true;
    // 遍历
    for (int i = 1; i <= s.length(); i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && wordDictSet.contains(s.substring(j, i))) {
                dp[i] = true;
                break;
            }
        }
    }
    return dp[s.length()];
}
```

------

## 746 使用最小花费爬楼梯

[746. 使用最小花费爬楼梯](https://leetcode.cn/problems/min-cost-climbing-stairs/)

**题目描述**

给你一个整数数组 `cost` ，其中 `cost[i]` 是从楼梯第 `i` 个台阶向上爬需要支付的费用。一旦你支付此费用，即可选择向上爬一个或者两个台阶。

你可以选择从下标为 `0` 或下标为 `1` 的台阶开始爬楼梯。

请你计算并返回达到楼梯顶部的最低花费。

**示例**

> 输入：cost = [10,15,20]
> 输出：15
> 解释：你将从下标为 1 的台阶开始。支付 15 ，向上爬两个台阶，到达楼梯顶部。总花费为 15 。

> 输入：cost = [1,100,1,1,1,100,1,1,100,1]
> 输出：6
> 解释：你将从下标为 0 的台阶开始。
> - 支付 1 ，向上爬两个台阶，到达下标为 2 的台阶。
> - 支付 1 ，向上爬两个台阶，到达下标为 4 的台阶。
> - 支付 1 ，向上爬两个台阶，到达下标为 6 的台阶。
> - 支付 1 ，向上爬一个台阶，到达下标为 7 的台阶。
> - 支付 1 ，向上爬两个台阶，到达下标为 9 的台阶。
> - 支付 1 ，向上爬一个台阶，到达楼梯顶部。
> 总花费为 6 。

**解题思路**

定义dp数组：dp[i] 的值为登上第i阶阶梯所要支付的代价；

`dp[i] = min{dp[i-i]+cost[i-1], dp[i-2]+cost[i-2]}`

`dp[0] = dp[1] = 0`

**参考代码**

```java
/**
* 时间复杂度：O(n)
* 空间复杂度：O(n)
*/
public static int minCostClimbingStairs(int[] cost) {
    int[] dp = new int[cost.length + 1];
    dp[0] = 0;
    dp[1] = 0;
    for (int i = 2; i <= cost.length; i++) {
        dp[i] = Math.min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);
    }
    System.out.println(Arrays.toString(dp));
    return dp[cost.length];
}

/**
* 时间复杂度：O(n)
* 空间复杂度：O(1)
*/
public static int minCostClimbingStairs2(int[] cost) {
    int pre = 0, cur = 0;
    for (int i = 2; i <= cost.length; i++) {
        int next = Math.min(cur + cost[i - 1], pre + cost[i - 2]);
        pre = cur;
        cur = next;
    }
    return cur;
}
```

------

## 322 零钱兑换

[322. 零钱兑换](https://leetcode.cn/problems/coin-change/)

**题目描述**

给你一个整数数组 `coins` ，表示不同面额的硬币；以及一个整数 `amount` ，表示总金额。

计算并返回可以凑成总金额所需的 **最少的硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。

你可以认为每种硬币的数量是无限的。

**示例**

> 输入：coins = [1, 2, 5], amount = 11
> 输出：3 
> 解释：11 = 5 + 5 + 1

> 输入：coins = [2], amount = 3
> 输出：-1

> 输入：coins = [1], amount = 0
> 输出：0

> **提示：**
>
> - `1 <= coins.length <= 12`
> - `1 <= coins[i] <= 2^31 - 1`
> - `0 <= amount <= 104`

**解题思路**

1、确定dp数组以及下标的含义

dp[i]：表示

**总结**

1、确定dp数组以及下标的含义：dp[j]：凑成总金额j的货币组合数为dp[j]

2、确定递推公式：`dp[j] = Math.min(dp[i], dp[i - coins[j]] + 1);;`

3、dp数组如何初始化：`dp[0]=1`

4、确定遍历顺序

**参考代码**

```java
public class Solution {
    public int coinChange(int[] coins, int amount) {
        int max = amount + 1;
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, max);
        dp[0] = 0;
        for (int i = 1; i <= amount; i++) {
            for (int j = 0; j < coins.length; j++) {
                if (coins[j] <= i) {
                    dp[i] = Math.min(dp[i], dp[i - coins[j]] + 1);
                }
            }
        }
        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```

## 62 不同路径

[62. 不同路径](https://leetcode.cn/problems/unique-paths/)

**题目描述**

一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

**示例**

> 输入：m = 3, n = 7
> 输出：28

> 输入：m = 3, n = 2
> 输出：3
> 解释：
> 从左上角开始，总共有 3 条路径可以到达右下角。
> 1. 向右 -> 向下 -> 向下
> 2. 向下 -> 向下 -> 向右
> 3. 向下 -> 向右 -> 向下

**解题思路**

树的深度优先遍历：

- 由于机器人只能向下或向右，可以根据其方向画出一棵树
- 会超出时间限制

动态规划

- `dp[i][j]`表示到行号为i，列号为j是的路径数量；

- 递推方程：`dp[i][j] = dp[i - 1][j] + dp[i][j - 1];`
- 初始化：要对第一行和第一列进行初始化，初始值都为1
- 遍历：先遍历行在遍历列或相反；
- 时间复杂度和空间复杂度：O(m*n)

**参考代码**

```java
/**
 * 深度优先遍历：超出时间限制
 */
public int uniquePaths(int m, int n) {
    return traversal(1, 1, m, n);
}

public int traversal(int i, int j, int m, int n) {
    if (i > m || j > n) return 0;
    if (i == m && j == n) return 1;
    return traversal(i + 1, j, m, n) + traversal(i, j + 1, m, n);
}
```

```java
/**
 * 动态规划算法
 */
public static int uniquePaths2(int m, int n) {
    int[][] dp = new int[m][n];
    for (int i = 0; i < m; i++) dp[i][0] = 1;
    for (int j = 0; j < n; j++) dp[0][j] = 1;
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
        }
    }
    return dp[m - 1][n - 1];
}
```

## 63 不同路径2

**题目描述**

一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish”）。

现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？

网格中的障碍物和空位置分别用 `1` 和 `0` 来表示。

**示例**

![img](leetcode题解.assets/robot1.jpg)

> 输入：`obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]`
> 输出：2
> 解释：3x3 网格的正中间有一个障碍物。
> 从左上角到右下角一共有 2 条不同的路径：
>
> 1. 向右 -> 向右 -> 向下 -> 向下
> 2. 向下 -> 向下 -> 向右 -> 向右

![img](leetcode题解.assets/robot2.jpg)

> 输入：`obstacleGrid = [[0,1],[0,0]]`
> 输出：1

> **提示：**
>
> - `m == obstacleGrid.length`
> - `n == obstacleGrid[i].length`
> - `1 <= m, n <= 100`
> - `obstacleGrid[i][j]` 为 `0` 或 `1`

**解题思路**

- `dp[i][j]` ：表示从（0 ，0）出发，到(i, j) 有`dp[i][j]`条不同的路径。
- 递推公式：`dp[i][j] = dp[i - 1][j] + dp[i][j - 1]`
- 初始化：主要初始化第一行第一列，遇到障碍则返回
- 遍历：从左到右，从上到下

**参考代码**

```java
public int uniquePathsWithObstacles(int[][] obstacleGrid) {
    int m = obstacleGrid.length;
    int n = obstacleGrid[0].length;
    // 若起点或终点都是障碍，则输出0
    if (obstacleGrid[0][0] == 1 || obstacleGrid[m - 1][n - 1] == 1) {
        return 0;
    }
    int[][] dp = new int[m][n];
    // 初始化
    for (int i = 0; i < n && obstacleGrid[0][i] == 0; i++) dp[0][i] = 1;
    for (int j = 0; j < m && obstacleGrid[j][0] == 0; j++) dp[j][0] = 1;
    // 遍历
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            if (obstacleGrid[i][j] == 0) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
    }
    return dp[m - 1][n - 1];
}
```

