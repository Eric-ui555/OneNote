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
>
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
public String reverseWords2(String s) {
    s = s.trim();                                     // 删除首尾空格
    // i指向单词的首字母的前一个字符，j指向单词最后一个字符
    int j = s.length() - 1, i = j;
    StringBuilder res = new StringBuilder();
    // 从后向前遍历
    // 找到第一个空格，然后将单词插入res中
    // 跳过单词间空格，j 指向下个单词的尾字符
    while (i >= 0) {
        while (i >= 0 && s.charAt(i) != ' ') i--;     // 搜索首个空格
        res.append(s, i + 1, j + 1).append(" ");  // 添加单词
        while (i >= 0 && s.charAt(i) == ' ') i--;     // 跳过单词间空格
        j = i;                                        // j 指向下个单词的尾字符
    }
    return res.toString().trim();                     // 转化为字符串并返回
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

