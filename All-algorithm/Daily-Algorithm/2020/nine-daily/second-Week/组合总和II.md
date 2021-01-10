### 组合总和II

#### 题目

给定一个数组 `candidates` 和一个目标数 `target` ，找出 `candidates` 中所有可以使数字和为 `target` 的组合。

`candidates` 中的每个数字在每个组合中只能使用一次。

**说明：**

* 所有数字（包括目标数）都是正整数。
* 解集不能包含重复的组合。 

**示例 1:**

```
输入: candidates = [10,1,2,7,6,1,5], target = 8,
所求解集为:
[
  [1, 7],
  [1, 2, 5],
  [2, 6],
  [1, 1, 6]
]
```

#### 思想

利用回溯的基本思想，基本的套路，还是和**组合总和**是相同，但是此题对于每一个值都是不能够重复使用的，所以需要在每一次的循环中都进行下一个值的选择。

#### 代码

```java
public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        List<List<Integer>> lists = new ArrayList<>();
        List<Integer> list = new ArrayList<>();
        Arrays.sort(candidates);
        dfs(0,candidates,lists,list,target);
        return  lists;
    }
    private  static void dfs(int index,int [] nums,List<List<Integer>> lists,List<Integer> list,int target){
        if(target<0)
            return;
        if(target == 0){
            lists.add(new ArrayList<>(list));
            return;
        }
        for(int i = index;i<nums.length;i++){
            if(i >index && nums[i]==nums[i-1])
                continue;
            list.add(nums[i]);
            dfs(i+1,nums,lists,list,target-nums[i]);
            list.remove(list.size()-1);
        }
    }
```

