**Python算法模板及注释(updating)**

# 素数筛

#### 埃氏筛。时间复杂度：O(nloglogn)

```python
# n = int(input('Please input a range:'))
n = 10 ** 8
list_prime = [True] * (n + 3)
end = int(n ** 0.5) + 1
for i in range(2, end):  # 从2开始，将已知的素数作为1~n各数的质因数，筛去这些合数
    if list_prime[i]:
        for j in range(i ** 2, n + 1, i):  # 因为是最小质因数，所以对于某个质数i，它所筛去的合数不可能小于 i ** 2
            list_prime[j] = False
```

#### 欧拉筛。时间复杂度：O(n)

```python
# n = int(input('Please input a range:'))
n = 10 ** 8
num_list = [True] * (n + 3)
prime_list = []  # 2单独处理，可以省去一半的操作 （偶数）
for i in range(3, n + 1, 2):
    if num_list[i]:
        prime_list.append(i)
    for j in prime_list:
        if j * i > n:
            break
        num_list[j * i] = False
        if i % j == 0:  # 如果i是j的倍数，那么 i * 其他大于 j 质数 在筛选 j 的倍数时候肯定会筛到，所以不用考虑
            break
# print([2] + prime_list)
```

在一般问题中，两者速度相差不大，甚至埃氏筛可能更快。



# **并查集**

#### 模板及注释

```python
"""
并查集被很多OIer认为是最简洁而优雅的数据结构之一，主要用于解决一些元素分组的问题。它管理一系列不相交的集合，并支持两种操作：

合并（Union）：把两个不相交的集合合并为一个集合。
查询（Find）：查询两个元素是否在同一个集合中。

并查集的重要思想在于，用集合中的一个元素代表集合。
"""


class Dsu:
    def __init__(self, size):  # 初始化
        self.pa = list(range(size))  # self.pa[x] 表示 x 的父节点；每个节点的父节点只有一个
        self.rank = [1] * size  # self.rank[x] 表示以 x 为父节点的子树（或者整树）的深度，初始化为 1

    def find_1(self, x):  # 向上查询，查找根节点
        return x if self.pa[x] == x else self.find_1(self.pa[x])  # 根节点（代表元素）的父节点是它本身；递归查询
    # 若两个元素的根节点相同，则这两个元素在同一个集合中

    def union_1(self, x, y):  # 合并：找到两个集合中的代表元素，并且将一个代表元素的根节点设置为另一个代表元素
        self.pa[self.find_1(x)] = self.find_1(y)

# 操作优化
# 优化 1：路径压缩。缩短从任意元素查找到根元素的路径长度。
# 大部分题目中不关注树的形态，只关注根节点。因此可以在查询操作中将访问过的每一个节点都指向根节点。
    def find_2(self, x):
        if self.pa[x] != x:
            self.pa[x] = self.find_2(self.pa[x])  # 与 find_1 不同之处在于此处将父节点重新赋值
        return self.pa[x]

# 优化2：启发式合并。在合并两个集合时将 “深度较小的集合的根节点” 的 “父节点” 设为 “深度较大集合的根节点”
# 需要设置变量 rank 纪录集合树的深度
    def union_2(self, x, y):
        x, y = self.find_2(x), self.find_2(y)  # 找到两个节点对应的根节点
        if x == y:
            return
        if self.rank[x] < self.rank[y]:
            x, y = y, x  # 保证 rank(x) > rank(y)，x所在的集合深度更大
        self.pa[y] = x
        self.rank[x] += self.rank[y]

```

#### 例题1：洛谷P1111-修复公路

https://www.luogu.com.cn/problem/P1111

```python
class Village:
    def __init__(self, size, ans, cnt):
        self.pa = list(range(size))
        self.rank = [1] * size
        self.ans = ans
        self.cnt = cnt

    def find(self, x):
        if self.pa[x] != x:
            self.pa[x] = self.find(self.pa[x])
        return self.pa[x]

    def union(self, x, y):
        x, y = self.find(x), self.find(y)
        if x == y:
            return
        self.cnt += 1
        if self.rank[x] < self.rank[y]:
            x, y = y, x
        self.pa[y] = x
        self.rank[x] += self.rank[y]


n, m = map(int, input().split())
villages = Village(n, -1, 0)
build = []
for i in range(m):
    build.append(tuple(map(int, input().split())))  # village_1, village_2, time
build.sort(key=lambda x: x[2])
for i in build:
    villages.union(i[0] - 1, i[1] - 1)
    if villages.cnt == n - 1:
        villages.ans = i[2]
        break
print(villages.ans)
```

