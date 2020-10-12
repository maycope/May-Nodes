###  第k个排序

#### 题目(60-困难)

给出集合 [1,2,3,…,n]，其所有元素共有 n! 种排列。

按大小顺序列出所有排列情况，并一一标记，当 n = 3 时, 所有排列如下：

"123"
"132"
"213"
"231"
"312"
"321"
给定 n 和 k，返回第 k 个排列。

说明：

* 给定 n 的范围是 [1, 9]。
* 给定 k 的范围是[1,  n!]。

示例 1:

输入: n = 3, k = 3
输出: "213"
示例 2:

输入: n = 4, k = 9
输出: "2314"

####  思路

利用全排列时候的数值之间的关系 先列举出来 个数字的阶乘 然后进行计算操作。



####  代码

```java
    public String getPermutation(int n, int k) {
final int[] arr = new int[]{1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880};
    boolean[] visited = new boolean[n + 1];
    Arrays.fill(visited, false);
    StringBuilder permutation = new StringBuilder();
    for (int i = n - 1; i >= 0; i--) {
        int cnt = arr[i];
        for (int j = 1; j <= n; j++) {
            if (visited[j]) {
                continue;
            }
            if (k > cnt) {
                k -= cnt;
                continue;
            }
            visited[j] = true;
            permutation.append(j);
            break;
        }
    }
    return permutation.toString();
    }
```

