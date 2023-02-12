---
title: leetcode题集
date: 2020-05-17 14:19:37
tags:
categories:
- LeetCode
---

## 动态规划

***最长公共子串***

**题目描述**

有两个字符串（可能包含空格），请找出其中最长的公共连续子串，输出其长度。

**示例1**

```
输入：str1="abcde"，str2="abcde"
输出：5
```

**示例2**

```
输入：str1="abcdefg"，str2="acdaefg"
输出：3
```

**图解**

两个串相同

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/最长公共子串1.png" width="160px"/>

两个串不同

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/最长公共子串3.png" width="180px"/>

**代码**

```java
public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String str1 = sc.next();
        String str2 = sc.next();
        int[][] dp = new int[1000][1000];
        int max = 0;
        for (int i = 0; i < str1.length(); i++) {
            for (int j = 0; j < str2.length(); j++) {
                if (str1.charAt(i) == str2.charAt(j)) {
                    if (i == 0 || j == 0) {
                        dp[i][j] = 1;
                    } else {
                        dp[i][j] = dp[i - 1][j - 1] + 1;
                    }
                } else {
                    dp[i][j] = 0;
                }
                if (dp[i][j] > max) {
                    max = dp[i][j];
                }
            }
        }
        System.out.println(max);
    }
}
```

**参考**

+ [最长公共子串（图文版）](https://www.cnblogs.com/dgwblog/p/11727658.html)

***最长公共子序列***

给定两个字符串 text1 和 text2，返回这两个字符串的最长公共子序列的长度。

一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。两个字符串的「公共子序列」是这两个字符串所共同拥有的子序列。

若这两个字符串没有公共子序列，则返回 0。

```
输入：text1 = "abcde", text2 = "ace" 
输出：3  
解释：最长公共子序列是 "ace"，它的长度为 3。
示例 2:

输入：text1 = "abc", text2 = "abc"
输出：3
解释：最长公共子序列是 "abc"，它的长度为 3。
示例 3:

输入：text1 = "abc", text2 = "def"
输出：0
解释：两个字符串没有公共子序列，返回 0。
```

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[m][n];
    }
}
```

***01背包***

**题目描述**

现在有一个背包（容器），它的体积（容量）为V，现在有N种物品（每个物品只有一个），每个物品的价值W[i]和占用空间C[i]都会由输入给出，现在问这个背包最多能携带总价值多少的物品？

**输入**

```
70 3
71 100
69 1
1 2
```

**输出**

```
3
```

**代码1**

```java
public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int v = sc.nextInt(); 
        int n = sc.nextInt(); 
        int[] w = new int[n + 1];
        int[] c = new int[n + 1];
        int[][] dp = new int[n + 1][v + 1];
        for (int i = 1; i <= n; i++) {
            w[i] = sc.nextInt();
            c[i] = sc.nextInt();
        }
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j <= v; j++) {
                if (j < w[i]) {
                    dp[i][j] = dp[i - 1][j];
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - w[i]] + c[i]);
                }
            }
        }
        System.out.println(dp[n][v]);
    }
}
```

**代码2**

```java
public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int v = sc.nextInt();
        int n = sc.nextInt();
        int[] w = new int[n + 1];
        int[] c = new int[n + 1];
        int[] dp = new int[v + 1];
        for (int i = 1; i <= n; i++) {
            w[i] = sc.nextInt();
            c[i] = sc.nextInt();
        }
        for (int i = 1; i <= n; i++) {
            for (int j = v; j >= 0; j--) {
                if (j >= w[i]) {
                    dp[j] = Math.max(dp[j], dp[j - w[i]] + c[i]);
                }
            }
        }
        System.out.println(dp[v]);
    }
}
```

***零钱兑换***

给定不同面额的硬币和一个总金额。写出函数来计算可以凑成总金额的硬币组合数。假设每一种面额的硬币有无限个。

```
输入: amount = 5, coins = [1, 2, 5]
输出: 4
解释: 有四种方式可以凑成总金额:
5=5
5=2+2+1
5=2+1+1+1
5=1+1+1+1+1
```

```java
class Solution {
    public int change(int amount, int[] coins) {
        int[] dp = new int[amount + 1];
        dp[0] = 1;
        for (int i = 0; i < coins.length; i++) {
            for (int j = coins[i]; j <= amount; j++) {
                dp[j] = dp[j] + dp[j - coins[i]];
            }
        }
        return dp[amount];
    }
}
```

***最长上升子序列***

给定一个无序的整数数组，找到其中最长上升子序列的长度。

```
输入: [10,9,2,5,3,7,101,18]
输出: 4 
解释: 最长的上升子序列是 [2,3,7,101]，它的长度是 4。
```

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        if (nums.length == 0) {
            return 0;
        }
        int[] dp = new int[nums.length];
        Arrays.fill(dp, 1);
        int max = -1;
        for (int i = 0; i < nums.length; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            max = Math.max(max, dp[i]);
        }
        return max;
    }
}
```

