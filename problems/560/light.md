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


# step 2
Javaでたくさん解いている方々：
- https://github.com/goto-untrapped/Arai60/pull/28/changes
- https://github.com/ryoooooory/LeetCode/pull/3/changes

コメント集：
- https://discord.com/channels/1084280443945353267/1303257587742933024/1322011962598625280
  - 「`prefix_sum_to_count[0] = 1` `prefix_sum_to_count[prefix_sum] += 1` は、形式的な操作として、ループの先頭に移動させてまとめようと思えばまとめられますね。」
  - たしかに、自分が書くときも、そういう書き直しをした
- https://discord.com/channels/1084280443945353267/1300342682769686600/1357378682163036160
  - `merge`で書いたところを`getOrDefault`と`put`で書く方法もある。それを1行で書くのは好ましくないようだ
  - 個人的には、`merge`で書く方法は丸暗記のようになってしまっていたかも


# step 3
1回目：3:36、2回目：4分ぐらい、3回目：2分半ぐらい、4回目：4分ぐらい

元とほぼ変更なし。
```java
class Solution {
  public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> sumToCount = new HashMap<>();
    int prefixSum = 0;
    sumToCount.put(prefixSum, 1);

    int result = 0;
    for (int i = 0; i < nums.length; ++i) {
      prefixSum += nums[i];
      result += sumToCount.getOrDefault(prefixSum - k, 0);
      sumToCount.merge(prefixSum, 1, Integer::sum);
    }
    return result;
  }
}
```

step 3で書くとき、若干、思考停止ぎみかもしれない。
