問題：https://leetcode.com/problems/top-k-frequent-elements/

# step 1
- ソートの書き方を迷った。結局、ラムダで`Comparator`を直接書いた
- はじめに書いたのは`numAndCount.sort(Comparator.comparing(Map.Entry<Integer, Integer>::getValue).reversed());`
  - reversedしたせいで`Map.Entry<Integer, Integer>::getValue`と長い式を書かなくてはいけなくなってしまった
  - 次のように2行に分けるかなあ
  - `Comparator<Map.Entry<Integer, Integer>> countComparator = Comparator.comparing(Map.Entry::getValue);`
  - `numAndCount.sort(countComparator.reversed());`
- 追記：長さが`k`未満のときに`subList`で例外が出てしまう


# step 2

TreeMapを使う（PriorityQueueを使う方法と同じ）：
```java
    Comparator<Integer> compareCounts = Comparator.comparingInt(numToCount::get);
    TreeSet<Integer> countToNum =
      new TreeSet<>(compareCounts.thenComparing(Comparator.naturalOrder()));
    for (Integer num : numToCount.keySet()) {
      countToNum.add(num);
      if (countToNum.size() > k) {
        countToNum.pollFirst();
      }
    }

    return countToNum.stream().mapToInt(Integer::intValue).toArray();
```

- 理解していなかったが`.thenComparing(Comparator.naturalOrder())`は必要。これが無いとcountが同じnumたちが省かれてしまう！
  - Comparatorを1文にまとめるなら、
  - `Comparator.comparingInt((Integer num) -> numToCount.get(num))`
  - `.thenComparing(Comparator.naturalOrder())`
  - などと、ラムダで型を指定すれば良い
  - （ChatGPTに訊いた）
