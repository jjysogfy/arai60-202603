問題：https://leetcode.com/problems/valid-parentheses/description/


# step 1 : 解く
- 他の言語で解いたことはあった
- Javaを使うことにしたが、いろいろと知らないので、人のPRをさっと眺めたり、ドキュメントを少し読んだりした
  - https://github.com/goto-untrapped/Arai60/pull/10/changes
  - https://docs.oracle.com/javase/jp/8/docs/api/java/util/Map.html

```java
// Step 1
class Solution {
  public boolean isValid(String s) {
    Map<Character, Character> brackets = new HashMap<>();
    brackets.put(')', '(');
    brackets.put('}', '{');
    brackets.put(']', '[');

    var stack = new ArrayList<Character>();
    for (char c : s.toCharArray()) {
      if (!brackets.containsKey(c)) {
        stack.add(c);
        continue;
      }
      if (stack.isEmpty()) {
        return false;
      }
      char top = stack.remove(stack.size() - 1);
      if (brackets.get(c) != top) {
        return false;
      }
      continue;
    }
    return stack.isEmpty();
  }
}
```

この段階で考えたこと。
- 詰まりながら20分ぐらいかけて書いた
  - Javaの使い方もそうだが、アルゴリズムの部分も迷った
  - 全体をよく考えず書きはじめてしまったので、`brackets.containsKey(c)` の場合を先に処理していたところ、途中で `!brackets.containsKey(c)` のほうを先に処理するよう書き直した。
  - そのせいで無駄な `continue` が残っている……
- 変数名はあまりよくないかもしれない。とくに `stack` は適当すぎるかも
- Javaのこともよくわからない
  - `HashMap` の初期化はもっと簡単にならないかと思った。double-brace initializationというものを使えると言っている人がいる（ https://stackoverflow.com/questions/6833925/better-map-constructor ）。しかし、上のコードと違い anonymous inner class を作ると書いてある。
  - よく理解できていないが、とりあえずは上の単純なコードで十分そうだと感じたので、調べるのも一旦やめる
- `s.toCharArray()` してfor文を使っているが、このあたりの文字列の扱いはちょっと怖い感じがした（もちろん今回の設定では問題ないが）


# step 2 : コードを読む・清書する
ここまでで1時間ぐらいはかかっている。
Javaで解いている人のコードを主に見てみる。

- https://github.com/goto-untrapped/Arai60/pull/10/changes
- https://github.com/ryoooooory/LeetCode/pull/13/changes

- この問題は文脈自由文法で捉えられる、ということを連想するとのこと
- 便利なコレクションが他にもある
  - `Stack`, `ArrayDeque`
  - https://docs.oracle.com/javase/jp/8/docs/api/java/util/Stack.html
  - Stackクラスについては、「より完全で一貫性のある一連のLIFOスタック・オペレーションが、Dequeインタフェースとその実装によって提供されています。このクラスよりもそれらを優先的に使用するようにしてください。」だそうなので、ArrayDequeのが良いかもしれない
- 変数名も参考になる
  - `closeToOpen`
- `Map.of` というのが便利そう
  - https://docs.oracle.com/javase/jp/25/docs/api/java.base/java/util/Map.html#unmodifiable
  - これまで見ていたdocumentationは、よく見ると古いバージョンだった。そのせいで `Map.of` が書いていなかった。新しいバージョンを見たらあった。
  - https://www.oracle.com/jp/java/technologies/documentation.html 日本語訳のあるのはバージョン25が最新版らしい。
