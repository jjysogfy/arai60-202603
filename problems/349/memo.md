問題：[349. Intersection of Two Arrays](https://leetcode.com/problems/intersection-of-two-arrays/)


# ファイルの構成
- memo.md: 主要部分
  - このファイルを見てくだされば大丈夫です
- side_notes.md: 脇にそれた調べごとなど


# step 1
- 解いたことがある
  - この問題のコメント集も、勝手にちょっと読んでしまったことがあるかもしれません
- いろいろやり方はあるか
  - `nums1`を`Set`に変換し、`nums2`の各要素がそこに現れるか調べる
  - `nums2`に同じ要素が2回出ると困る
  - 重複を除くため、`nums2`も`Set`にする

```java
class Solution {
  public int[] intersection(int[] nums1, int[] nums2) {
     Set<Integer> uniqueNums1 = new HashSet<>();
     for (int num : nums1) {
      uniqueNums1.add(num);
     }
     Set<Integer> uniqueNums2 = new HashSet<>();
     for (int num : nums2) {
      uniqueNums2.add(num);
     }
     List<Integer> intersection = new ArrayList<>();

     for (int num : uniqueNums2) {
      if (uniqueNums1.contains(num)) {
        intersection.add(num);
      }
     }

     var unboxedIntersection = new int[intersection.size()];
     for (int i = 0; i < intersection.size(); ++i) {
      unboxedIntersection[i] = intersection.get(i);
     }
     return unboxedIntersection;
  }
}
```

- Javaのことがわからず、調べていたら時間がかかった。なんだかんだ1時間ぐらい？
- `nums1`の`Set`への変換がわからなかった
  - （適当に`Set.of(nums1)`とか書いて間違えた）
- `List<Integer>`から`int[]`への変換もわからなかった
  - 何か良いやり方があるだろうと、ChatGPTに訊いたりGoogle検索したり
    - `Stream`などと言われた
    - これは知らなきゃいけないと思ったが、調べると時間がかかるので、**一旦飛ばす**
    - （`Stream`を使う方法も、多少は煩雑に見えるが……）
  - 結局、どちらも安直な方法で書いた
  - 変数名が`intersection`、`unboxedIntersection`などと長いこともあり、煩雑
- 変数名は`boxedIntersection`、`intersection`のほうがマシだったかも？
  - いきなり`boxed`とか言われると少しだけ戸惑うかと思いこうしていたが、返り値の型があるので十分察せるか。……とか言うまでもなく、普通にわかるかも
  - 他には思いつかない
- アルゴリズムはたくさんありそう
  - `Set`を使うか、ソートするか
  - `nums1[i]`の動く範囲が小さいので、度数分布を配列で扱うことも可能ではある
  - 今回は出力の順序はなんでも良いが、決まっていて欲しいケースもあるのかもしれない
    - 上のアルゴリズムだと、順序は何も保証されなさそう


# step 2
まず、Javaでたくさん解いている方のコードを読む。
- goto-untrapped氏 https://discord.com/channels/1084280443945353267/1233603535862628432/1234331603359236167
  - レビューが付いてない？
- ryoooooory氏は解いていなさそう
  - （名前が似ている別の問題 https://discord.com/channels/1084280443945353267/1218823830743547914/1241419055701823508 ）

とりあえず、goto-untrapped氏を見る
- IntersectionOfTwoArraysStep1.javaについて
- `existNumMap`は`Map`ではなく`Set`で十分
  - 追記：変数名も気になる。まず自然言語では「`nums1`に現れる数のSet」。……こういうのは困るが、やっぱり上述のように`uniqueNums1`あたりか？
- `List<Integer> intersectionNums`の宣言は、代入の直前に動かしたほうがわかりやすい
- 好みかもだが、変数名`ans`は、`intersectionNums`との関係が見づらい
- （`intersectionB`のほうは、動かないコード）
- Lines 44: ifはいらないのでは
- 変数名`uniqueBits`はわかりづらい
  - 良い案は持ってない。強いていえば`intersectionBits`のがマシ
  - `Bits`と付けるのは良いのかわからないが、ビットで集合を表すときの、通じやすい変数名を知らない
- Lines 58-64: ビットで集合を表すときのコード、こういう書き方になるんだ
  - `num`が1つずつ増えるから`for`で書きたい気もしたが、やっぱりよく見る形じゃない`for`は書きたくないかも
- 最後の`Stream`使うところはよくわかってない
  - また出てきたので、一旦、ChatGPTに訊いて雰囲気だけ触れておく
  - そうやってなあなあで進める。あまりよくないが、今回は、コードを読む方を優先してやってみることにしている

- IntersectionOfTwoArraysStep2.javaを読む
- `intersection2_1`は、動くことはわかるという意味では読めるが、どういう気持ちなのかはよくわからなかった
  - `numToCount` が複雑な変数だと感じる
    - keyは、`nums1`に現れる数すべて
    - （最終的な）対応するvalueは、`nums1`にだけ現れるなら1で、両方に現れるなら0

- `intersection2_2`について
- Line 66: `ans[index++] = num;`はインクリメントの返り値を使っていてちょっと難しい
- Line 50-60: メインの処理
  - 同じ値が続くとき、何度もその値をaddすることになる
  - `List`を直接作っていたら重複することになってまずいが、このコードでは`Set`を使うので大丈夫
  - コードの見た目は単純になって良いが、「手でやる」ときにそういうことはしないような気もする
    - ……そうでもないか、そんなに変なことはしてない気もしてきた

- `intersection2_3`
- `retainAll`、`addAll`というメソッドを使う。なるほど
  - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/Collection.html#retainAll(java.util.Collection)
- `set2`不要では？ `.retainAll(nums2)`など
  - ……実行して気付いたが、arrayを`List`などに変換する必要があり、ダメ
  - `Set`じゃなく`List`でも良いのはともかく、そんなに無駄な処理ではなかったかも
- arrayの処理
  - 長さを変更できないので、Line 115: `new int[set1.size()]`と長めに作ってから、Line 122: `Arrays.copyOf(ans, index)`で`index`個だけ切り出す
- Line 107: `if (set1.size() < set2.size())` これは高速化のためだと思うが、実際どのぐらい効くんだろう


コメント集を見てみる
- https://discord.com/channels/1084280443945353267/1303257587742933024/1319664094235594824
  - 片方がとても大きいがソートされていると条件を変えたとき、どんなコードを書きますか？ という話
  - なるほど：「この問題の推定される出題意図は条件を変えたときに案がいくつか出てくるかです」
- https://discord.com/channels/1084280443945353267/1250449116975075328/1324495144522612758
  - 変数名`i`、`j`の話
  - その話とはちょっとずれるが、この「解法3」のコードは読みやすく感じた
    - 方法としては、ソートして2つのポインタを持つ方法
  - 数行のかたまりごとに、だいたい何をやってそうかパッと見で見当がつく
    - 同じようなコード読んだ直後だから、という原因はもちろんあると思うが
  - Lines 147-150: `unique_nums1`の役割が微妙に難しい気がする
    - 一度`nums2`に出た要素は削除するせい（Line 149）
    - 「`intersected_nums`に入る可能性のある数」ぐらいの意味か
    - 変数名を`possibly_in_intersection_nums`にするとか？ うーん、かえってわかりづらくなったかもしれない
  - Line 52: 「解法3」について、「nums1またはnums2の要素数が莫大でメモリ上に乗らない場合でも、ソート済みであれば要素を一つずつ取ってきて一致するかどうかを確かめられる」とあり、なるほど
  - set vs unordered_set https://github.com/aoshi2025s/leetcode-review/pull/2#discussion_r1900302147
    - Javaの`HashSet` vs `TreeMap`は？
    - 「こういうのを読むのは最終ゴールなので」とのことなので、**一旦放置する**


---

ここまでで、3時間ぐらいは使ってしまった。
- 取り組み方を変えてみたのだった
  - 調べることばかりになりがちだったので、今回は人のコードを読むことを優先してみた
- これでよかったかどうかは検討中
- 時間をかなり使ったので、ここで一区切りとして、次のstepに進む


# step 2.2 : 清書


