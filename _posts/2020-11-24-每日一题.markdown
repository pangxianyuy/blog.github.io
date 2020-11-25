---
layout:     post
title:      "11-24日打卡"
subtitle:   " 努力生存"
date:       2020-11-24 
author:     "胖咸鱼y"
header-img: "img/in-post/LeetCode/11_24.jpg"
catalog: true
tags:
    - LeetCode
    - 每日更新系列
typora-root-url: ..
---

## 每日打卡计划

1、给出一个**完全二叉树**，求出该树的节点个数。 

完全二叉树的定义如下：在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置。若最底层为第 h 层，则该层包含 1~ 2h 个节点。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/count-complete-tree-nodes
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。



```
输入: 
    1
   / \
  2   3
 / \  /
4  5 6

输出: 6
```

**解题思路：**

if(最左节点深度=最右节点深度)：

​	则此树为完全二叉树，节点个数为2^h-1

否则：

​	递归求解

代码：

```python3
class Solution:
    def countNodes(self, root: TreeNode) -> int:
        if not root: return 0
        left_height = 0
        left_node = root
        right_height = 0
        right_node = root
        while left_node:
            left_node = left_node.left
            left_height += 1
        while right_node:
            right_node = right_node.right
            right_height += 1
        if left_height == right_height:
            return 2**left_height - 1
        return 1 + self.countNodes(root.left) + self.countNodes(root.right)
```



2、18、四数之和

给定一个包含 n 个整数的数组 nums 和一个目标值 target，判断 nums 中是否存在四个元素 a，b，c 和 d ，使得 a + b + c + d 的值与 target 相等？找出所有满足条件且不重复的四元组。

注意：

答案中不可以包含重复的四元组。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/4sum
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。



>给定数组 nums = [1, 0, -1, 0, -2, 2]，和 target = 0。
>
>满足要求的四元组集合为：
>[
>  [-1,  0, 0, 1],
>  [-2, -1, 1, 2],
>  [-2,  0, 0, 2]
>]
>
>来源：力扣（LeetCode）
>链接：https://leetcode-cn.com/problems/4sum
>著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。



**解题思路**

```python3
class Solution:
    def fourSum(self, nums: List[int], target: int) -> List[List[int]]:
        sort_nums = sorted(nums)
        ls = len(nums)
        res = {}
        res_final=[]
        pairs = {}
        for i in range(ls - 3):
            for j in range(i + 1, ls - 2):
                res_2 = sort_nums[i] + sort_nums[j]
                try:
                    pairs[target - res_2].append([i, j])
                except KeyError:
                    pairs[target - res_2] = [[i, j]]


        for key, temp in pairs.items():
            for pair in temp:
                j = pair[1] + 1
                k = ls - 1
                while j < k:
                    current = sort_nums[j] + sort_nums[k]
                    if current == key:
                        result = (sort_nums[pair[0]], sort_nums[pair[1]], sort_nums[j], sort_nums[k])
                        res[result] = True
                        j += 1
                    elif current < key:
                        j += 1
                    else:
                        k -= 1
        return [_ for _ in res.keys()]
```

