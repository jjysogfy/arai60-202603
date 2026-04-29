問題（127. Word Ladder）：https://leetcode.com/problems/word-ladder/

# step 1
```java
class Solution {
  static final char[] ALPHABETS;
  static {
    int numAlphabets = 'z' - 'a' + 1;
    ALPHABETS = new char[numAlphabets];
    for (int i = 0; i < numAlphabets; ++i) {
      ALPHABETS[i] = (char) ('a' + i);
    }
  }

  record WordAndNum(String word, int num) {}

  public static int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> permitted = new HashSet(wordList);
    Queue<WordAndNum> wordsToVisit = new ArrayDeque<>();
    wordsToVisit.offer(new WordAndNum(beginWord, 1));
    Set<String> visited = new HashSet<>();

    while (!wordsToVisit.isEmpty()) {
      WordAndNum wordAndNum = wordsToVisit.poll();
      String word = wordAndNum.word();
      if (word.equals(endWord)) {
        return wordAndNum.num();
      }

      for (int i = 0; i < word.length(); ++i) {
        for (char c : ALPHABETS) {
          String nextWord = alteredAt(word, i, c);
          maybeOffer(nextWord, wordAndNum.num() + 1, permitted, wordsToVisit, visited);
        }
      }
    }
    return 0;
  }

  private static String alteredAt(String word, int i, char c) {
    return word.substring(0, i) + c + word.substring(i + 1, word.length());
  }

  private static void maybeOffer(String word, int num, Set<String> permitted,
      Queue<WordAndNum> wordsToVisit, Set<String> visited) {
    if (!permitted.contains(word)) {
      return;
    }
    if (visited.contains(word)) {
      return;
    }
    wordsToVisit.offer(new WordAndNum(word, num));
    visited.add(word);
  }
}
```

- 29分。スペルミスを除いて一発で通った
  - 提出前、見直したもののどうせ何か見落としてるんだろうと思っていたので、正直うれしいと思った
- 問題文はしばらく前に読んで、解き方を考えてもいたが、書いたことはなかった
- ALPHABETSの初期化は、あまり上手くない気がする
  - 1行で簡単に書きたかったが、思いつかずやめた
    - IntStreamはあるがCharStreamはない
    - 追記：サロゲートペアを考えると、`char[]`より`int[]`にすべきで、それなら書けるかも
    - 追記：`int[] ALPHABETS = IntStream.rangeClosed('a', 'z').toArray()`
    - 追記：`alteredAt`も変更が必要。`c`の代わりに`Character.toString(c)`とするなど
    - 追記：`String[]`という手もあるか
    - 追記：ただ、実際にサロゲートペアがあると`ladderLength`自体は壊れそう
  - static初期化ブロックを初めて使った
- WordAndNumまわりの変数名は迷った
  - はじめ`Queue<WordAndNum> wordsToVisit`を`Queue<String>`で書いていた
  - `wordsAndNumsToVisit`とかはかえってわかりづらいと思い変数名はそのままにした
- 実行時間について
  - ステップ数はだいたい、`wordList.length * ALPHABETS.length * word.length^2`つまり`10^5 * 10^2`ぐらい。Javaは1秒`10^6`から`10^7`程度
    - nextWordの計算で`word.length`がひとつ掛かると思う
  - `10^7`ステップだと危ういなあと思いつつ書いていた
  - 結果は272ms