***乘积最大子数组***

给你一个整数数组 nums ，请你找出数组中乘积最大的连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

```
示例 1:

输入: [2,3,-2,4]
输出: 6
解释: 子数组 [2,3] 有最大乘积 6。
示例 2:

输入: [-2,0,-1]
输出: 0
解释: 结果不能为 2, 因为 [-2,-1] 不是子数组。
```

```java
class Solution {
    public int maxProduct(int[] nums) {
        int max = nums[0], min = nums[0], ans = nums[0];
        for (int i = 1; i < nums.length; i++) {
            int temp = max;
            max = Math.max(Math.max(max * nums[i], min * nums[i]), nums[i]);
            min = Math.min(Math.min(temp * nums[i], min * nums[i]), nums[i]);
            ans = Math.max(ans, max);
        }
        return ans;
    }
}
```

***单词拆分***

给定一个非空字符串 s 和一个包含非空单词的列表 wordDict，判定 s 是否可以被空格拆分为一个或多个在字典中出现的单词。

```
示例 1：

输入: s = "leetcode", wordDict = ["leet", "code"]
输出: true
解释: 返回 true 因为 "leetcode" 可以被拆分成 "leet code"。
示例 2：

输入: s = "applepenapple", wordDict = ["apple", "pen"]
输出: true
解释: 返回 true 因为 "applepenapple" 可以被拆分成 "apple pen apple"。
     注意你可以重复使用字典中的单词。
示例 3：

输入: s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
输出: false
```

```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        int n = s.length();
        boolean[] dp = new boolean[n + 1];
        dp[0] = true;
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j < i; j++) {
                if (dp[j] && wordDict.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[n];
    }
}
```

## 数据结构

***最小栈***

设计一个支持 push ，pop ，top 操作，并能在常数时间内检索到最小元素的栈。

push(x) —— 将元素 x 推入栈中。
pop() —— 删除栈顶的元素。
top() —— 获取栈顶元素。
getMin() —— 检索栈中的最小元素。

```
输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.getMin();   --> 返回 -2.
```

```java
class MinStack {
    Deque<Integer> queue;
    Queue<Integer> queue_min;
    
    public MinStack() {
        queue = new LinkedList<>();
        queue_min = new PriorityQueue<>();
    }
    
    public void push(int x) {
        queue.addFirst(x);
        queue_min.add(x);
    }
    
    public void pop() {
        int x = queue.removeFirst();
        queue_min.remove(x);
    }
    
    public int top() {
        return queue.getFirst();
    }
    
    public int getMin() {
        return queue_min.element(); 
    }
}
```

***简单表达式计算***

给定一个合法的表达式字符串，其中只包含非负整数、加法、减法、乘法以及除法符号（不会有括号），例如`7+3*4*5+2+4-3-1`，请写程序计算该表达式的结果并输出；

```
输入例子1:
7+3*4*5+2+4-3-1
2-3*1
输出例子1:
69
-1
```

