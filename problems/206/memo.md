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
- 再帰をスタックとループに書き直す人も[いる](https://github.com/goto-untrapped/Arai60/pull/27/changes#diff-0f2410aa19c752486fd39f11faebbe287a36b9dc9016dd3c6c1e0531259b409aR65)
  - 追記：よく見たら、スタックを使うというだけで、再帰の書き換えとは言っていなかった

ここまで40分ぐらい。

goto-untrapped氏のレビューを見る。
- 再帰について、組を返すという考えもある https://github.com/goto-untrapped/Arai60/pull/27#discussion_r1641596128
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

ここまで65分ぐらい。


# step 2.2 : 清書する


# step 3 : 3回書けるようになる

