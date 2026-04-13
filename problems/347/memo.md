問題：https://leetcode.com/problems/top-k-frequent-elements/

# step 1
- 解いたことがある
```java
class Solution {
  public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> numToCount = new HashMap<>();
    for (int num : nums) {
      numToCount.merge(num, 1, Integer::sum);
    }

    List<Map.Entry<Integer, Integer>> numAndCount = new ArrayList<>(numToCount.entrySet());
    numAndCount.sort((entry1, entry2) -> Integer.compare(entry2.getValue(), entry1.getValue()));
    return numAndCount
      .subList(0, k).stream()
      .mapToInt(Map.Entry::getKey)
      .toArray();
  }
}
```

- 21分。Javaがわからず時間がかかった
- ソートの書き方（Comparatorの書き方）を迷った
  - はじめに書いたのは`numAndCount.sort(Comparator.comparing(Map.Entry<Integer, Integer>::getValue).reversed());`（煩雑）
- 追記：長さが`k`未満のときに`subList`で例外が出てしまう


# step 2
- https://github.com/goto-untrapped/Arai60/pull/55/changes
  - `entrySet`じゃなく`keySet`を使っていて、Comparator内で`numToCount`にアクセスしている。なるほど
  - 「JavaのPriorityQueueへの要素追加はofferだっけと思って調べたらofferとadd2種類あるんですね」
    - そうなんだ。あまり勉強できてないが、2つあることは覚えておく
  - （Stream APIについて）「なんだかんだループで書くほうが自由度が高いことが多いので。」
    - そういうものなんだ
  - コードは一部しか読めていないが、疲れたのでやめる

`keySet`を使うと、後半はこんな感じか：
```java
    List<Integer> sortedNums = new ArrayList<>(numToCount.keySet());
    sortedNums.sort((num1, num2) -> Integer.compare(numToCount.get(num2), numToCount.get(num1)));
    return sortedNums.subList(0, k)
      .stream().mapToInt(Integer::intValue).toArray();
```

- https://github.com/ryoooooory/LeetCode/pull/16/changes
  - 「TreeMap も見ておいてください。私は、PriorityQueue よりもこちらのほうが大事な感覚があります。」
    - じゃあTreeMapで書いてみよう
    - （「清書」の部分で書きます）
  - 「Map.Entry を pair や tuple の代わりに使っている点に違和感を感じました。」
    - なるほど、これ良くないのか
  - 「forでも問題ありませんが、iterator を使っても良さそうです」

調べごとをしたメモ：
- `.comparing`でなく`.comparingInt`を使うと型推論が効いてくれる
- Stream.limitを知った

step 1（ソートする方法）の書き直し：
```java
class Solution {
  public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> numToCount = new HashMap<>();
    for (int num : nums) {
      numToCount.merge(num, 1, Integer::sum);
    }

    List<Integer> sortedNums = new ArrayList<>(numToCount.keySet());
    sortedNums.sort(Comparator.comparingInt(numToCount::get).reversed());
    return sortedNums.stream()
      .mapToInt(Integer::intValue)
      .limit(k)
      .toArray();
  }
}
```

Setから配列への変換に、Stream APIではなくiterator（というかfor-each）を使ってみる：
```java
class Solution {
  public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> numToCount = new HashMap<>();
    for (int num : nums) {
      numToCount.merge(num, 1, Integer::sum);
    }

    List<Integer> sortedNums = new ArrayList<>(numToCount.keySet());
    sortedNums.sort(Comparator.comparingInt(numToCount::get).reversed());

    int[] result = new int[k];
    int index = 0;
    for (int num : sortedNums) {
      if (index >= k) {
        break;
      }
      result[index] = num;
      ++index;
    }
    return result;
  }
}
```

## 清書
TreeSetを使う方法で書いて覚えてみる。（PriorityQueueを使っている人がいるが、その代わりにTreeSetを使っているだけ。）
```java
class Solution {
  public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> numToCount = new HashMap<>();
    for (int num : nums) {
      numToCount.merge(num, 1, Integer::sum);
    }

    Comparator<Integer> byCountThenNum = Comparator
      .comparingInt((Integer num) -> numToCount.get(num))
      .thenComparing(num -> num);
    var sortedNums = new TreeSet<Integer>(byCountThenNum);
    for (Integer num : numToCount.keySet()) {
      sortedNums.add(num);
      if (sortedNums.size() > k) {
        sortedNums.pollFirst();
      }
    }

    return sortedNums.stream().mapToInt(Integer::intValue).toArray();
  }
}
```

- `.thenComparing(Comparator.naturalOrder())`は必要（気づかなかった）
- Comparatorの書き方はChatGPTに訊いた。こうやって型推論させるのか
- `TreeSet<>`とIntegerは省略できるが、そのまま書いておく
- `TreeSet`は`NavigableSet`で受けたほうが良いのかな
- 変数名、sortedNumsよりtopKNumsのほうが良いかも？


# step 3
6分半ぐらいで書けるようになった。
