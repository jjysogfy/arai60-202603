問題：53. Maximum Subarray
https://leetcode.com/problems/maximum-subarray/

# step 1
```java
class Solution {
  public int maxSubArray(int[] nums) {
    int[] prefixSums = new int[nums.length + 1];
    for (int i = 0; i < nums.length; ++i) {
      prefixSums[i + 1] = prefixSums[i] + nums[i];
    }

    int lowest = prefixSums[0];
    int largestSum = Integer.MIN_VALUE;
    for (int right = 0; right < nums.length; ++right) {
      int sum = prefixSums[right + 1] - lowest;
      largestSum = Math.max(largestSum, sum);
      lowest = Math.min(lowest, prefixSums[right + 1]);
    }

    return largestSum;
  }
}
```

思ったこと：
- だいたい5分ぐらい考えた
- そのあと7:30で書いた
- アイデア
  - 前にやった問題[560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)を連想
    - 累積和を標高とみるたとえで考えていた
    - 標高の差がもっとも大きい2地点を探したい
    - 前から見ていって、今まででもっとも低い標高をメモしながら進めればいい
  - Arai60ではDPと分類されていて、たしかにそれでもできそう
    - 前から見ていって、「これまでのlargest sum」と「末尾を含むlargest sum」とを管理すればよさそう
  - LeetCodeには、分割統治でもできると書いてある
    - numsをいくつかの区間に分ける。
    - 「区間内のlargest sum」と「区間の左端（右端）を含むlargest sum」の3つを管理すればよさそう？
    - 「which is more subtle.」と書いてある。たしかにちょっと実装はこんがらがりそう
- 変数名lowestはsmallestのほうがlargestSumと合う
- 変数名sumは避けたほうが無難だったか


# step 2
