問題：https://leetcode.com/problems/two-sum/description/


# 取り組みかた・このファイルの構成
基本的に[「標準的な進め方」](https://docs.google.com/document/d/1bjbOSs-Ac0G_cjVzJ2Qd8URoU_0BNirZ8utS3CUAeLE/edit?tab=t.0#heading=h.6ana9osx0wrd)にならう。

- step 1 で、まず解く。step 2.1 で、人のコードやコメント集を読む。それをもとに、step 2.2 では清書する。step 3 では、3回続けて解けるように練習する。

Arai60から、カテゴリ「Heap, PriorityQueue」を一旦飛ばして、カテゴリ「HashMap」をやってみることにした。


# step 1 : 解く
解くまえに思ったこと：
- たぶん解いたことがある
- HashMap を使う、時間計算量 `O(n log n)` のアルゴリズムがある
  - `i` についてループ。`nums[j] == target - nums[i]` となる `j > i` を、HashMapで見つける
  - `j` についてループしたほうがみやすいか
- 安直に `O(n^2)` のアルゴリズムもある
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.xbcr3241jgv8)を見つつ、とりあえず1秒に1Gステップ程度で見積もる
  - 問題の設定だと、`10^8`ステップ程度だから、0.1秒程度となり間に合う

```java
// O(n^2)
class Solution {
  public int[] twoSum(int[] nums, int target) {
    for (int i = 0; i < nums.length; ++i) {
      for (int j = i + 1; j < nums.length; ++j) {
        if (nums[i] + nums[j] == target) {
          return new int[] {i, j};
        }
      }
    }
    return null; // 問題の設定では、実行されない
  }
}
```

書いて思ったこと：
- 配列の初期化の方法を忘れて、調べつつ書いた。5〜10分
- 実行時間は44msなので、見積もりの範囲内か

```java
// O(n log n)
class Solution {
  public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> numToIndex = new HashMap<>();
    for (int j = 0; j < nums.length; ++j) {
      var i = numToIndex.get(target - nums[j]);
      if (i != null) {
        return new int[] {i, j};
      }

      numToIndex.put(nums[j], j);
    }
    return null;
  }
}
```

書いて思ったこと：
- 7分ぐらい
- `numToIndex.get` と `!= null` で書いた
  - はじめ `numToIndex.containsKey` と `numToIndex.get` で書いていたが、keyを2回書くのが面倒でこの書き方にした
- なお、最後のreturn文（`return null`）は消すとコンパイルエラー
  - `return new int[0]` と迷ったが、`null` のほうが適切だと思った。空列の総和は0なので
- ループの中身を書くのにちょっと頭をひねった
  - 考えるとき、「引き継ぎ」である `numToIndex.put(nums[j], j);` と、その前を区切りたくなって、空行が入った
- 変数の宣言（ `Map<Integer, Integer> numToIndex = new HashMap<>();` ）の書き方をいつも少し迷う
  - 変数の型を `Map` にするか、`HashMap` にするか。後者のとき `var` と書くか。

ここまで30分ぐらい。


# step 2.1 : 読む
コメント集を見る。
- Exception https://github.com/takumihara/leetcode/pull/1#discussion_r1806764103
- Javaの例外についてよくわからないので、手元にある入門書を見ておく。読みやすいがそれほど信用できない気もしているので、真に受けすぎないようにしておく
  - できたら公式ドキュメントを読んでおきたいが、大変なのでほどほどにしておく
  - Javaの例外を表すクラスには、大きく分けてError、Exception、RuntimeExceptionの3つがある。今回関係するのは後者2つのどちらかだと思う
  - Exceptionは必ず捕捉される必要がある。RuntimeExceptionはそうではない
    - 後者は前者のsubclassである。特定のsubclassだけ特別扱いされるのはなんとなく変な感じもするが、どうやらそういうものらしい
    - ドキュメントはこのあたりか https://docs.oracle.com/en/java/javase/26/docs/specs/jls/jls-11.html#jls-11.1.1
  - そのため、LeetCodeに限ると、RuntimeExceptionはthrowできるが、Exceptionはできない
    - 捕捉してください、というコンパイルエラーになる
    - `public int[] twoSum(int[] nums, int target) throws Exception` と宣言しても、呼び出し側で同じコンパイルエラーが出る
- どのException（のsubclass）をthrowすべきか？
  - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/lang/Exception.html
  - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/lang/RuntimeException.html
  - たくさんあってよくわからなかった
- そもそも、この問題で、例外を投げたくなるのかよくわかっていない
  - もちろん、状況によるんだろうけど
  - 欲しいペアが存在することだけはっきりしていて、しかしそれがどれかわからない、というのがどれだけよくある設定なのか？ 存在するかどうかわからなかったら、なんとなく、例外まで投げないほうが便利そうではないか？ とか思った

- 発想の話 https://discord.com/channels/1084280443945353267/1229085360403775569/1229955174446137344
  - 「この問題、手でやるならどうしますか?」と言われると、とりあえずソートしようかな、と思った
  - あれ、それで何が起きるか？ ……前と後ろから同時に見ていけばできるか
  - とか思ったが、「本当に手でやる」ことをあんまり想像していなかったかも。本当にやるなら、まず、1000枚というのはちょっとげんなりする量だ
  - そうなると、ソートして、それからまた前後から1枚ずつ見ていく、というのは二度手間のような気がしてくる
  - 「ノートに何があったかを書き留めたりしませんか。」とのこと。で、1000枚もあるので、そのメモもソートされていないと大変である。すると自分のはじめのコードのようになる。本当に手で1000枚やると考えると、どのみち、面倒ではある