```java
// 双向队列
public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String input = sc.next();
        Deque<Integer> nums = new LinkedList<>();
        Deque<Character> opes = new LinkedList<>();
        int sum = 0;
        for (int i = 0; i < input.length(); i++) {
            char c = input.charAt(i);
            if (c >= '0' && c <= '9') {
                sum = sum * 10 + (c - '0');
            } else {
                nums.offerFirst(sum);
                sum = 0;
                if (opes.isEmpty()) {
                    opes.offerFirst(c);
                } else {
                    if (opes.peekFirst() == '*') {
                        opes.pollFirst();
                        int num2 = nums.pollFirst();
                        int num1 = nums.pollFirst();
                        int res = num1 * num2;
                        nums.offerFirst(res);
                    } else if (opes.peekFirst() == '/') {
                        opes.pollFirst();
                        int num2 = nums.pollFirst();
                        int num1 = nums.pollFirst();
                        int res = num1 / num2;
                        nums.offerFirst(res);
                    }
                    opes.offerFirst(c);
                }
            }
        }
        nums.offerFirst(sum);
        if (opes.peekFirst() != null) {
            if (opes.peekFirst() == '*') {
                opes.pollFirst();
                int num2 = nums.pollFirst();
                int num1 = nums.pollFirst();
                int res = num1 * num2;
                nums.offerFirst(res);
            } else if (opes.peekFirst() == '/') {
                opes.pollFirst();
                int num2 = nums.pollFirst();
                int num1 = nums.pollFirst();
                int res = num1 / num2;
                nums.offerFirst(res);
            }
        }
        while (nums.size() > 1) {
            int num1 = nums.pollLast();
            int num2 = nums.pollLast();
            char c = opes.pollLast();
            if (c == '+') {
                int res = num1 + num2;
                nums.offerLast(res);
            } else if (c == '-') {
                int res = num1 - num2;
                nums.offerLast(res);
            }
        }
        System.out.println(nums.peekFirst());
    }
}
```

## DFS

***岛屿数量***

给你一个由 '1'（陆地）和 '0'（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

```
输入:
[
['1','1','0','0','0'],
['1','1','0','0','0'],
['0','0','1','0','0'],
['0','0','0','1','1']
]
输出: 3
解释: 每座岛屿只能由水平和/或竖直方向上相邻的陆地连接而成。
```

```java
// DFS
class Solution {
    public int numIslands(char[][] grid) {
        int count = 0;
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[0].length; j++) {
                if (grid[i][j] == '1') {
                    func(grid, i, j);
                    count++;
                }
            }
        }
        return count;
    }

    public void func(char[][] grid, int x, int y) {
        if (grid[x][y] != '1') {
            return;
        }
        grid[x][y] = '2';
        int[][] direction = {{0, -1}, {0, 1}, {-1, 0}, {1, 0}};
        for (int[] d : direction) {
            int nx = x + d[0];
            int ny = y + d[1];
            if (nx < 0 || nx >= grid.length || ny < 0 || ny >= grid[0].length) {
                continue;
            }
            func(grid, nx, ny);
        }
    }
}
```

***单词搜索***

给定一个二维网格和一个单词，找出该单词是否存在于网格中。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

```
示例:

board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

给定 word = "ABCCED", 返回 true
给定 word = "SEE", 返回 true
给定 word = "ABCB", 返回 false
```

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[0].length; j++) {
                if (board[i][j] == word.charAt(0)) {
                    if (dfs(board, word, i, j, 0)) {
                        return true;
                    }
                }
            }
        }
        return false;
    }

    public boolean dfs(char[][] board, String word, int i, int j, int k) {
        if (k == word.length()) {
            return true;
        }
        if (i < 0 || i >= board.length || j < 0 || j >= board[0].length) {
            return false;
        }
        if (board[i][j] != word.charAt(k)) {
            return false;
        }
        char t = board[i][j];
        board[i][j] = '0';
        boolean res = dfs(board, word, i + 1, j, k + 1)
                || dfs(board, word, i - 1, j, k + 1)
                || dfs(board, word, i, j + 1, k + 1)
                || dfs(board, word, i, j - 1, k + 1);
        board[i][j] = t;
        return res;
    }
}
```

***重建二叉树***

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

```
例如，给出

前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
返回如下的二叉树：

    3
   / \
  9  20
    /  \
   15   7
```

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        List<Integer> pre = new ArrayList<>();
        for (int i = 0; i < preorder.length; i++) {
            pre.add(preorder[i]);
        }
        List<Integer> in = new ArrayList<>();
        for (int i = 0; i < inorder.length; i++) {
            in.add(inorder[i]);
        }
        return func(pre, in);
    }

    public TreeNode func(List<Integer> pre, List<Integer> in) {
        if (pre.size() == 0) {
            return null;
        }
        int val = pre.get(0);
        TreeNode root = new TreeNode(val);
        int ans = in.indexOf(val);
        root.left = func(pre.subList(1, ans + 1), in.subList(0, ans));
        root.right = func(pre.subList(ans + 1, pre.size()), in.subList(ans + 1, in.size()));
        return root;
    }
}
```

