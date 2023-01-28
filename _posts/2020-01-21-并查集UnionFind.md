---
title: 并查集UnionFind
tags: 算法
---

并查集(UnionFind)主要是用来解决图论中「动态连通性」问题的，数据结构很简单，却能用来表示无向图。简单的代码如下：

```java
class UnionFind {
    int[] parent;
    int cnt;
    public UnionFind(int n) {
        parent = new int[n];
        for (int i=0; i<n; i++) {
            parent[i] = i;
        }
        cnt = n;
    }

    public int find(int x) {
        while (parent[x] != x) {
            x = parent[x];
        }
        return x;
    }

    public void union(int x, int y) {
        int px = find(x);
        int py = find(y);
        if (px == py)
            return;
        parent[px] = py;
        cnt--;
    }

    public int getCnt() {
        return cnt;
    }
}
```

可以看到代码非常简单，仅仅用一个parent数组就可以表示一个图了。使用0~n-1这n个数组下标来表示图中的n个节点，其中parent[x]=y表示x的父亲是y。find函数查找节点x的祖先节点，union函数用来合并x和y节点的祖先，也就是将两个连通图合并起来。cnt变量用来记录所有的连通图个数，每次合并后cnt减一。
但是上面的代码有个问题就是查找祖先的复杂度较高，如果所有节点刚好连成一个链表的话，查找一次时间复杂度为O(n)。
我们可以按如下两种方式进行优化：

1. **平衡性优化**：添加一个weight数组用来表示当前连通图的节点个数，合并x和y的祖先时，判断两个连通图的weight大小，把小的连到大的上。这样可以保证整个图尽量平衡，高度差不会太大。
2. **压缩路径**：在find的时候加入一行代码（**parent[x] = parent[parent[x]]**），把x的父亲设置为父亲的父亲，也就是爷爷。这样的话在查找x的祖先的过程中能把路径的长度缩小为一半。多次查找不同的节点的祖先后，整个图的高度会变成1。
   话不多说，代码优化如下：

```java
class UnionFind {
    int[] parent;
    int cnt;
    int[] weight; // weight数组记录连通图的节点个数
    public UnionFind(int n) {
        parent = new int[n];
        weight = new int[n];
        for (int i=0; i<n; i++) {
            parent[i] = i;
            weight[i] = 1;
        }
        cnt = n;
    }

    public int find(int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]]; // 压缩路径
            x = parent[x];
        }
        return x;
    }

    public void union(int x, int y) {
        int px = find(x);
        int py = find(y);
        if (px == py)
            return;
        // 合并时根据weight来合并
        if (weight[px] > weight[py]) {
            parent[py] = px;
            weight[px] += weight[py];
        } else {
            parent[px] = py;
            weight[py] += weight[px];
        }
        cnt--;
    }

    public int getCnt() {
        return cnt;
    }
}
```

UnionFind最基础的典型的使用就是判断一个图的连通分支数量。通过一个简单的例子来看看UnionFind的应用，leetcode 547题——朋友圈：[https://leetcode-cn.com/problems/friend-circles/](https://links.jianshu.com/go?to=https%3A%2F%2Fleetcode-cn.com%2Fproblems%2Ffriend-circles%2F)
