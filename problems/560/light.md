問題：https://leetcode.com/problems/subarray-sum-equals-k/

# step 1
- 解いたことがある

```java
class Solution {
  public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> sumToCount = new HashMap<>();
    int prefixSum = 0;

    int result = 0;
    for (int i = 0; i < nums.length; ++i) {
      sumToCount.merge(prefixSum, 1, Integer::sum);
      prefixSum += nums[i];
      result += sumToCount.getOrDefault(prefixSum - k, 0);
    }
    return result;
  }
}
```

- 9分ぐらい
- 深く考えずに書き始めてしまい、盛大に書き直した
