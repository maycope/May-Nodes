### 组合总和III

#### 题目

找出所有相加之和为 **n** 的 **k** 个数的组合。组合中只允许含有 `1 - 9` 的正整数，并且每种组合中不存在重复的数字。

说明：

* 所有数字都是正整数。

* 解集不能包含重复的组合。 

  

**示例 1:**

```
输入: k = 3, n = 7
输出: [[1,2,4]]
```

**示例 2:**

```
输入: k = 3, n = 9
输出: [[1,2,6], [1,3,5], [2,3,4]]
```

#### 思路

基本的回溯思想即可。判断在什么位置停止是此类问题的第一个需要考虑的地方，对于此题，要求有特定的数目，就需要对list的长度进行判断。对于总和采用减法的方法，来找寻出来等于0的时刻即可。

#### 代码

```java
  public List<List<Integer>> combinationSum3(int k, int n) {
   List<List<Integer>> lists = new ArrayList<>();
        List<Integer> list = new ArrayList<>();
        dfs(k,n,list,lists,1);
        return  lists;

    }
    private  static  void dfs(int k ,int n,List<Integer> list,List<List<Integer>> lists ,int start){
        /**
         * 首先是判断什么时候返回
         */
        if(list.size() == k && n == 0){
            lists.add(new ArrayList<>(list));
            return;
        }
        for(int i = start ;i<=9;i++){
            list.add(i);
            dfs(k,n-i,list,lists,i+1);
            list.remove(list.size()-1);

        }
    }
```

