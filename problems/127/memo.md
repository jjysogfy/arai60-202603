問題（127. Word Ladder）：https://leetcode.com/problems/word-ladder/

# ファイル構成
この`memo.md`のコードを主にレビューしていただきたいです。
（ファイルが長くなりすぎないよう、一部は別ファイル`side_notes.md`としました。）


# step 1
```java
// step 1のコード
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
// step 1のコード 終わり
```

- 29分。スペルミスを除いて一発で通った
  - 提出前、見直したもののどうせ何か見落としてるんだろうと思っていたので、正直うれしいと思った
- 問題文はしばらく前に読んで、解き方を考えてもいたが、書いたことはなかった
- 方針：
  - ダイクストラ（というかBFS）をすればいい
  - でも隣接する単語をどうやってリストアップするか？
  - 隣接する単語は、高々`10 * 26`とおり（i番目の文字をa-zのどれかに変える）
  - これらのうちどれがwordListに含まれるかは、wordListをSetに変換すれば計算できる

- 思ったこといくつか：
- ALPHABETSの初期化は、あまり上手くない気がする
  - 1行で簡単に書きたかったが、思いつかずやめた
    - 追記：`char[]`でなく`int[]`や`String[]`にする手がある
    - 追記：`int[] ALPHABETS = IntStream.rangeClosed('a', 'z').toArray()`
  - static初期化ブロックを初めて使った
- WordAndNumまわりの変数名は迷った
  - はじめ`Queue<WordAndNum> wordsToVisit`を`Queue<String>`で書いていた
  - `wordsAndNumsToVisit`とかはかえってわかりづらいと思い変数名はそのままにした
- 実行時間について
  - ステップ数はだいたい、`wordList.length * ALPHABETS.length * word.length^2`
    - つまり`10^5 * 10^2`ぐらい
    - nextWordの計算で`word.length`がひとつ掛かると思う
  - 追記：Javaは1秒`10^7`から`10^8`程度。クロック周波数が`1G=10^9`オーダーで、そこから1、2桁落ちる
    - （覚え間違いで、`10^6`から`10^7`程度と思っていた）
    - （`10^7`ステップだと危ういなあと思いつつ書いていた）
  - 結果は272ms
- あとAIにレビューさせて気づいたことだが、ALPHABETSの変数名は単数形のほうが自然かも


# step 2
今回は、最近解いた方から見てみる
- https://github.com/h-masder/Arai60/pull/21/changes
- https://github.com/atmaxstar/coding_practice/pull/3/changes
- https://github.com/rimokem/arai60/pull/20/changes

あとはコメント集など、いろいろ見ている
- https://github.com/hayashi-ay/leetcode/pull/42
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1215117909450424410

- https://github.com/goto-untrapped/Arai60/pull/57
- https://github.com/ryoooooory/LeetCode/pull/23

気づいたこと
- `"h*t"` -> `["hot", "hat"]`のような辞書を作る方法
  - 賢い。グラフが特殊な形なのを利用して、効率的に隣接リストを表現してる感じ
  - step 1のコードでいう`alteredAt(word, i, c)`を`c`ごとに計算しなくてよくなる
  - 計算量が`word.length`倍改善する
  - 変数ALPHABETSを持ち出さなくていいのも良い
  - 入力に`'*'`が入ると困る、という意見
    - `("h", "t")`のようなtupleを使う手。Javaだとrecordが良いか
  - ワイルドカードという感じ。`'?'`, `'_'`などのほうがもう少し慣例的？
- 素直にwordListの二重ループで隣接リストを作るのが自然、という考え
  - https://github.com/hayashi-ay/leetcode/pull/42
  - 遅くはなる。Pythonだと間に合わないこともあるらしい
  - これも変数ALPHABETSを持ち出さなくていい
