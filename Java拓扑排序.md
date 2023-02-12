---
title: Java拓扑排序
date: 2020-10-12 20:44:17
tags:
categories:
- 数据结构与算法
---

你这个学期必须选修 numCourse 门课程，记为 0 到 numCourse-1 。

在选修某些课程之前需要一些先修课程。 例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示他们：[0,1]

给定课程总量以及它们的先决条件，请你判断是否可能完成所有课程的学习？

 ```
 示例 1:

输入: 2, [[1,0]] 
输出: true
解释: 总共有 2 门课程。学习课程 1 之前，你需要完成课程 0。所以这是可能的。
示例 2:

输入: 2, [[1,0],[0,1]]
输出: false
解释: 总共有 2 门课程。学习课程 1 之前，你需要先完成​课程 0；并且学习课程 0 之前，你还应先完成课程 1。这是不可能的。
 ```

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        // 定义一个数组来存储课程的出度
        int[] indegree = new int[numCourses];
        // 定义一个队列用来排序
        Queue<Integer> queue = new LinkedList<>();
        // 初始化所有课程的出度
        for (int[] arr : prerequisites) {
            indegree[arr[0]]++;
        }
        // 将所有出度为0的课程入队,然后将出度标记为-1,防止后面重复入队
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == 0) {
                queue.offer(i);
                indegree[i] = -1;
            }
        }
        // 开始出队,每出队一门课程,就维护出度数组,然后将所有出度为0的数组入队,将出度标记为-1,防止后面重复入队
        while (!queue.isEmpty()) {
            int course = queue.poll();
            for (int[] arr : prerequisites) {
                if (arr[1] == course) {
                    indegree[arr[0]]--;
                }
            }
            for (int i = 0; i < numCourses; i++) {
                if (indegree[i] == 0) {
                    queue.offer(i);
                    indegree[i] = -1;
                }
            }
        }
        // 遍历indegree中出度为-1的数量,也就是已经完成的课程数,和numCourses一样就返回true,否则返回false
        int ans = 0;
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == -1) {
                ans++;
            }
        }
        return ans == numCourses;
    }
}
```

