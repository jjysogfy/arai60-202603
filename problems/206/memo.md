# 206. Reverse Linked List
問題：https://leetcode.com/problems/reverse-linked-list/description/


# 取り組みかた・このファイルの構成
基本的に[「標準的な進め方」](https://docs.google.com/document/d/1bjbOSs-Ac0G_cjVzJ2Qd8URoU_0BNirZ8utS3CUAeLE/edit?tab=t.0#heading=h.6ana9osx0wrd)にならう。

- step 1 で、まず解く。step 2.1 で、人のコードやコメント集を読む。それをもとに、step 2.2 では清書する。step 3 では、3回続けて解けるように練習する。


# step 1 : 解く
解くときに考えたこと：
- 他の言語では解いたことがある。書いてみる

```java
// step 1
class Solution {
  // `head` を書き換えるコード
  public ListNode reverseList(ListNode head) {
    var source_node = head;
    ListNode target_node = null;
    while (source_node != null) {
      var node = source_node;
      source_node = source_node.next;
      node.next = target_node;

      target_node = node;
    }
    return target_node;
  }
}
```

解いてから考えたこと：
- 5分ぐらいで書いて、3分ぐらい見直した。
  - はじめ、`source_node = source_node.next;` と `node.next = target_node;` の順番を逆にしてしまった。
  - 小さいサンプルで、頭のなかで行ごとに動作させたら、なんとか気づいた
- 変数名は、他にも良いのがありそうだが、（今のところ）許容範囲と感じる
- 空行の入れ方はおかしいかもしれない。書いていて区切りたくなったところに入れた
  - `source_node` の先頭ノードを `source_node` から外して、`target_node` に付けかえる、をひとまとまりと見た
- `node` 変数はいらないかもしれないと思ったが、わからないし間違えそうなのでこう書いた
- 引数 `head` を変更すべきかどうか
- 追記：変数名をcamelCaseにすべき

解き始めてからここまで、15分ちょっと。


# step 2.1 : 人のコードを読む
まず、Javaで完走しているらしい人のコードを見る。
- https://github.com/goto-untrapped/Arai60/pull/27
- https://github.com/ryoooooory/LeetCode/pull/14

思ったこと：
- 再帰を使う人もいる。たしかに、LeetCodeの「Follow up」にもそう書いてある
  - これもやったことがある
  - `head.next` を普通にreverseすると、`ListNode` の末尾に付け加える必要が出てきてやりづらい。そこで、「末尾に付け加えるリスト」も引数に入れたヘルパー関数を作れば[よい](https://github.com/goto-untrapped/Arai60/pull/27/changes#diff-0f2410aa19c752486fd39f11faebbe287a36b9dc9016dd3c6c1e0531259b409aR55)
  - 参照をうまく使って（という言い方でいいのか？）処理する人も[いる](https://github.com/goto-untrapped/Arai60/pull/27/changes#diff-0f2410aa19c752486fd39f11faebbe287a36b9dc9016dd3c6c1e0531259b409aR40)。`head.next` をreverseすると、その末尾は `head.next` と表せる
    - `var newTail = head.next` とか、名前があったほうが見やすい気がする。読むのにちょっと悩んだ
    - 追記：元の先頭は、reverseすると末尾になる、ということなので、ある意味当たり前ではあるものの……。
- スタックで書く方法も[ある](https://github.com/goto-untrapped/Arai60/pull/27/changes#diff-0f2410aa19c752486fd39f11faebbe287a36b9dc9016dd3c6c1e0531259b409aR65)
  - 前から順にスタックに積むと、積まれ方はすでに逆順になっている。スタックから取り出しながら、`.next` を書き換える。
  - 「後ろからループ」できるならreverseは簡単なはずで、そのために一度リストをスタックに取り出す、という感じか

goto-untrapped氏のレビューを見る。
- 再帰について、組を返すという考えもある https://github.com/goto-untrapped/Arai60/pull/27#discussion_r1641596128
  - （上の「参照をうまく使」う方法で、再帰呼び出しで返ってくるリストの末尾について、`head.next` と参照するかわりに返り値に含めてしまう、という感じ）
- Javaでどうやって組を返すのか気になった
  - ChatGPTに訊いたら、クラスを作るか `record` を使うとのこと。
  - `record` はJava 16からの機能だそうだ（[「This JEP proposes to finalize the feature in JDK 16」](https://openjdk.org/jeps/395)）。
  - `record` の使い方については、もっと調べたほうがいいかもしれない。しかし、以下のコード例を見たりして、雰囲気がわかって満足したので次に進む
  - （あと、組を返すのは、このようにJavaだと少し面倒だとわかり、今回の問題だと進んで選びたくはない感じもする）

```java
// ChatGPTによるコード
// 複数の値を `record` として返す関数の例
record Result(int sum, int product) {}

public static Result calculate(int a, int b) {
    return new Result(a + b, a * b);
}
```

ryoooooory氏の[コード](https://github.com/ryoooooory/LeetCode/pull/14)も見る。
- FirstSolution.java, L4: 自分が変数名 `target_node` としたものを `prev` と呼んでいる。気持ちがよくわかっていないが、処理済みだから、ということ？
  - 少しわかってきた。`prev = current` としてから次のループに行くので、確かに `prev` という感じがする。では、なぜ初めわからなかったかというと、自分としては、変数の指すものとして、各ノードというよりリスト全体に意識が向いていたからだと思った。
- FirstSolution.java, L15: スタックを使う方法。さっきと少し違って混乱した。空リストについて、先程は番兵で処理していたが、今回ははじめに `return null` して処理している。
- [ifのdanglingについて](https://github.com/ryoooooory/LeetCode/pull/14#discussion_r1657558481)
  - どうでもいいことだが、「{}を省略すること」の「ぶらさがり」感がピンときていない
  - https://en.wikipedia.org/wiki/Dangling_else パース結果が一つに決まらなくなるので、どのifに対するelseなのか「ぶらぶらする」、ぐらい？
- [else ifを避けるとき](https://github.com/ryoooooory/LeetCode/pull/14#discussion_r1653787365)
  - 気にしていなかった。そういう考えもあるのか

ここまで2時間ちょっとかかった。

コメント集も軽く見ておく。
- `prev` という変数名はパズルになってしまっているという意見が[ある](https://discord.com/channels/1084280443945353267/1307605446538039337/1315556530837262366)。自分も読んでいて迷ったのだった
- スタックを使う方法で、番兵を使うかなどのバリエーションがあること https://discord.com/channels/1084280443945353267/1318201011722129459/1338003562143420547
  - 上述のryoooooory氏のコードは、「ループの外に出す」方法か
  - 追記：よくわかっていないことに気づいた。「分岐をする」がどういうことかわからない
- [ひっくり返し方が3種類あるという話](https://discord.com/channels/1084280443945353267/1350090869390311494/1355838883325022278)
  - 読んでしばらく考えていた。よくわからなくなってきた。あとでもう少し考える
  - 追記：やっぱりよくわからなかった。1が「リンク」に注目しているということがわからない。「色々覚えろというわけではなく」だそうなので、とりあえず気にしない。


# step 2.2 : 清書する
```java
// step 2
class Solution {
  public ListNode reverseList(ListNode head) {
    ListNode rest = head;
    ListNode reversed = null;

    while (rest != null) {
      var newHead = rest;
      rest = rest.next;
      newHead.next = reversed;
      reversed = newHead;
    }
    return reversed;
  }
}
```

このコードへのコメント：
- いろいろな方法があったが、結局初めに書いた方法で書くことにした
- 変数名を変えた程度

ここまで、3時間半ぐらい。


# step 3 : 3回書けるようになる
```java
class Solution {
  public ListNode reverseList(ListNode head) {
    ListNode reverse = null;
    ListNode rest = head;
    while (rest != null) {
      var newHead = rest;
      rest = rest.next;
      newHead.next = reverse;
      reverse = newHead;
    }
    return reverse;
  }
}
```

- コードは基本的に同じ
  - 変数名は変えようか悩んだが、ほぼそのままに
- 1、2分で書けるようになった
- 途中から、休憩を挟んだり他のことをやったりしていたので、はっきりした時間がわからないが、5、6時間ぐらいはかかったような気がする
  - 今回は、はじめから解けてはいたので、それを踏まえると時間をかけすぎだった気がする
  - コメント集を読み始めたあたりから、いつの間にか時間が経ってしまった
  - かけた時間のわりには、得られたものが少ない気がしている




## 追記：他の方法について
- 再帰を使う方法や、スタックを使う方法も、試しに書いてはみた。
- 元々はこのファイルに残しておくつもりはなかったが、やっぱり書いておいたほうが良いような気がしてきたので。

```java
// ためしに再帰を使う
class Solution {
  public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) {
      return head;
    }

    var reversed = this.reverseList(head.next);
    var reversedTail = head.next;
    head.next = null;
    reversedTail.next = head;
    return reversed;
  }
}
```

```java
// ためしにスタックを使う
class Solution {
  public ListNode reverseList(ListNode head) {
    if (head == null) {
      return head;
    }

    var stack = new ArrayDeque<ListNode>();
    var node = head;
    while (node != null) {
      stack.push(node);
      node = node.next;
    }

    var sentinel = new ListNode();
    var previous = sentinel;
    ListNode top = null;
    while (!stack.isEmpty()) {
      top = stack.pop();
      // 追記：topへのリンクを張る
      previous.next = top;
      previous = top;
    }
    // 追記：末尾からのリンクも正しくする
    top.next = null;
    return sentinel.next;
  }
}
```

- 後者は何回かエラーを出しながら書いた
- 変数名、特に `top` はよろしくない
  - はじめは `var top = stack.pop();` と書いていたが、それでは `top.next = null;` が書けないので変更したせいでこうなった
  - `node` をshadowしたい気分もあるが、Javaではできない

いろいろとよくわからなくなってきたが、さすがに時間をかけすぎているので、このあたりでとりあえずやめる。
