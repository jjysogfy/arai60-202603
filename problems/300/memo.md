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
  - 高速な解法を思い出せた気でいたが、間違えていた
    - 思い出した気でいたときのメモは`side_notes.md`に残しておく
- （ところで、どんな`nums`のせいでこういうアルゴリズムが必要になるのか、わかっていないと気づく）


# step 2
Javaでたくさん解いている方。
- https://github.com/goto-untrapped/Arai60/pull/18
- https://github.com/ryoooooory/LeetCode/pull/34

気づいたこと
## step 2.1 `O(N log N)`の方法
`O(N log N)`の方法について、読んだことをもとに、自分でも考えてみる
- https://github.com/ryoooooory/LeetCode/pull/34#discussion_r2133973501
- `lengths[i + 1]`を高速に計算するにはどうするか
  - `lengths`はstep 1の記号：`lengths[i]`は、`nums[i]`で終わるLISの長さ
- `nums[i + 1]`の具体的な値は重要ではなく、`nums[0], .., nums[i]`との関係だけが大事
- とくに、`nums[i + 1]`より真に小さいnumsのうち一番大きいものが大事（`nums[k]`とおく）
- 次のようなTreeMap `numToLength`を考える
  - `numToLength.lowerEntry(num).getValue()`は、num以下の数からなるLISの長さ
- 例：
  - numsが100, 400, 200, 300とする
  - numToLengthは次のように遷移する
    - `{100: 1}` ->
    - `{100: 1, 400: 2}` ->
    - `{100: 1, 200: 2}` ->
    - `{100: 1, 200: 2, 300: 3}`
  - 安直には次のようにやりそうになってしまうが、不都合
    - `{100: 1}` ->
    - `{100: 1, 400: 2}` ->
    - `{100: 1, 200: 2, 400: 2}` ->
    - `{100: 1, 200: 2, 300: 3, 400: 2}`
    - 更新時に、適宜removeする必要がある

```java
// step 2.1 その1
class Solution {
  public int lengthOfLIS(int[] nums) {
    // numToLength.lowerEntry(num).getValue() == num以下に収まるLISの長さ
    NavigableMap<Integer, Integer> numToLength = new TreeMap<>();

    for (int num : nums) {
      Map.Entry<Integer, Integer> lower = numToLength.lowerEntry(num);
      int newLength = lower == null ? 1 : lower.getValue() + 1;

      NavigableMap<Integer, Integer> tail = numToLength.tailMap(num, true);
      Iterator<Integer> iter = tail.values().iterator();
      while (iter.hasNext()) {
        int length = iter.next();
        if (length > newLength) {
          break;
        }
        iter.remove();
      }

      numToLength.put(num, newLength);
    }

    return numToLength
        .sequencedValues()
        .getLast();
  }
}
```

- 実際には、`iter`でループを回す必要はない
  - ループは高々1回しか回らない
- 

