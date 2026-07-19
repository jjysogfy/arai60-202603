
# ファイル構成
このファイルより先に`memo.md`をご覧ください。


# step 2

## 清書
- 清書
- `"h*t"` -> `["hot", "hat"]`のような辞書を作る方法
- 40msほど
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
- 1187ms

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
// step 2 その1 終わり
```

- `("h", "t")`のような組を使う方法（`"h*t"`のような文字列ではなく）
- 型名`WordPair`は`Pattern`のままで良かったかな
- 72ms

```java
// step 2 その2 文字列の組をパターンとして使う方法
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
// step 2 その2 終わり
```

- 清書に近い方法。`"h*t"`のような文字列を使う方法。ただし文字列そのものよりはindexを扱う
- かかった時間：16:52

```java
// step 2 その3 文字列そのものよりindexを扱う方法
public class Solution {
  // Returns 0 if there is no ladder
  public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    List<String> extendedWordList = new ArrayList<>(wordList);
    extendedWordList.add(beginWord);
    Map<String, List<Integer>> patternToNeighbors = buildGraph(extendedWordList);

    List<Integer> wordIndexes = new ArrayList<>();
    wordIndexes.add(wordList.size());
    boolean[] visited = new boolean[extendedWordList.size()];
    visited[wordList.size()] = true;
    int length = 1;
    while (!wordIndexes.isEmpty()) {
      List<Integer> nextWordIndexes = new ArrayList<>();
      for (int wordIndex : wordIndexes) {
        String word = extendedWordList.get(wordIndex);
        if (word.equals(endWord)) {
          return length;
        }
        addNeighbors(patternToNeighbors, word, nextWordIndexes, visited);
      }
      wordIndexes = nextWordIndexes;
      ++length;
    }

    return 0;
  }

  String buildPattern(String word, int i) {
    char[] wordArray = word.toCharArray();
    wordArray[i] = '*';
    return String.valueOf(wordArray);
  }

  Map<String, List<Integer>> buildGraph(List<String> wordList) {
    Map<String, List<Integer>> patternToNeighbors = new HashMap<>();
    for (int wordIndex = 0; wordIndex < wordList.size(); ++wordIndex) {
      String word = wordList.get(wordIndex);
      for (int i = 0; i < word.length(); ++i) {
        String pattern = buildPattern(word, i);
        patternToNeighbors.computeIfAbsent(pattern, _k -> new ArrayList<>())
            .add(wordIndex);
      }
    }
    return patternToNeighbors;
  }

  void addNeighbors(Map<String, List<Integer>> patternToNeighbors,
      String word, List<Integer> nextWordIndexes, boolean[] visited) {
    for (int i = 0; i < word.length(); ++i) {
      String pattern = buildPattern(word, i);
      for (int nextWordIndex : patternToNeighbors.get(pattern)) {
        if (visited[nextWordIndex]) {
          continue;
        }
        visited[nextWordIndex] = true;
        nextWordIndexes.add(nextWordIndex);
      }
    }
  }
}
// step 2 その3 終わり
```

