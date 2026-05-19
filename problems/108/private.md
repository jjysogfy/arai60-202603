問題（108. Convert Sorted Array to Binary Search Tree）：https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/

```java
class Solution {
  public TreeNode sortedArrayToBST(int[] nums) {
    return sortedArrayToBstHelper(nums, 0, nums.length);
  }

  TreeNode sortedArrayToBstHelper(int[] nums, int start, int end) {
    if (end - start == 0) {
      return null;
    }
    if (end - start == 1) {
      return new TreeNode(nums[start]);
    }
    int middle = (start + end) / 2;
    TreeNode leftNode = sortedArrayToBstHelper(nums, start, middle);
    TreeNode rightNode = sortedArrayToBstHelper(nums, middle + 1, end);
    return new TreeNode(nums[middle], leftNode, rightNode);
  }
}
```