ちょっと話がそれるが、思ったこと：
- 返り値の型が `int[]` なのは、ちょっとだけ気持ちが悪い
  - いつも長さ2なので
  - Javaで複数の値を返すのは面倒なので、これも妥協の範囲なのかな
- 追記：ところで、配列ではboxingが不要なこと（ArrayListなどのコレクションとは違う）とか、理由を理解していない、と気付いた
  - 調べなきゃいけないが、また時間がかかりそうなので、一旦おいておく

ここまで2時間ぐらい。

Javaで完走していそうな方のコードとレビューを読む。
- https://github.com/ryoooooory/LeetCode/pulls
- 例外について https://github.com/ryoooooory/LeetCode/pull/18#discussion_r1735412557
  - 「ここはかなり状況によりますね。ユーザーフェイシングかが一番大きいかも知れません。」とのこと
- note.md, L9: 「Integer.MAX_VALUEは2,147,483,647」
  - `nums[i]`の制約を意識できていなかった
  - Integer.MAX_VALUEは `2**31 ~ 2 * (1000**3) ~ 2 * 10**9` と見積もれる
- 例外を投げるコード
  - `IllegalArgumentException("No two sum pair found.")` をthrowしている

- https://github.com/goto-untrapped/Arai60/pull/2
  - ryoooooory氏もそうだったが、`complement = target - nums[i]` とおいて、`numToIndex.containsKey(complement)` と `numToIndex.get(complement)` を使っている

完走していそうな方のなかから、もう少し見てみる。
- https://github.com/olsen-blue/Arai60/pull/11
- L40-43: Pythonの話だが、「Exceptionには「"より"詳細("more" precise)」なものと「”より”曖昧」なものがあるのか？」とのこと。
  - JavaでもExceptionクラスは「より曖昧」といえるし、似た事情になっていそう
- L10: ループが i = 1 からになっている。確かに i = 0 は無駄だ。

- https://github.com/fhiyo/leetcode/pull/14


# step 2.2 : 清書
```java
class Solution {
  public int[] twoSum(int[] nums, int target) {
    var numToIndex = new HashMap<Integer, Integer>();
    for (int i = 0; i < nums.length; ++i) {
      var j = numToIndex.get(target - nums[i]);
      if (j != null) {
        return new int[] {j, i};
      }
      numToIndex.put(nums[i], i);
    }
    return null;
  }
}
```

- 変数名を変えたぐらい
  - `j` はもっといい名前がある気もするが、思いつかないので妥協

# step 3 : 3回書けるように
- 1回目: 3:34、2: 2:23、3: 1:47、4: 1:58

- `int j = numToIndex.get(target - nums[i]);` だとコンパイルエラーになる。気づかなかった
  - `j != null` は型エラーになる
  - `Integer j` としなくてはならない
  - こういう点は `.containsKey` を使う方法のがわかりやすい

- 他に、`.containsKey` を使う方法との違いがあるかどうか気になってきた
  - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/Map.html#get(java.lang.Object)
  - `j != null` と書くと、keyに対応するvalue自体がnullである場合と区別がつかない、と書いてある
  - 確かにそれは意図するところではなかったので、`.containsKey` を使うことにする

```java
class Solution {
  public int[] twoSum(int[] nums, int target) {
    var numToIndex = new HashMap<Integer, Integer>();
    for (int i = 0; i < nums.length; ++i) {
      int complement = target - nums[i];
      if (numToIndex.containsKey(complement)) {
        return new int[] {numToIndex.get(complement), i};
      }
      numToIndex.put(nums[i], i);
    }
    return null;
  }
}
```

- 3時間半ぐらい？
- 次回は https://leetcode.com/problems/group-anagrams/description/ に取り組む予定






## 追記：別の方法
上述の[発想の話](https://discord.com/channels/1084280443945353267/1229085360403775569/1229955174446137344)で出てきたやり方でも書いてみる。

```java
// ソートして、左右からポインタを動かす方法
class Solution {
  public int[] twoSum(int[] nums, int target) {
    Comparator<NumWithIndex> comparator = Comparator.comparing(numWithIndex -> numWithIndex.num);
    var numsWithIndexes = new NumWithIndex[nums.length];
    for (int i = 0; i < nums.length; ++i) {
      numsWithIndexes[i] = new NumWithIndex(nums[i], i);
    }
    Arrays.sort(numsWithIndexes, comparator);

    int left = 0;
    int right = nums.length - 1;
    while (left < right) {
      int sum = numsWithIndexes[left].num + numsWithIndexes[right].num;
      if (sum == target) {
        return new int[] {numsWithIndexes[left].index, numsWithIndexes[right].index};
      } else if (sum < target) {
        ++left;
      } else {
        --right;
      }
    }
    return null;
  }

  record NumWithIndex(int num, int index) {}
}
```

- Comparatorの使い方がわからずに、20分ぐらい調べた。というか、今も何も理解していない
  - 理解できなかった一つの理由は、ジェネリクスを知らないこと。そのせいでドキュメントの型を読めていないことなど
  - `Comparator.comparing` であって `Comparator<>.comparing` ではない
  - `.equals` とコンパチブルではない（`.index`を見ていない）のはよくないかもしれない
- ロジックの部分は8分ぐらいで書いた
  - leftとrightと書いたが、step 3 でのコードのように、i（つまりright）が主役で、iを止めるごとにj（つまりleft）が動く、としたほうがイメージしやすいかも
- 変数名 `numsWithIndexes` はちょっとうるさい
- こうして、まだ調べたり考えたりすべきことはあるが、一旦やめる
