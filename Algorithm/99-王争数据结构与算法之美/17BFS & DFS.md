[TOC]

# BFS & DFS

## 什么是搜索算法 
算法作用于数据结构, 图的搜索算法有很多, 其中最暴力的是 `深度优先` 和 `广度优先`

## 查找从s 到 t 的路径
```java
//无向图
public class Graph {

    /*
      顶点个数
    */
    private int numsOfV;

    /*
      邻接表
    */
    private LinkedList<Integer> adj[];

    public Graph(int numsOfV) {
        this.numsOfV = numsOfV;
        adj = new LinkedList[numsOfV];
        for (int i = 0; i < numsOfV; i++) {
            adj[i] = new LinkedList<>();
        }
    }

    public void addEdge(int s, int t) {
        //因为是无向图, 所以是相互链接顶点
        adj[s].add(t);
        adj[t].add(s);
    }
    /*
    *
    * 找到s到t的最短路径
    * 按层遍历
    *
    * */
    public void bfs(int s,int t) {
        if (s == t) { return; }
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(s);
        boolean[] visited = new boolean[numsOfV];
        visited[s] = true;
        int[] pre = new int[numsOfV];
        for (int i = 0; i < numsOfV; i++) {
            pre[i] = -1;
        }
        while (!queue.isEmpty()) {
            int cur = queue.poll();
            for (int i = 0; i < adj[cur].size(); i++) {
                int num = adj[cur].get(i);
                if (!visited[num]) {
                    pre[num] = cur;
                    //找到目标了, 打印路径
                    if (num == t) {
                        printPath(pre,s,num);
                        return;
                    }
                    visited[num] = true;
                    queue.offer(num);
                }
            }
        }
    }

    boolean fount = false;
    public void dfs(int s, int t) {
        if (s == t) {return;}
        boolean[] visited = new boolean[numsOfV];
        visited[s] = true;
        int[] pre = new int[numsOfV];
        for (int i = 0; i < numsOfV; i++) {
            pre[i] = -1;
        }
        recDfs(pre,visited,s,s,t);
    }
    private void recDfs(int[] pre,boolean[] visited, int cur,int s, int t) {
        if (fount) { return; }

        visited[t] = true;
        if (cur == t) {
            printPath(pre,s,t);
            return;
        }

        for (int i = 0; i < adj[cur].size(); i++) {
            int num = adj[cur].get(i);
            if (!visited[num]) {
                pre[num] = t;
                recDfs(pre,visited,num,s,t);
            }
        }
    }

    private void printPath(int[] pre, int s, int t) {
        if (s != t && pre[t] == -1) {
            printPath(pre,s,pre[t]);
        }
        System.out.println(" " + t);
    }
}
```
BFS 广度优先可以找到最短的路径, DFS就不一定了

## 参考链接

[递归 -- 数据结构与算法之美](https://time.geekbang.org/column/article/70891)