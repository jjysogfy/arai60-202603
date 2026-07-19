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
  - goodは良くなかった気がする。waysとgoodWaysって何が違うのかって感じがする
- 実務ならn ≥ 1などチェックしたほうが良い、ような場合が多そう

- nは300までなので、間に合う
- Answers are guaranteed to be fit into a 32 bit integer. と書いてある
  - そういう入力に限られるということ
  - （用途によっては）なにかしたほうがよさそう
    - オーバーフローした時点で例外を投げる・特殊な値を返す・プログラムを停止する。大きい整数型を使う。など


# step 2
人のコードを見てみる
- https://github.com/rimokem/arai60/pull/30/changes
- https://github.com/goto-untrapped/Arai60/pull/44/changes
- https://github.com/ryoooooory/LeetCode/pull/33/files


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
  - LRU cache自体を書いている人もいる


配列を使わない方法でstep 1を書き直してみる。
```java
// step 2 その1
class Solution {
  int countWays(int n, int k) {
    // n個のpostをpaintする方法の数
    // 初期値はn=1の場合
    int numWays = k;

    // 最後の2つが同じ色にならないように、n個のpostをpaintする方法の数
    int numDistinct = k;

    for (int i = 1; i < n; ++i) {
      int newNumWays = (k - 1) * numWays + numDistinct;
      numDistinct = (k - 1) * numWays;
      numWays = newNumWays;
    }

    return numWays;
  }
}
```

- 間違えつつ書いた
  - 新しい変数`newNumWays`を用意せずに間違えた
- 変数名numDistinctは **悩む**
  - コメントをしっかり書いて、変数名は短めに、という考えだった
  - しかし、文法的におかしい気もする


## step 2.2 清書
i-2番目とi-1番目からi番目を求める書き方の人もいる。やってみる
```java
// step 2.2 清書
class Solution {
  int countWays(int n, int k) {
    if (n == 1) {
      return k;
    }

    int numWays = k * k;
    int previousNumWays = k;

    for (int i = 3; i <= n; ++i) {
      int newNumWays = (k - 1) * (numWays + previousNumWays);
      previousNumWays = numWays;
      numWays = newNumWays;
    }

    return numWays;
  }
}
```

- これも間違えつつ書いた
  - nが小さい場合の注意が必要
  - n=0のnumWaysは1とするのが自然だが、漸化式には合わない
    - n=0のときは「最後に使った色」が存在しないから
- 考えかた：
  - 長さが1と2のブロックがあり、同じ色のブロックが隣接しないように長さnまで並べる
- 変数名が付けやすい気がする


# step 3
1回目：7:16、2回目：3:04、3回目：2:16

- 清書を作るまでに結構覚えてしまったのでなんとか書けたけど、あんまりすんなり書いている感覚はないかも

```java
// step 3
//   清書のコードとほぼ同じ（nextNumWaysの変数名の変更と、その定義式の微妙な変更ぐらい）
class Solution {
  int countWays(int n, int k) {
    if (n == 1) {
      return k;
    }

    int numWays = k * k;
    int previousNumWays = k;

    for (int i = 3; i <= n; ++i) {
      int nextNumWays = (k - 1) * numWays + (k - 1) * previousNumWays;
      previousNumWays = numWays;
      numWays = nextNumWays;
    }

    return numWays;
  }
}
```
