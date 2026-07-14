問題：276. Paint Fence
https://www.geeksforgeeks.org/problems/painting-the-fence3727/1
( https://leetcode.com/problems/paint-fence/ )


# step 1
```java
class Solution {
  int countWays(int n, int k) {
    // ways[i] = i+1個のpostをpaintする方法の数
    int[] ways = new int[n];
    ways[0] = k;

    // goodWays[i] = 最後の2つが同じ色にならないように、i+1個のpostをpaintする方法の数
    int[] goodWays = new int[n];
    goodWays[0] = k;

    for (int i = 1; i < n; ++i) {
      ways[i] = (k - 1) * ways[i - 1] + goodWays[i - 1];
      goodWays[i] = (k - 1) * ways[i - 1];
    }

    return ways[n - 1];
  }
}
```

- しばらく前から考えていた
- 時間：10:09
- 最後の色で場合分けすると、漸化式が立つ
- 変数名にかなり迷った
  - 適当に短めの名前を付けて、コメントで補足することにした
  - `ways`というより`numsWays`だったかも
- 実務ならn ≥ 1などチェックしたほうが良い、ような場合が多そう

- nは300までなので、間に合う
- Answers are guaranteed to be fit into a 32 bit integer. と書いてある
  - そういう入力に限られるということ
  - （用途によっては）なにかしたほうがよさそう
    - オーバーフローした時点で例外を投げる・特殊な値を返す・プログラムを停止する。大きい整数型を使う。など


# step 2
人のコードを見てみる
- https://github.com/rimokem/arai60/pull/30/changes


気づいたこと：
- `O(log n)`の方法がある
  - （言われてみれば、見たことがある）
  - （定数係数）線形漸化式なので、行列のべき乗だから、`M ^ (2n) = (M*M) ^ n`のようにかけ算回数を減らせる
- 「一般項」をdoubleで計算する方法もありうる
  - 誤差には注意
  - 負荷が高く、それほど勧められる書き方ではなさそう

- k=2ならフィボナッチ数

- メモ化なしの再帰でも間に合ってしまうらしい
  - https://github.com/goto-untrapped/Arai60/pull/44/changes#r1703434310
  - 結果がint型に収まることを使った計算時間見積もり
- LRU cacheでメモ化する人もいる


