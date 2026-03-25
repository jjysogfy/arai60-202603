問題：[49. Group Anagrams](https://leetcode.com/problems/group-anagrams/)


# step 1 : 解く
解くまえ・解くあいだに思ったこと：
- 解いたことがある
  - anagramかどうかの判定：どの文字が何回現れるか、という「カウント」を `Map` なり array なりで保持し、それが等しいか判定すればいい
- Javaの書き方がわからない
  - この状況でのソート（配列の `Comparator`？）
  - 配列などをkeyとする `Map`？
- さっさと人の答えを読もう

Javaでたくさん解いている人のコードを参照する。
- https://github.com/goto-untrapped/Arai60/pull/6/changes
- https://github.com/ryoooooory/LeetCode/pull/1/changes
- goto-untrapped氏の `GroupAnagramsStep4.java` を見てみた
  - 賢い、「カウント」ではなく、ソートした文字列を扱う。たぶん前に見たことがあるが、すっかり忘れていた

```java
// goto-untrapped氏と同じコードになった
class Solution {
  public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> sortedToAnagrams = new HashMap<>();
    for (String str : strs) {
      char[] sortedChars = str.toCharArray();
      Arrays.sort(sortedChars);
      String sorted = String.valueOf(sortedChars);

      if (!sortedToAnagrams.containsKey(sorted)) {
        sortedToAnagrams.put(sorted, new ArrayList<>());
      }
      sortedToAnagrams.get(sorted).add(str);
    }

    return new ArrayList<>(sortedToAnagrams.values());
  }
}
```

- goto-untrapped氏と同じコードになった
  - いきなり人のstep 4を参考にするとレビューするところが少なくなってしまうから、LeetCodeの答えとかをまずは参考にすべきかも


# step 2.1 : 読む
上述のgoto-untrapped氏、ryoooooory氏のほか、たくさん解いている人のPRを読んでみる

- https://github.com/olsen-blue/Arai60/pull/12/changes

- 「何が辞書キーとして使える/使えないかも調べてみると」 https://github.com/olsen-blue/Arai60/pull/12/changes#r1915836160
  - たしかに。
  - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/Map.html#:~:text=Note:%20great%20care%20must%20be%20exercised%20if%20mutable%20objects%20are%20used%20as%20map%20keys.%20The%20behavior%20of%20a%20map%20is%20not%20specified%20if
  - 気をつけないと、挙動が「not specified」だという
    - Javaの「not specified」がどういう意味かは知らない
    - C++だと本当に何が起こるかわからないと聞いたことがある。コメント集にもあった。「undefined behavior」 https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.j7tbto3qup6b
    - C/C++に特有だと言っている人がいる https://stackoverflow.com/questions/39886415/does-java-have-undefined-behavior-like-c-does
    - ChatGPTに訊いてもJavaにはUBはないと言われた：「Instead of UB, you typically get: - A defined result - A runtime exception -　Or specified “unspecified but bounded” behavior」「This means multiple valid outcomes, not “anything can happen”」など
    - 「not specified」の定義がドキュメントに書いてあるのかはわからなかった
  - 「While it is permissible for a map to contain itself as a value, extreme caution is advised: the `equals` and `hashCode` methods are no longer well defined on such a map.」
  - ここでの「well defined」とは？
    - 安易に考えると、コンパイルできるが無限ループになるとか？
    - 実際にやってみたらそうなった（コードは以下）。「Exception in thread "main" java.lang.StackOverflowError」
  - そもそも、自身を含む `HashMap` とか作りたくないけど

```java
var m = new HashMap<String, Object>();
m.put("", m);
var m2 = new HashMap<String, Object>();
m2.put("", m2);
System.out.println(m.equals(m2));
```

- （続き）
  - ちなみに、`m.equals(m)` はtrueになった。`==` の場合は別扱いしているんだろうか。
    - https://docs.oracle.com/javase/specs/jls/se26/html/jls-15.html#jls-15.21.3
    - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/lang/Object.html#equals(java.lang.Object)
    - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/AbstractMap.html#equals(java.lang.Object)
    - reflexiveの要請のほかは、特にドキュメントに書いてはいなさそう
    - あんまりこんな変な状況のことを調べてもナンかもしれない
  - 元の話題に戻る。そもそも、キーがmutableでも良いということになっている（great careが必要と言っているが）ほうが意外だった
    - とはいえ、自分では使わないようにしようと思った
  - `HashMap` の場合、キーのhashが「まともな」ものであって欲しい気もする
    - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/HashMap.html
    - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/lang/Object.html#hashCode()
    - そのあたりは大したこと書いてなさそう？ 「However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.」

olsen-blue氏のコードの話に戻る。
- goto-untrapped氏のコードでいう `sorted` の変数名が、olsen-blue氏のコードでの `common_key` である
  - この変数名はちょっと引っかかった
  - アナグラムたちに共通するキー、ぐらいの意味か
  - うーん、そう言われると真っ当な気もする
  - いきなり「共通のキーと単語をしまう箱を用意します」（`common_key_to_words = defaultdict(list)`）と言われると、何に共通するの？とは思う
  - 「共通のキーとして、単語をソートしたものを使います」（`common_key = tuple(sorted(word))`）まで見れば、なるほど、と思うかも
  - 良いか悪いかよくわからない
- 「度数分布」という言い方、たしかに https://github.com/olsen-blue/Arai60/pull/12#pullrequestreview-2548365222
- frozenset https://github.com/olsen-blue/Arai60/pull/12#discussion_r1912037436
  - Javaにそういうものがあるか、ChatGPTに訊いてみた
  - `Set.of(...)`、`Collections.unmodifiableSet`、Third-Party Libraries: guava を提示された
  - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/Set.html#unmodifiable
  - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/Collection.html#unmodview


コメント集も見る
- frozenset を使う方法 https://github.com/kazukiii/leetcode/pull/13#discussion_r1642894457
  - 書いてみよう

```java
class Solution {
  public List<List<String>> groupAnagrams(String[] strs) {
    Map<Map<Character, Integer>, List<String>> countsToAnagrams = new HashMap<>();
    for (String str : strs) {
      Map<Character, Integer> charToCount = new HashMap<>();
      for (char c : str.toCharArray()) {
        int count = charToCount.getOrDefault(c, 0);
        charToCount.put(c, count + 1);
      }
      charToCount = Map.copyOf(charToCount);

      if (!countsToAnagrams.containsKey(charToCount)) {
        countsToAnagrams.put(charToCount, new ArrayList<>());
      }
      countsToAnagrams.get(charToCount).add(str);
    }

    return new ArrayList<>(countsToAnagrams.values());
  }
}
```

- 一発で通らず、結構間違えた。20分ぐらいかかった
  - コードをちゃんと読み直す前にsubmitして（これは良くない）、その後デバッグして時間がかかったりした
  - `Map` のドキュメントで欲しいメソッドを探すのにも時間がかかった
- `charToCount = Map.copyOf(charToCount);` は、このコードならやらなくて良いかも

引き続きコメント集を見る
- `charToCount` を文字列に変換する人もいる。そのエンコードの方法について https://github.com/ichika0615/arai60/pull/11#discussion_r1975712971
  - 例：`http` → `h1p1t2`
  - 「この場合の問題文ではないようですが、数字が入力に来ると衝突しますね。」（例：`a1` → `a111`）
  - 「エスケープシーケンス」「[可変長数値表現](https://ja.wikipedia.org/wiki/可変長数値表現)も参照」とのこと
  - エスケープシーケンスは、たとえば、`c` が数字のときは `#c` に置き換え、`#` そのものは `##` に置き換える、など（例：`55` → `#52`）
  - 可変長数値表現とは？
    - バイト列で数値列を表現
    - たとえば、数値列 `10, 3, 2026` を `1. 0$ 3$ 2. 0. 2. 6$` とエンコードする、みたいな雰囲気（どちらかというと `.1 $0 $3 .2 .0 .2 $6` だが）
    - `$` `.` は1bitで表現できる。残る7bitを数値の表現に使う
  - ichika0615氏は「絶対に現れない記号(`'$'`とかでしょうか)を間に挟む」「`character + '$' + str(count)`」と書いている
    - むしろ `'$' + character + str(count)` でも良いか。`character` は1文字だから
  - この問題と可変長数値表現の関係はよくわからなかった


# step 2.2 : 清書
step 1に載せたgoto-untrapped氏のコードと同一なので省略


# step 3
1回目：4:13、2回目:3:10、3回目：2:45

- 本当に丸暗記になっている感じがあり、これじゃ意味が薄いかも
- とりあえず、もう少し考えながらもう一度やっておく

4回目：3:55

- コードは、step 1に載せたgoto-untrapped氏のコードと同一なので省略
  - ……というのは不自然なような気もしてきた。他の選択肢が見えてないということなのでは？


## 反省： step 2.1 の続き
- そこで、コードを見直してみたが、ほぼ何も思いつかない
- [ryoooooory氏のコード](https://github.com/ryoooooory/LeetCode/pull/1/changes)はざっと眺めただけだった。もう少し読んでみる
  - いろいろの違いはあるものの、どの点もgoto-untrapped氏のコードのほうが好みに合う気がする
  - ryoooooory氏は、`sortedToAnagrams` を経由せず直接にresultを構築
  - `String.valueOf` vs `new String`
  - 変数名が参考になる
    - goto-untrapped氏のコードに取り入れるなら `sortedToGroup` とかか？ この場合は `sortedToAnagrams` のほうがわかりやすい気がする
  - `sortedToAnagrams` に追加する処理の書き方
    - `if (sortedToAnagrams.containsKey(sorted)) { ... } else { ... }` のようになっている
    - その部分だけ書いてみる

```java
      if (sortedToAnagrams.containsKey(sorted)) {
        sortedToAnagrams.get(sorted).add(str);
      } else {
        sortedToAnagrams.put(sorted, new ArrayList<>(Collections.singletonList(str)));
      }
```

- 取り組み方に何かと課題を感じるが、とりあえず今回はここでストップ
  - 2日空いてしまった。時間がかかりすぎてしまったのが理由の一つだ
  - 今の感じだと「調べ学習」がメインになってしまっている感がある
    - （それはそれで勉強になっている気はする、が……）
  - 練習のメインの目的からは、少しズレているのではないか
    - ひとまず、コードの読み・書きのウェイトをもっと増やすべきなのでは
  - ちょっと今のままでは効率が悪い気がしている

