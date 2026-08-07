問題：https://leetcode.com/problems/valid-parentheses/description/


# step 1 : 解く
- 他の言語で解いたことはあった
- Javaを使うことにしたが、いろいろと知らないので、人のPRをさっと眺めたり、ドキュメントを少し読んだりした
  - https://github.com/goto-untrapped/Arai60/pull/10/changes
  - https://docs.oracle.com/javase/jp/8/docs/api/java/util/Map.html

```java
// step 1
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
  - 全体をよく考えず書きはじめてしまってから、途中で書き換えたせいで、無駄な `continue` が残っている……
  - （解き始める前に、どの問題を解くか選んだり、調べごとをしたりもしていたので、そういう時間を含めればもっとたくさん時間がかかっている）
- 変数名はあまりよくないかもしれない。とくに `stack` は適当すぎるかも
- Javaのこともよくわからない
  - `HashMap` の初期化はもっと簡単にならないかと思った。double-brace initializationというものを使えると言っている人がいる（ https://stackoverflow.com/questions/6833925/better-map-constructor ）。しかし、上のコードと違い anonymous inner class を作ると書いてある。
  - よく理解できていないが、とりあえずは上の単純なコードで十分そうだと感じたので、調べるのも一旦やめる
- `s.toCharArray()` してfor文を使っているが、Unicode文字列の扱いはちょっと怖い感じがした（もちろん今回の設定では問題ないが）


# step 2.1 : コードを読む
解き始めてから1時間ぐらいはかかっている。調べごとをしたり、このファイルを書いたりでいつの間にか時間がかかってしまった。

Javaで解いている人のコードを主に見てみる。

- https://github.com/goto-untrapped/Arai60/pull/10/changes
- https://github.com/ryoooooory/LeetCode/pull/13/changes

- この問題を見ると、文脈自由文法を連想するとのこと
- 便利なコレクションが他にもある
  - `Stack`, `ArrayDeque`
  - https://docs.oracle.com/javase/jp/8/docs/api/java/util/Stack.html
  - Stackクラスについては、「より完全で一貫性のある一連のLIFOスタック・オペレーションが、Dequeインタフェースとその実装によって提供されています。このクラスよりもそれらを優先的に使用するようにしてください。」だそうなので、ArrayDequeのが良いかもしれない
- 変数名も参考になる
  - `closeToOpen`
- `Map.of` というのが便利そう
  - https://docs.oracle.com/javase/jp/25/docs/api/java.base/java/util/Map.html#unmodifiable
  - （これまで見ていたdocumentationは、よく見ると古いバージョンだった。そのせいで `Map.of` が書いていなかった。バージョン9で追加された機能だった。新しいバージョンを見たらあった。なお今年のバージョン26は[出たばかり](https://blogs.oracle.com/java/the-arrival-of-java-26)のせいか日本語訳がなく、一つ前の[25](https://docs.oracle.com/javase/jp/25/docs/api/index.html)を参照した。）
- `openToClose` を使う人もいる（`closeToOpen` ではなく）
  - たとえば https://github.com/ryoooooory/LeetCode/pull/13/changes#diff-f9961ace8ead467d6aeebc53b241da26a31ee32d345d85fcd0c2c6b7b3c1c609
  - かっこ以外の文字が来たときの振る舞いも、自分のコードとは違う
  - かっこ以外の文字を読んだらすぐ `return false` している。自分のコードでは、まずはかっこ以外の文字もスタックに積んで実行を続け、あとから `return false` する
- `switch` 文を使った人もいる（特に step 1）
- Javaでの文字列について書いている人もいる
  - https://github.com/HitoshiKoba/Arai60-public/pull/2/changes/BASE..8c5b480e383a45d3c8e4e750eb214d7adad253ef#diff-7f1d09185c5788a4021637266180cd9c8b3a1cf8fda54c5f8a53331a71c45861
  - `char[]` はパスワード処理に使われると書いてある
  - 確かに、ドキュメントを見ると `char[]` が使われている https://docs.oracle.com/javase/jp/25/docs/api/java.base/java/net/PasswordAuthentication.html
- スタックに閉じかっこ `)` を積むという考えもある
  - https://github.com/philip82148/leetcode-swejp/pull/11#discussion_r2057085868
  - `c` が閉じかっこのときの処理で考えることが減り、見やすい気がする
  - 悪い点はなにか？ （シビアな）計算時間の問題が指摘されている。一つの開きかっこに「閉じかっこ」が複数種類がある文法（そんなものがあるとして）だと、この方法ではやりづらいとは思った
- `openToClose` をconstにしている人もいる


コメント集も見る。
- 番兵を使うと、`stack.isEmpty()` が不要に https://github.com/SanakoMeine/leetcode/pull/7
- かっこ以外の文字をどうするか https://discord.com/channels/1084280443945353267/1225849404037009609/1231648833914802267
  - 自分のコードは、かっこ以外の文字が来たとき、`false` になる
  - しかし、確かに、かっこ以外の文字は読み飛ばすほうが便利なことが多いかも
- 例外を投げることについて https://github.com/mura0086/arai60/pull/11
  - 誰かがコードを書き換えたときを想像している。なるほど。経験がないのでピンとは来ないけど
- 進め方について https://github.com/HitoshiKoba/Arai60-public/pull/2

ここまで3時間ぐらいかかっていると思う。先に進む。


# step 2.2 : 清書する
```java
// step 2
import java.util.Map;
import java.util.ArrayDeque;

