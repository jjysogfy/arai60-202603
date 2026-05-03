
# ファイル構成
このファイルより先に`memo.md`をご覧ください。


# step 2

## 清書
- 清書
- `"h*t"` -> `["hot", "hat"]`のような辞書を作る方法
```java
// step 2 清書 `"h*t"`のような文字列を使う
public class Solution {
  // Returns 0 if there is no ladder
  public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Map<String, List<String>> adjacency = buildGraph(beginWord, wordList);

    List<String> words = new ArrayList<>();
    words.add(beginWord);
    Set<String> visited = new HashSet<>();
    visited.add(beginWord);
    int length = 1;
    while (!words.isEmpty()) {
      List<String> nextWords = new ArrayList<>();
      for (String word : words) {
        if (word.equals(endWord)) {
          return length;
        }
        addNeighbors(adjacency, word, nextWords, visited);
      }

      words = nextWords;
      ++length;
    }

    return 0;
  }

  String buildPattern(String word, int i) {
    char[] wordArray = word.toCharArray();
    wordArray[i] = '*';
    return new String(wordArray);
  }

  Map<String, List<String>> buildGraph(String beginWord, List<String> wordList) {
    Map<String, List<String>> adjacency = new HashMap<>();
    List<String> words = new ArrayList<>(wordList);
    if (!wordList.contains(beginWord)) {
      words.add(beginWord);
    }
    for (String word : words) {
      for (int i = 0; i < word.length(); ++i) {
        String pattern = buildPattern(word, i);
        adjacency.computeIfAbsent(pattern, p -> new ArrayList<>())
            .add(word);
      }
    }
    return adjacency;
  }

  void addNeighbors(Map<String, List<String>> adjacency,
      String word, List<String> nextWords, Set<String> visited) {
    for (int i = 0; i < word.length(); ++i) {
      for (String nextWord : adjacency.get(buildPattern(word, i))) {
        if (visited.contains(nextWord)) {
          continue;
        }
        visited.add(nextWord);
        nextWords.add(nextWord);
      }
    }
  }
}
// step 2 清書 終わり
```


## step 2 他のコード

- wordListの二重ループで隣接リストを作るコード
- 23分かかった
- 迷ったところ
  - エラー処理をするかどうか
    - ほぼ何もしないことに
  - BFSの書き方
    - 少しは慣れたつもりだったが、こうして複雑になるとちょっと混乱
    - 提出前にじっと見直して、`return length;`が無い、とか気づいた
  - buildGraphの処理
    - `copyOfWordList.add(beginWord)`みたいにすればループが1つにまとまるはず
    - beginWordへの辺があるのは変だと思い（あまり根拠はない）、そうはしなかった
    - 提出してから気づいたが、beginWordがwordListに含まれる場合を検討してなかった

```java
// step 2 その1 wordListの二重ループで隣接リストを作る
public class Solution {
  // Returns 0 if there is no ladder
  public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Map<String, List<String>> wordToNeighbors = buildGraph(beginWord, wordList);

    Queue<String> words = new ArrayDeque<>();
    words.add(beginWord);
    int length = 1;
    Set<String> visited = new HashSet<>();
    visited.add(beginWord);
    while (!words.isEmpty()) {
      Queue<String> nextWords = new ArrayDeque<>();
      while (!words.isEmpty()) {
        String word = words.remove();
        if (word.equals(endWord)) {
          return length;
        }

        for (String neighbor : wordToNeighbors.get(word)) {
          if (visited.contains(neighbor)) {
            continue;
          }
          visited.add(neighbor);
          nextWords.add(neighbor);
        }
      }
      words = nextWords;
      ++length;
    }

    return 0;
  }

  boolean isAdjacent(String word1, String word2) {
    int distance = 0;
    for (int i = 0; i < word1.length(); ++i) {
      if (word1.charAt(i) != word2.charAt(i)) {
        ++distance;
      }
      if (distance > 1) {
        return false;
      }
    }
    return distance == 1;
  }

  Map<String, List<String>> buildGraph(String beginWord, List<String> wordList) {
    Map<String, List<String>> wordToNeighbors = new HashMap<>();
    List<String> neighborsOfBeginWord = new ArrayList<>();
    wordToNeighbors.put(beginWord, neighborsOfBeginWord);
    for (String word : wordList) {
      if (isAdjacent(beginWord, word)) {
        neighborsOfBeginWord.add(word);
      }
    }
    for (String word1 : wordList) {
      List<String> neighbors = new ArrayList<>();
      wordToNeighbors.put(word1, neighbors);
      for (String word2 : wordList) {
        if (isAdjacent(word1, word2)) {
          neighbors.add(word2);
        }
      }
    }
    return wordToNeighbors;
  }
}
```

- `("h", "t")`のような組を使う方法（`"h*t"`のような文字列ではなく）
- 型名`WordPair`は`Pattern`のままで良かったかな

```java
public class Solution {
  record WordPair(String former, String latter) {
  }

  // Returns 0 if there is no ladder
  public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Map<WordPair, List<String>> adjacency = buildGraph(beginWord, wordList);

    List<String> words = new ArrayList<>();
    words.add(beginWord);
    Set<String> visited = new HashSet<>();
    visited.add(beginWord);
    int length = 1;
    while (!words.isEmpty()) {
      List<String> nextWords = new ArrayList<>();
      for (String word : words) {
        if (word.equals(endWord)) {
          return length;
        }
        addNeighbors(adjacency, word, nextWords, visited);
      }

      words = nextWords;
      ++length;
    }

    return 0;
  }

  WordPair buildWordPair(String word, int i) {
    return new WordPair(word.substring(0, i), word.substring(i + 1));
  }

  Map<WordPair, List<String>> buildGraph(String beginWord, List<String> wordList) {
    Map<WordPair, List<String>> adjacency = new HashMap<>();
    List<String> words = new ArrayList<>(wordList);
    if (!wordList.contains(beginWord)) {
      words.add(beginWord);
    }
    for (String word : words) {
      for (int i = 0; i < word.length(); ++i) {
        WordPair wordPair = buildWordPair(word, i);
        adjacency.computeIfAbsent(wordPair, p -> new ArrayList<>())
            .add(word);
      }
    }
    return adjacency;
  }

  void addNeighbors(Map<WordPair, List<String>> adjacency,
      String word, List<String> nextWords, Set<String> visited) {
    for (int i = 0; i < word.length(); ++i) {
      WordPair wordPair = buildWordPair(word, i);
      for (String nextWord : adjacency.get(wordPair)) {
        if (visited.contains(nextWord)) {
          continue;
        }
        visited.add(nextWord);
        nextWords.add(nextWord);
      }
    }
  }
}
```

