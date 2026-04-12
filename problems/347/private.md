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

# step 2
- https://github.com/goto-untrapped/Arai60/pull/55/changes
  - `entrySet`じゃなく`keySet`を使って、`Comparator`で`numToCount`にアクセスしている。なるほど
  - 「JavaのPriorityQueueへの要素追加はofferだっけと思って調べたらofferとadd2種類あるんですね」
  - （Stream APIについて）「なんだかんだループで書くほうが自由度が高いことが多いので。」
  - コードは一部しか読めていないが、疲れたのでやめる

