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
    - ChatGPTに提示されたのは、`set = Arrays.stream(arr).boxed().collect(Collectors.toSet());`と`arr = list.stream().mapToInt(Integer::intValue).toArray();`
    - `Stream`が出てきていて、これは知らなきゃいけないと思ったが、調べると時間がかかるので、**一旦飛ばす**
  - 結局、どちらも安直な方法で書いた
    - 煩雑ではある。変数名が`intersection`、`unboxedIntersection`などと長いせいもある
- 変数名は`boxedIntersection`、`intersection`のほうがマシだったかも？
  - いきなり`boxed`とか言われると少しだけ戸惑うかと思いこうしていた
  - 返り値の型があるので十分察せるか。……とか言うまでもなく、普通にわかるかも
  - 他の変数名は思いつかない
  - 追記：関数名と変数名がかぶっていた。あんまりよくないのかも
- アルゴリズムはたくさんありそう
  - `Set`を使うか、ソートするか
  - `nums1[i]`の動く範囲が小さいので、度数分布を配列で扱うことも可能ではある
    - （`nums1`と`nums2`の度数分布を求めれば、intersectionも計算できる）
  - 今回は出力の順序はなんでも良いが、決まっていて欲しいケースもあるのかもしれない
    - 上のアルゴリズムだと、順序は何も保証されなさそう


# step 2
まず、Javaでたくさん解いている方のコードを読む。
- goto-untrapped氏 https://discord.com/channels/1084280443945353267/1233603535862628432/1234331603359236167
  - レビューが付いてない？
- ryoooooory氏は解いていなさそう
  - （名前が似ている別の問題 https://discord.com/channels/1084280443945353267/1218823830743547914/1241419055701823508 ）

コメント集も参照した
- aoshi2025s氏 https://discord.com/channels/1084280443945353267/1303257587742933024/1319664094235594824

読んでいて思ったことなどを書いていたら、かなり長くなってしまった。
そういう部分は、レビューしていただく中心部分ではないと感じるので、別ファイル`side_notes.md`に移動させた。

---

ここまでで、3時間ぐらいは使ってしまった。
- 取り組み方を変えてみたのだった
  - 調べることばかりになりがちだったので、今回は人のコードを読むことを優先してみた
- 今回のやりかたも、あまりうまくなかったなと思った
  - もう少し、コードを書くほうのウェイトを増やしたほうが良い気がする
  - そもそも、なんでも細かく書きすぎのような気もする
    - コメント集：「返事が長い事自体が、エンジニアリングをしていないので」 https://discord.com/channels/1084280443945353267/1301587996298182696/1338692024575987737
    - https://discord.com/channels/1084280443945353267/1245404801177616394/1253416611566850048
- もう時間をかなり使ったので、ここで一区切り。次のstepに進む


# step 2.2 : 書き換え
まず、いくつかの方法で書いてみる。
```java
// `retainAll`を使う方法
class Solution {
  public int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> intersection = new HashSet<>();
    for (int num : nums1) {
      intersection.add(num);
    }

    List<Integer> boxedNums2 = new ArrayList<>();
    for (int num : nums2) {
      boxedNums2.add(num);
    }
    intersection.retainAll(boxedNums2);

    int[] intersectionArray = new int[intersection.size()];
    int index = 0;
    for (int num : intersection) {
      intersectionArray[index] = num;
      ++index;
    }
    return intersectionArray;
  }
}
```

- 人のレビューを見ていて、この方法で書くなら`retainAll`がどういう実装か知りたくなるべき、と思った
  - それはまた時間がかかるので、**一旦置いておく**
- 変数名は相変わらず悩む
  - 変数名に`Array`とか付けなくて良い、と指摘されそうだが、こういう「型変換」っぽい場合にはわかりやすく感じてしまうかも
  - 変数名が関数名と同じだと良くないかも
- `Stream`を勉強していないので、使わないことにしているが、さすがに煩雑
  - 「型変換」のために行数の大半が費やされる
- そもそも、こういう関数で、引数・返り値を`int[]`と`List<Integer>`のどちらにするか、よくわからない

次は、[aoshi2025s氏](https://github.com/aoshi2025s/leetcode-review/pull/2/changes)の解法2とstep 3を参考にしたコード。
```java
class Solution {
  public int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> uniqueNums1 = new HashSet<>();
    for (int num : nums1) {
      uniqueNums1.add(num);
    }

    List<Integer> intersection = new ArrayList<>();
    for (int num : nums2) {
      if (!uniqueNums1.contains(num)) {
        continue;
      }
      uniqueNums1.remove(num);
      intersection.add(num);
    }

    int[] intersectionArray = new int[intersection.size()];
    for (int i = 0; i < intersection.size(); ++i) {
      intersectionArray[i] = intersection.get(i);
    }
    return intersectionArray;
  }
}
```

- 変数名`uniqueNums1`は悩む
  - `uniqueNums1.remove(num);`しているのを気にする
  - `possiblyInIntersection`、`candidates`あたりを考えたが、逆にわかりづらい気がしないではない
  - うーん、`onlyInNums1`とかどうか