## 常用算法

***长度最小的子数组***

给定一个含有 n 个正整数的数组和一个正整数 s ，找出该数组中满足其和 ≥ s 的长度最小的 连续 子数组，并返回其长度。如果不存在符合条件的子数组，返回 0。

```
输入：s = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

```java
// 滑动窗口
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        int n = nums.length;
        if (n == 0) {
            return 0;
        }
        int ans = Integer.MAX_VALUE;
        int start = 0, end = 0;
        int sum = 0;
        while (end < n) {
            sum += nums[end];
            while (sum >= s) {
                ans = Math.min(ans, end - start + 1);
                sum -= nums[start];
                start++;
            }
            end++;
        }
        return ans == Integer.MAX_VALUE ? 0 : ans;
    }
}
```

***亲戚***

规定：x和y是亲戚，y和z是亲戚，那么x和z也是亲戚。如果x,y是亲戚，那么x的亲戚都是y的亲戚，y的亲戚也都是x的亲戚。

```
输入格式
第一行：三个整数n,m,p，（n<=5000,m<=5000,p<=5000），分别表示有n个人，m个亲戚关系，询问p对亲戚关系。
以下m行：每行两个数Mi，Mj，1<=Mi，Mj<=N，表示Mi和Mj具有亲戚关系。
接下来p行：每行两个数Pi，Pj，询问Pi和Pj是否具有亲戚关系。

6 5 3
1 2
1 5
3 4
5 2
1 3
1 4
2 3
5 6

输出格式
P行，每行一个’Yes’或’No’。表示第i个询问的答案为“具有”或“不具有”亲戚关系。

Yes
Yes
No
```

```java
// 并查集
public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        int m = sc.nextInt();
        int p = sc.nextInt();
        int[] nums = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            nums[i] = i;
        }
        for (int i = 0; i < m; i++) {
            int x = sc.nextInt();
            int y = sc.nextInt();
            union(nums, x, y);
        }
        for (int i = 0; i < p; i++) {
            int x = sc.nextInt();
            int y = sc.nextInt();
            if (find(nums, x, y)) {
                System.out.println("Yes");
            } else {
                System.out.println("No");
            }
        }
    }

    public static boolean find(int[] nums, int x, int y) {
        return nums[x] == nums[y];
    }

    public static void union(int[] nums, int x, int y) {
        int num1 = nums[x];
        int num2 = nums[y];
        if (num1 != num2) {
            for (int i = 0; i < nums.length; i++) {
                if (nums[i] == num1) {
                    nums[i] = num2;
                }
            }
        }
    }
}
```

参考：[并查集(Java实现)](https://www.cnblogs.com/noKing/p/8018609.html)

***无重复字符的最长子串***

给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。

```
示例 1:
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。

示例 2:
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
示例 3:

输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

```java
// 滑动窗口
class Solution {
    public int lengthOfLongestSubstring(String s) {
        List<Character> list = new ArrayList<>();
        int start = 0, end = 0, max = 0, n = s.length();
        while (end < n) {
            char c = s.charAt(end);
            if (list.contains(c)) {
                list.remove(0);
                start++;
            } else {
                list.add(c);
                max = Math.max(max, end - start + 1);
                end++;
            }
        }
        return max;
    }
}
```

***LRU缓存机制***

运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

获取数据 get(key) - 如果关键字 (key) 存在于缓存中，则获取关键字的值（总是正数），否则返回 -1。
写入数据 put(key, value) - 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字/值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

```
LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // 返回  1
cache.put(3, 3);    // 该操作会使得关键字 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.put(4, 4);    // 该操作会使得关键字 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4
```

```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }
    
    public int get(int key) {
        if (super.get(key) == null) {
            return -1;
        } else {
            return super.get(key);
        }
    }
    
    public void put(int key, int value) {
        super.put(key, value);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > capacity;
    }
}
```

***组合总和***

给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的数字可以无限制重复被选取。

