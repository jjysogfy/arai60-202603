問題：300. Longest Increasing Subsequence https://leetcode.com/problems/longest-increasing-subsequence/

# step 1
```java
class Solution {
  public int lengthOfLIS(int[] nums) {
    // lengths[i] = length of the LIS ending with nums[i]
    int[] lengths = new int[nums.length];
    for (int i = 0; i < nums.length; ++i) {
      lengths[i] = 1;
      for (int k = 0; k < i; ++k) {
        if (nums[k] >= nums[i]) {
          continue;
        }
        lengths[i] = Math.max(lengths[i], lengths[k] + 1);
      }
    }
    return Arrays.stream(lengths).max().orElse(0);
  }
}
```

- 問題文を見てから一週間ぐらいして書いた
  - 前にも見たことがある
- かかった時間：6:01
- 方針：
  - DPと書いてあるし、見たこともあるので、念頭に置いて考える
  - `nums[nums.length - 1]`で場合分けしてみる
  - LISに含まれるか、含まれないか
  - いずれにせよ、`[0, nums.length - 2]`を見れば計算できる

