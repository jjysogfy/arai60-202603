問題：[49. Group Anagrams](https://leetcode.com/problems/group-anagrams/)


# step 1 : 解く
解くまえ・解くあいだに思ったこと：
- 解いたことがある
  - anagramかどうかの判定：どの文字が何回現れるか、という「カウント」を `Map` なり array なりで保持し、それが等しいか判定すればいい
  - その「カウント」をソートして、各groupが連続して並ぶようにする（例：`["bat", "nat","tan", "ate","eat","tea"]`）
  - 各groupごとにまとめる
  - 「カウント」をkeyとする `Map` を作る手もあるか
- Javaの書き方がわからない
  - この状況でのソート（配列の `Comparator`？）
  - 配列などをkeyとする `Map`？
    - 追記：少なくとも普通の配列は可変なので、やばそう
- さっさと人の答えを読もう
- ここまで書くのになんだかんだ15分ぐらいかかっている気がする

Javaでたくさん解いている人のコードを参照する。
- https://github.com/goto-untrapped/Arai60/pull/6/changes
- https://github.com/ryoooooory/LeetCode/pull/1/changes
- goto-untrapped氏の `GroupAnagramsStep4.java` を見てみた
  - 賢い、「カウント」ではなく、ソートした文字列を扱う（例：`["abt", "ant","ant", "aet","aet","aet"]`）
  - いや、たぶん自分も前に見ているが、すっかり忘れていた
  - 変数名も考えられていると感じる

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

    return ArrayList<>(sortedToAnagrams.values());
  }
}
```


# step 2 : 読む
上述のgoto-untrapped氏、ryoooooory氏のほか、

