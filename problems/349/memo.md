問題：[349. Intersection of Two Arrays](https://leetcode.com/problems/intersection-of-two-arrays/)


# step 1
- 解いたことがある
- いろいろやり方はありそう
  - とりあえず、次の方法で：
  - `nums1`を`Set`に変換し、`nums2`の各要素がそこに現れるか調べる
  - `nums2`に同じ要素が2回出ると困る
  - 重複を除くため、`nums2`も`Set`にする

```java
// step 1
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
- `List<Integer>`から`int[]`への変換もわからなかった
  - 何か良いやり方があるだろうと、ChatGPTに訊いたりGoogle検索したり
    - ChatGPTに提示されたのは、`set = Arrays.stream(arr).boxed().collect(Collectors.toSet());`と`arr = list.stream().mapToInt(Integer::intValue).toArray();`
    - `Stream`が出てきていて、これは知らなきゃいけないと思ったが、調べると時間がかかるので、一旦飛ばす
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

わかったことをいくつか：
- `List<Integer> intersection`のかわりに`Set<Integer>`を使う手もある
- [`retainAll`](https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/Collection.html#retainAll(java.util.Collection))を使える
- マージソート的な方法（ソートして2つポインタを持つ）もある
- `nums1`が巨大だがソートされていて、`nums2`は小さい場合に、二分探索で処理する方法
- なるほど：「この問題の推定される出題意図は条件を変えたときに案がいくつか出てくるかです」

---

ここまでで、3時間ぐらいは使ってしまった。
- 取り組み方を変えてみたのだった
  - 調べることばかりになりがちだったので、今回は人のコードを読むことを優先してみた
- 今回のやりかたも、あまりうまくなかったなと思った
  - もう少し、コードを書くほうのウェイトを増やしたほうが良い気がする
  - そもそも、なんでも細かく書きすぎのような気がする
    - コメント集：「返事が長い事自体が、エンジニアリングをしていないので」 https://discord.com/channels/1084280443945353267/1301587996298182696/1338692024575987737
- もう時間をかなり使ったので、ここで一区切り。次に進む


いくつかの方法で書いてみる。
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
  - それはまた時間がかかるので、一旦置いておく
- 変数名は相変わらず悩む
  - 変数名に`Array`とか付けなくて良い、と指摘されそうだが、こういう「型変換」っぽい場合には良いんじゃないかと感じてしまうかも
  - 変数名が関数名と同じだと良くないかも
- 「型変換」のために行数の大半が費やされ、煩雑
- こういう関数で、引数・返り値を`int[]`と`List<Integer>`のどちらにするか、よくわからない

```java
// Streamを使うバージョン
class Solution {
  public int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> intersection = Arrays.stream(nums1).boxed().collect(Collectors.toSet());

    List<Integer> boxedNums2 = Arrays.stream(nums2).boxed().toList();
    intersection.retainAll(boxedNums2);

    return intersection.stream().mapToInt(Integer::intValue).toArray();
  }
}
```

- ChatGPTに訊いたことを参考に、`Stream`を使ったバージョン
  - `Stream`を勉強できていないので、そんなにわかってない。そのうちしたい

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
  - `possiblyInIntersection`、`candidates`、`onlyInNums1`あたりを考えたが、逆にわかりづらい気がしないではない


# step 2.2 : 清書
step 1に近い方法のが頭に入るような気もする。それで書いておく

```java
class Solution {
  public int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> uniqueNums1 = new HashSet<>();
    for (int num : nums1) {
      uniqueNums1.add(num);
    }

    Set<Integer> intersection = new HashSet<>();
    for (int num : nums2) {
      if (uniqueNums1.contains(num)) {
        intersection.add(num);
      }
    }

    int[] unboxedIntersection = new int[intersection.size()];
    int index = 0;
    for (int num : intersection) {
      unboxedIntersection[index] = num;
      ++index;
    }
    return unboxedIntersection;
  }
}
```


# step 3
1回目：2:29（ミスあり）、2回目：2:30（ミスあり）、3回目：2:25、4回目：2:38、5回目：2:15

コードはstep 2.2と同じままだったので省略。

次回：
- 順番通りなら、929. Unique Email Addresses
- その前に、Streamやジェネリクスを一度勉強すべきかもとも思っている
  - ちょっと古いが、こんなもの https://docs.oracle.com/javase/tutorial/ を見つけた
- ファイルが長すぎるし、時間もかかりすぎている（6、7時間ぐらい？）ので、何かもう少し工夫したいところ




# step 4
```java
class Solution {
  public int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> nums1Set = new HashSet<>();
    for (int num : nums1) {
      nums1Set.add(num);
    }
    Set<Integer> intersectionSet = new HashSet<>();
    for (int num : nums2) {
      if (nums1Set.contains(num)) {
        intersectionSet.add(num);
      }
    }

    int[] intersectionArray = new int[intersectionSet.size()];
    int index = 0;
    for (int num : intersectionSet) {
      intersectionArray[index] = num;
      ++index;
    }
    return intersectionArray;
  }
}
```
