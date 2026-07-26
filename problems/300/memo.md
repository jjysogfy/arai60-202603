問題：300. Longest Increasing Subsequence
https://leetcode.com/problems/longest-increasing-subsequence/

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
- ミス：
  - `IntStream.max`がOptionalを返すのを忘れていた
- 方針：
  - （DPと書いてあるし、この問題を見たこともあるので、念頭に置いて考える）
  - `nums[nums.length - 1]`で場合分けしてみる：
    - LISに含まれるか、含まれないか
    - どちらの場合でも、`[0, nums.length - 2]`を見てLISの長さを計算できる
  - 時間計算量`O(N^2)`、`N <= 2500`なので、実行時間は60ms--600ms程度と見積もる
    - 41msだった。合っている
  - ソートと二分探索とかで高速化できたはず、となんとなく覚えている
    - `nums[0], .., nums[i]`がkeyで、`lengths`がvalueの`TreeMap`を作る
    - `TreeMap`への`nums[i + 1]`の挿入位置を見ると`lengths[i + 1]`が計算できる
    - 追記：これ、よく考えると微妙だ
      - keyが大きくなるごとにlengthは小さくなるべき
      - `nums[i + 1]`を挿入するときに注意がいる
      - それを踏まえると次の方法が自然に出てくる気がする
  - しばらく考えて思い出したけど、次の方法が有名だったはず
    - `lengthToLast[l]`を長さlのLISの末尾に現れる数で最小のものとする
    - numsを前から見てlengthToLastを更新する
    - 自分じゃ時間内にゼロから思いつくのは無理かな、と思ってしまう
- （ところで、どんな`nums`のせいでこういうアルゴリズムが必要になるのか、わかっていないと気づく）


# step 2
Javaでたくさん解いている方。
- https://github.com/goto-untrapped/Arai60/pull/18
- https://github.com/ryoooooory/LeetCode/pull/34

