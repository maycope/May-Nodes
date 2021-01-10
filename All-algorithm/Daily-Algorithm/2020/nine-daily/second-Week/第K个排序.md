## 第K个排序

### 题目

题目来源于每日一题。

[第K个排序](https://leetcode-cn.com/problems/permutation-sequence/)

### 普通思想

此题是对于全排列的变形处理[全排列](https://leetcode-cn.com/problems/permutations/)，可以找先进行一个全排列处理，然后利用k值来循环查找到我们要找的具体的值，但是毫无例外地超时了。

### 进一步思考

我们没有必要完完全全完成所以的搜索，例如对于 n=4，k=9的时候来说，可以提前找出来对应的个数值。

如何提前找到呢？对于全排列来说就是阶乘的问题，例如对于4的全排列来说，就是会有24个不同的结果，且是按序排列的，我们对于每个1，2，3，4都是六个不同的值，至于为什么是6个呢，就是1开头之后，2，3，4 全排列，然后加入1，所以对于每个值都会有剩余的全排列，现在给我们一个9 ，我们对1开头进行全排列之后（这里就可以不同计算出来具体的值）弄一个阶乘的数组就好了，对于1来说全排列有6个，我们在完成之后就会发现，在以1开头的队列里面不能够找到正确的值，就可以以2进行开头，同样2开头的时候，剩余的数子就是3了，因为是`9-6`，于是就发现了肯定是在2开头的里面。这个时候，就可以把2计算在内。这个时候，剩下的就是三个数字了能够进行的全排列就是6个，但是我们不能直接找到6，这样就是肯定超出来，所以就是2的全排列，也就是2 ，带进去之后是**3-2** 就是说，以21开头的34 和43 并不能满足情况。于是就找到了23，但是这里也是困惑的地方，就是一个循环体，既然1，不能够满足情况，就让3来，因为每次的进入都会让k的值减少。这样就可以得到我们想要的结果。

所以开始的地方在于我哦可以知道对于1来说后面有六个不同的结果，但是怎么知道 21来说有两个不同的记过 还是靠 阶乘数组。然后进一步往前推进阶乘数组，缩小范围，妙啊 不管是递归还是循环 都蛮好的。

1. 首先是计算出来阶乘，也可以手动进行基础的赋值操作。
2. 然后是 判断出来跳出来的条件，就是传入的值已经满足所需要的数量的时候。
3. 循环体里面
   1. 是判断是否访问过，
   2. 是判断是否是太小了数量 不在范围里面。需要减去之前不能够满住情况的条件。
   3. 若是以上的两条都不能够满住，就是说明，当前访问的数字的当前的顺序会在结果中出现。
   4. 将出现的数字加入到已经有的结果里面，在完成之后，需要退出当前的循环体，表示当前的循环已经正确找到了当前顺序下的值。开启下一轮的向前推进的循环。

### 利用递归来实现。

```java
import java.util.Arrays;

public class Solution {

    /**
     * 记录数字是否使用过
     */
    private boolean[] used;

    /**
     * 阶乘数组
     */
    private int[] factorial;

    private int n;
    private int k;

    public String getPermutation(int n, int k) {
        this.n = n;
        this.k = k;
        calculateFactorial(n);

        // 查找全排列需要的布尔数组
        used = new boolean[n + 1];
        Arrays.fill(used, false);

        StringBuilder path = new StringBuilder();
        dfs(0, path);
        return path.toString();
    }


    /**
     * @param index 在这一步之前已经选择了几个数字，其值恰好等于这一步需要确定的下标位置
     * @param path
     */
    private void dfs(int index, StringBuilder path) {
        if (index == n) {
            return;
        }

        // 计算还未确定的数字的全排列的个数，第 1 次进入的时候是 n - 1
        int cnt = factorial[n - 1 - index];
        for (int i = 1; i <= n; i++) {
            if (used[i]) {
                continue;
            }
            if (cnt < k) {
                k -= cnt;
                continue;
            }
            path.append(i);
            used[i] = true;
            dfs(index + 1, path);
            // 注意 1：没有回溯（状态重置）的必要

            // 注意 2：这里要加 return，后面的数没有必要遍历去尝试了
            return;
        }
    }

    /**
     * 计算阶乘数组
     *
     * @param n
     */
    private void calculateFactorial(int n) {
        factorial = new int[n + 1];
        factorial[0] = 1;
        for (int i = 1; i <= n; i++) {
            factorial[i] = factorial[i - 1] * i;
        }
    }
}
```

### 利用循环来实现

```java
class Solution {
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
}
```



