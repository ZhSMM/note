#### [26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)

```c++
class Solution {
    public:
    int removeDuplicates(vector<int>& nums) {
        int i = 0, j = 1;
        while (j < nums.size()) {
            if (nums[i] != nums[j]) {
                nums[++i] = nums[j];
            }
            ++j;
        }
        return i + 1;
    }
};
```



#### [88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/description/)

```c++
class Solution {
    public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int i = m - 1, j = n - 1;
        for (int k = m + n - 1; k >=0; k--) {
            if (j < 0 || (i >= 0 && nums1[i] > nums2[j])) {
                nums1[k] = nums1[i--];
            } else {
                nums1[k] = nums2[j--];
            }
        }
    }
};
```