- 振り返ると、自分のコードが英小文字に限っていたのは、あんまり好ましくなかった気がする？
- 双方向BFS、というものがあるらしい
  - ある点からの距離がdとなる点の個数は、dについて指数関数的に増える
  - beginWordとendWordの両方からBFSをすると、指数がd/2で済む
  - この問題だとそれはやりすぎでは、という話もある https://github.com/hayashi-ay/leetcode/pull/42/changes#r1515359437
- 語の隣接リストはより高速に計算できるらしい？
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1216123084889788486
  - 面白そうだけど、時間もかかりそうなので、頭の片隅に入れておきつつ放置
- 「頭から半分または尻尾から半分が一致しているはずなので、それでバケットを作ってバケット内でのみ比較すればいいというやりかたもありますね。」
  - これはよく理解できてない
- 4重ループだと、ネストが深いので関数化すべき、という感覚らしい
  - https://github.com/ryoooooory/LeetCode/pull/23
  - step 1のコードは3重ループで済んでいるが、キューを2つ持つBFSの書き方だと4重になる
- 添字を持たせる隣接リスト`Map<String, List<Integer>>`を使う手もある
  - https://github.com/goto-untrapped/Arai60/pull/57
  - 高速化が期待できる


## 清書
- `"h*t"` -> `["hot", "hat"]`のような辞書を作る方法で清書
  - いきなり思いつかなくてもいい方法みたいだけど、せっかくなので
- コードは、step 3と被るし長めなので`side_notes.md`に載せた
  - `side_notes.md`には他の方法のコードも載せておいた


# step 3
1回目：14:49、2回目：10:14、3回目：11:05、4回目：10:05

- 10分は切っていないが、このぐらいで良いかと思った

```java
public class Solution {
  // Returns 0 if there is no ladder
  public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Map<String, List<String>> patternToNeighbors = buildGraph(beginWord, wordList);

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
        addNeighbors(patternToNeighbors, word, nextWords, visited);
      }
      words = nextWords;
      ++length;
    }

    return 0;
  }

  String buildPattern(String word, int i) {
    char[] wordArray = word.toCharArray();
    wordArray[i] = '*';
    return String.valueOf(wordArray);
  }

  Map<String, List<String>> buildGraph(String beginWord, List<String> wordList) {
    Map<String, List<String>> patternToNeighbors = new HashMap<>();
    List<String> words = new ArrayList<>(wordList);
    words.add(beginWord);
    for (String word : words) {
      for (int i = 0; i < word.length(); ++i) {
        String pattern = buildPattern(word, i);
        patternToNeighbors.computeIfAbsent(pattern, _k -> new ArrayList<>())
            .add(word);
      }
    }
    return patternToNeighbors;
  }

  void addNeighbors(Map<String, List<String>> patternToNeighbors,
      String word, List<String> nextWords, Set<String> visited) {
    for (int i = 0; i < word.length(); ++i) {
      String pattern = buildPattern(word, i);
      for (String nextWord : patternToNeighbors.get(pattern)) {
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

- step 2から変わったところ：
  - `buildGraph`で`beginWord`を特別扱いしないことにした
    - 単に`wordList`（のコピー）に`add`した
  - `new String(char[])`の代わりに`String.valueOf(char[])`にしてみた
  - 変数名`adjacency`を`patternToNeighbors`に
  - 他は大体そのまま
    - （使わない変数`p`を`_k`にrename、`buildPattern(word, i)`を変数におく）

## step 3 補足
- `for (int i = 0; i < word.length(); ++i)`がくり返しているのでまとめてもいい、と思った
- そんなに自然かというとよくわからない

```java
// `for (int i = 0; i < word.length(); ++i)`をまとめたコード
  List<String> buildPatterns(String word) {
    List<String> patterns = new ArrayList<>();
    for (int i = 0; i < word.length(); ++i) {
      char[] wordArray = word.toCharArray();
      wordArray[i] = '*';
      patterns.add(String.valueOf(wordArray));
    }
    return patterns;
  }
```