class Solution {
  public boolean isValid(String s) {
    final Map<Character, Character> openToClose = Map.of('(', ')', '{', '}', '[', ']');
    var closeBracketStack = new ArrayDeque<Character>();

    for (char c : s.toCharArray()) {
      if (openToClose.containsKey(c)) {
        var closeBracket = openToClose.get(c);
        closeBracketStack.push(closeBracket);
        continue;
      }

      if (closeBracketStack.isEmpty()) {
        return false;
      }
      char top = closeBracketStack.pop();
      if (top != c) {
        return false;
      }
    }
    return closeBracketStack.isEmpty();
  }
}
```

変更点
- LeetCodeでは不要のようだが、`import`文を書いておいた
- `openToClose` （`closeToOpen` ではなく）を使う
- （変更しなかったところ：かっこ以外の文字がある場合は `return false`）


# step 3 : 3回書けるようになる
```java
// step 3
class Solution {
  public boolean isValid(String s) {
    final var openToClose = Map.of('(', ')', '{', '}', '[', ']');
    var closeBracketStack = new ArrayDeque<Character>();

    for (char c : s.toCharArray()) {
      if (openToClose.containsKey(c)) {
        closeBracketStack.push(openToClose.get(c));
        continue;
      }

      if (closeBracketStack.isEmpty()) {
        return false;
      }
      var top = closeBracketStack.pop();
      if (top != c) {
        return false;
      }
    }
    return closeBracketStack.isEmpty();
  }
}
```

- 5回ぐらい書いた。1回2分ぐらいで書けるようになった。
- ちゃんと計っていないが、全体で4、5時間かかっている気がする。
  - このファイル（`20-Valid-Parentheses.md`）を書くのにも時間がかかっていると思う。余計なことを書いていそう？
- 見直していて思ったが `var top = closeBracketStack.pop();` の `var` は危うい。`Character` どうしの `==` は意図したものにならないので



# step 4 : レビューを反映
step 3 をもとに、レビューを反映したコードに書き換え。

```java
class Solution {
  public boolean isValid(String s) {
    final Map<Character, Character> openToClose = Map.of(
      '(', ')',
      '{', '}',
      '[', ']'
    );
    ArrayDeque<Character> closeBrackets = new ArrayDeque<>();

    for (char c : s.toCharArray()) {
      if (openToClose.containsKey(c)) {
        closeBrackets.push(openToClose.get(c));
        continue;
      }
      if (closeBrackets.isEmpty()) {
        return false;
      }
      char expected = closeBrackets.pop();
      if (c != expected) {
        return false;
      }
    }

    return closeBrackets.isEmpty();
  }
}
```

## 補足
- hayashi-ay氏のコメントで「autoboxingや`valueOf`経由で作られた`Character`」とあった
  - ここ、どういうことだろうと漠然と気になっていた
    - 訊いたら良かったと思うが、こういう疑問をちゃんと形にするが苦手な気がする
  - `new`ではないとかいうことか
  - そもそも`new`がdeprecated（ https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/lang/Character.html#%3Cinit%3E(char) ）で、それはまさにそういう理由のようだ

