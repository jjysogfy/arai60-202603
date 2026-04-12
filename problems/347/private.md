問題：https://leetcode.com/problems/top-k-frequent-elements/

# step 1
21分。Javaがわからないせいで時間がかかった。
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

- ソートの書き方を迷った。結局、ラムダで`Comparator`を直接書いた
- はじめに書いたのは`numAndCount.sort(Comparator.comparing(Map.Entry<Integer, Integer>::getValue).reversed());`
  - reversedしたせいで`Map.Entry<Integer, Integer>::getValue`と長い式を書かなくてはいけなくなってしまった
  - 次のように2行に分けるかなあ
  - `Comparator<Map.Entry<Integer, Integer>> countComparator = Comparator.comparing(Map.Entry::getValue);`
  - `numAndCount.sort(countComparator.reversed());`
- 追記：長さが`k`未満のときに`subList`で例外が出てしまう


# step 2
- https://github.com/goto-untrapped/Arai60/pull/55/changes
  - `entrySet`じゃなく`keySet`を使って、`Comparator`で`numToCount`にアクセスしている。なるほど
  - 「JavaのPriorityQueueへの要素追加はofferだっけと思って調べたらofferとadd2種類あるんですね」
  - （Stream APIについて）「なんだかんだループで書くほうが自由度が高いことが多いので。」
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
  - 「Map.Entry を pair や tuple の代わりに使っている点に違和感を感じました。」
    - なるほど、これ良くないのか
  - 「forでも問題ありませんが、iterator を使っても良さそうです」

TreeMapを使う（PriorityQueueを使う方法と同じ）：
```java
    Comparator<Integer> compareCounts = Comparator.comparing(numToCount::get);
    TreeSet<Integer> countToNum = new TreeSet<>(compareCounts.thenComparing(Comparator.naturalOrder()));
    for (Integer num : numToCount.keySet()) {
      countToNum.add(num);
      if (countToNum.size() > k) {
        countToNum.pollFirst();
      }
    }

    return countToNum.stream().mapToInt(Integer::intValue).toArray();
```

- Comparatorを使うとコードが長くなりがちと感じる
  - （型推論が弱いのも面倒。`reversed`を気楽に使えない）
  - 理解していなかったが`.thenComparing(Comparator.naturalOrder())`は必要。これが無いとcountが同じnumたちが省かれてしまう！
- TreeSetはNavigableSetで受けたほうが良いのかな

少しドキュメントを見たりして書き直す。
- `comparingInt`を使うと型推論が効いてくれる
- Stream.limitを知った
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

Stream APIの代わりにループを使う。iterator、というかfor-eachを使ってみる
```java
class Solution {
  public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> numToCount = new HashMap<>();
    for (int num : nums) {
      numToCount.merge(num, 1, Integer::sum);
    }

    List<Integer> sortedNums = new ArrayList<>(numToCount.keySet());
    sortedNums.sort(Comparator.comparingInt(numToCount::get).reversed());
    // int[] result = new int[k];
    // for (int i = 0; i < k; ++i) {
    //   result[i] = sortedNums.get(i);
    // }
    // return result;
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
