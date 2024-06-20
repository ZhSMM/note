# Leetcode

#### [1. 两数之和](https://leetcode.cn/problems/two-sum/)

```rust
impl Solution {
    pub fn two_sum(nums: Vec<i32>, target: i32) -> Vec<i32> {
        use std::collections::HashMap;
        let mut map = HashMap::new();
        for (idx, &val) in nums.iter().enumerate() {
            let ex = target - val;
            if let Some(&index) = map.get(&ex) {
                return vec![index as i32, idx as i32];
            }
            map.insert(val, idx);
        }
        return vec![];
    }
}
```



#### [88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)

```rust
impl Solution {
    pub fn merge(nums1: &mut Vec<i32>, m: i32, nums2: &mut Vec<i32>, n: i32) {
        let mut index = m + n - 1;
        let mut mi = m - 1;
        let mut ni = n - 1;
        while index >= 0 {
            if mi < 0 || ni < 0 {
                break;
            }
            if nums1[mi as usize] >= nums2[ni as usize] {
                nums1[index as usize] = nums1[mi as usize];
                mi = mi - 1;
            } else {
                nums1[index as usize] = nums2[ni as usize];
                ni = ni - 1;
            }
            index = index - 1;
        }
        while ni >= 0 {
            nums1[ni as usize] = nums2[ni as usize];
            ni = ni - 1;
        }
    }
}
```