```
示例 1：

输入：candidates = [2,3,6,7], target = 7,
所求解集为：
[
  [7],
  [2,2,3]
]

示例 2：

输入：candidates = [2,3,5], target = 8,
所求解集为：
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]
```

```java
class Solution {
    private Deque<Integer> list = new LinkedList<>();
    private List<List<Integer>> res = new LinkedList<>();

    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);
        func(0, 0, target, candidates);
        return res;
    }

    private void func(int k, int sum, int target, int[] nums) {
        if (sum == target) {
            res.add(new LinkedList<>(list));
            return;
        } else if (sum > target || k >= nums.length || target - sum < nums[k]) {
            return;
        } 
        list.offerLast(nums[k]);
        func(k, sum + nums[k], target, nums);         
        list.pollLast();
        func(k + 1, sum, target, nums); 
    }
}
```

***原地反转链表***

反转一个单链表。

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

```java
class Node {
    int val;
    Node next;
    Node(int x) { 
    	val = x; 
    }
}

public class Solution {
    public Node reverse(Node root) {
        Node next = null;
        Node prev = null;
        while (root != null) {
            next = root.next;
            root.next = prev;
            prev = root;
            root = next;
        }
        return prev;
    }
}
```

## 贪心

***两地调度***

公司计划面试 2N 人。第 i 人飞往 A 市的费用为 costs[i][0]，飞往 B 市的费用为 costs[i][1]。

返回将每个人都飞到某座城市的最低费用，要求每个城市都有 N 人抵达。

```
输入：[[10,20],[30,200],[400,50],[30,20]]
输出：110
解释：
第一个人去 A 市，费用为 10。
第二个人去 A 市，费用为 30。
第三个人去 B 市，费用为 50。
第四个人去 B 市，费用为 20。

最低总费用为 10 + 30 + 50 + 20 = 110，每个城市都有一半的人在面试。
```

```java
class Solution {
    public int twoCitySchedCost(int[][] costs) {
        int res = 0;
        int[] temp = new int[costs.length];
        for (int i = 0; i < costs.length; i++) {
            temp[i] = costs[i][1] - costs[i][0];
            res = res + costs[i][0];
        }
        Arrays.sort(temp);
        for (int i = 0; i < costs.length / 2; i++) {
            res = res + temp[i];
        }
        return res;
    }
}
```

## DFS

***资金匹配***

现有一笔借款请求（target），借款需要从可用资金列表中（n笔）匹配k笔资金，请问有多少种匹配方案。

例如：借款金额10万，可用资金有8、7、3、2、1，则一共有3种匹配方案，分别（8,2）（7,3）（7,2,1）。

请将上述过程抽象成算法，并实现kSum函数。

```java
import java.util.*;
public class Main {
    // 匹配方案个数
    public static int count = 0;
    public static void kSum(List<Integer> numbers, int target) {
        //实现逻辑
        Collections.sort(numbers);
        dfs(numbers, 0, target);
    }
    public static void dfs(List<Integer> numbers, int begin, int remain) {
        if (remain == 0) {
            count++;
            return;
        }
        for (int i = begin; i < numbers.size(); i++) {
            if (numbers.get(i) > remain) {
                break;
            }
            dfs(numbers, i + 1, remain - numbers.get(i));
        }
    }
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String source = sc.nextLine();
        String[] str = source.split(",");
        List<Integer> in = new ArrayList<Integer>();
        for (int i = 0; i < str.length; i++)
            in.add(Integer.parseInt(str[i]));
        int target = Integer.parseInt(sc.nextLine());
        kSum(in, target);
        System.out.println(count);
    }
}
```

***二叉树的所有路径***

给定一个二叉树，返回所有从根节点到叶子节点的路径。

```
输入:

   1
 /   \
2     3
 \
  5

输出: ["1->2->5", "1->3"]
```

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<String> binaryTreePaths(TreeNode root) {
        List<String> res = new ArrayList<>();
        func(root, "", res);
        return res;
    }

    public void func(TreeNode root, String cur, List<String> res) {
        if (root == null) {
            return;
        }
        StringBuilder sb = new StringBuilder(cur);
        sb.append(root.val);
        if (root.left == null && root.right == null) {
            res.add(sb.toString());
        } else {
            sb.append("->");
            func(root.left, sb.toString(), res);
            func(root.right, sb.toString(), res);
        }
    }
}
```