問題（127. Word Ladder）：https://leetcode.com/problems/word-ladder/

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
    - IntStreamはあるがCharStreamはない
    - 追記：サロゲートペアを考えると、`char[]`より`int[]`にすべきで、それなら書けるかも
    - 追記：`int[] ALPHABETS = IntStream.rangeClosed('a', 'z').toArray()`
    - 追記：`alteredAt`も変更が必要。`c`の代わりに`Character.toString(c)`とするなど
    - 追記：ただ、実際にサロゲートペアがあると`ladderLength`自体は壊れそう
    - 追記：`String[] ALPHABETS`という手もあるか
  - static初期化ブロックを初めて使った
- WordAndNumまわりの変数名は迷った
  - はじめ`Queue<WordAndNum> wordsToVisit`を`Queue<String>`で書いていた
  - `wordsAndNumsToVisit`とかはかえってわかりづらいと思い変数名はそのままにした
- 実行時間について
  - ステップ数はだいたい、`wordList.length * ALPHABETS.length * word.length^2`
    - つまり`10^5 * 10^2`ぐらい
    - nextWordの計算で`word.length`がひとつ掛かると思う
  - Javaは1秒`10^6`から`10^7`程度
    - 追記：覚え間違い！ `10^7`から`10^8`程度。クロック周波数が`1G=10^9`オーダーで、そこから1、2桁落ちる
  - `10^7`ステップだと危ういなあと思いつつ書いていた
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

気づいたこと
- `"h*t"` -> `["hot", "hat"]`のような辞書を作る方法
  - 賢い。グラフが特殊な形なのを利用して、効率的に隣接リストを表現してる感じ
  - step 1のコードでいう`alteredAt(word, i, c)`を`c`ごとに計算しなくてよくなる
  - 計算量が`word.length`倍改善する
  - 変数ALPHABETSを持ち出さなくていいのも良い
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

