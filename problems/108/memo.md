問題（108. Convert Sorted Array to Binary Search Tree）：https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/

# step 1
```java
class Solution {
  public TreeNode sortedArrayToBST(int[] nums) {
    return sortedArrayToBstHelper(nums, 0, nums.length);
  }

  TreeNode sortedArrayToBstHelper(int[] nums, int start, int end) {
    if (end - start == 0) {
      return null;
    }
    if (end - start == 1) {
      return new TreeNode(nums[start]);
    }
    int middle = (start + end) / 2;
    TreeNode leftNode = sortedArrayToBstHelper(nums, start, middle);
    TreeNode rightNode = sortedArrayToBstHelper(nums, middle + 1, end);
    return new TreeNode(nums[middle], leftNode, rightNode);
  }
}
```

コードについて思ったこと：
- 書いてから時間が空いたので、そのときのことを忘れてしまった……
- `if (end - start == 1)`の部分は不要だ


# step 1.2：他に思ったこと
- 区間の変数名は何があるか？
  - start/end, begin/end（半開区間）
    - 半開区間なら、C++のイテレータに合わせてbegin/endが良い、と練習会で見かけたと思う
  - first/last（閉区間）
  - left/right
  - low/high
- 区間の長さが偶数の場合の「中央」の位置
  - 2箇所ある
  - `middle = (start + end) / 2`は「右寄り」のほう
    - 気づいてなかった。切り下げだが「右寄り」
  - 「左寄り」は`(start + end - 1) / 2`
- `sortedArrayToBST`というメソッド名
  - いくつか気になった点があるけど、目くじら立てるほどじゃないとも思った
  - BSTではなくBstにしてcamelCaseを徹底したい
  - 配列がsortedである必要はそんなにない（node.valを見てないので）
    - ただBSTでもなくなるので、ちょっと問題を述べづらくなる
  - 動詞始まりじゃないので、微妙にメソッドらしくないかも


# step 2
Javaでたくさん解いている人のコードを読む
- https://github.com/ryoooooory/LeetCode/pull/27/changes
- https://github.com/goto-untrapped/Arai60/pull/48/changes
  - Solution2_4から2_7までは読んでいない

コードを読んで思ったこと：
- 「参照の値渡し」という言葉
  - https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value
  - 「By those definitions, Java is always pass-by-value.」
  - 英語だと、「参照の値渡し」に相当する用語は見つけられなかった
  - 用語はともかく、引数に代入しても呼び出し元に影響ないことがわかっていればいいかと思っている（？）
- 再帰をループになおしている人もいる

```java
// step 2 その1 スタックとループで書く
class Solution {
  private class TreeNodeRef implements Consumer<TreeNode> {
    TreeNode node;

    @Override
    public void accept(TreeNode node) {
      this.node = node;
    }
  }

  public TreeNode sortedArrayToBST(int[] nums) {
    var nodeRef = new TreeNodeRef();
    Deque<StackFrame> stack = new ArrayDeque<>();
    stack.push(new StackFrame(0, nums.length, node -> nodeRef.node = node));
    while (!stack.isEmpty()) {
      StackFrame frame = stack.pop();
      int begin = frame.begin;
      int end = frame.end;
      Consumer<TreeNode> writer = frame.writer;
      if (begin >= end) {
        continue;
      }
      int middle = (begin + end) / 2;
      TreeNode node = new TreeNode(nums[middle]);
      writer.accept(node);
      stack.push(new StackFrame(begin, middle, leftNode -> node.left = leftNode));
      stack.push(new StackFrame(middle + 1, end, rightNode -> node.right = rightNode));
    }
    return nodeRef.node;
  }

  record StackFrame(int begin, int end, Consumer<TreeNode> writer) {
  }
}
```

書いて思ったこと：
- Javaでは`node.left`などの参照を取れないので、コールバックを使ってみた
  - あまり見かけない
  - 前に書いたことはある https://github.com/jjysogfy/arai60-202603/pull/12/changes#diff-5793c905e594ca7f475c5d14533717def859f8cf7b22a7ffd23cef21b9029107R111
- スタックに何を積むか、ちょっと迷ってしまった


## step 2.2：他に思ったこと（黄金比分割）
この節は少し話がそれているかもしれません。
- 問題はheight-balanced（AVL）で十分だが、解答では "size-balanced" BSTを構成している
  - どのノードの左右の子も、サイズの差が1以内、ということのつもり
- 高さhのheight-balanced木で、サイズ最小のものは[Fibonacci tree](https://en.wikipedia.org/wiki/Fibonacci_sequence#:~:text=A%20Fibonacci%20tree%20is%20a%20binary%20tree%20whose)
  - サイズは、Fibonacci数で書かれて`O(phi^h)`
- より "全然size-balancedじゃない" height-balanced木の例：
  - 根の左の子がFibonacci、右の子がsize-balancedという木を考えると、サイズ比がとても大きくなる（約`(2 / phi)^h`）

- Fibonacci treeを踏まえると、step 1のアルゴリズムは等分ではなく「黄金比分割」でも動作するのでは？
  - たぶん正しい。ChatGPT 5.5に証明してもらった
  - おおまかなアイデア：
    - Fibonacci treeのサイズ比を計算すると、黄金比になる（modulo 切り下げ）
    - 逆にいえば、黄金比分割アルゴリズムをやるとFibonacci tree（にノードを足したもの）ができる

```java
// step 2 その2 黄金比分割
class Solution {
  public TreeNode sortedArrayToBST(int[] nums) {
    return sortedArrayToBstHelper(nums, 0, nums.length);
  }

  private TreeNode sortedArrayToBstHelper(int[] nums, int begin, int end) {
    if (begin >= end) {
      return null;
    }
    double phi = (1 + Math.sqrt(5)) / 2;
    int middle = (int) ((phi * begin + end) / (phi + 1));
    TreeNode left = sortedArrayToBstHelper(nums, begin, middle);
    TreeNode right = sortedArrayToBstHelper(nums, middle + 1, end);
    return new TreeNode(nums[middle], left, right);
  }
}
```

書いて思ったこと：
- 浮動小数点数まわりは不安
  - sqrtの正しさは保証されている https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/lang/Math.html#sqrt(double)
  - 制約は`nums.length <= 10^4`だった
  - それなら大丈夫そう？（根拠は持ってない）


## step 2.3：清書
```java
class Solution {
  public TreeNode sortedArrayToBST(int[] nums) {
    return sortedArrayToBstHelper(nums, 0, nums.length);
  }

  private TreeNode sortedArrayToBstHelper(int[] nums, int begin, int end) {
    if (begin >= end) {
      return null;
    }
    int middle = (begin + end) / 2;
    TreeNode left = sortedArrayToBstHelper(nums, begin, middle);
    TreeNode right = sortedArrayToBstHelper(nums, middle + 1, end);
    return new TreeNode(nums[middle], left, right);
  }
}
```


# step 3
1回目（ミスあり）：4:10、2回目：2:20、3回目（ミスあり）：2:30
4回目：2:32、5回目：2:41、6回目：2:45

ミス：
- sortedArrayToBstHelperで、beginとendの代わりに、subListを使おうとして（続く）
- 、ライブラリの使い方をいろいろ間違えた
  - 配列をListに変換忘れ、添字アクセスを`.get`にし忘れた、など
- `sortedArrayToBstHelper(nums, 0, middle)`としてしまった（0 -> begin）

```java
// step 3
class Solution {
  public TreeNode sortedArrayToBST(int[] nums) {
    return sortedArrayToBstHelper(nums, 0, nums.length);
  }

  private TreeNode sortedArrayToBstHelper(int[] nums, int begin, int end) {
    if (end - begin <= 0) {
      return null;
    }
    int middle = (begin + end) / 2;
    TreeNode leftNode = sortedArrayToBstHelper(nums, begin, middle);
    TreeNode rightNode = sortedArrayToBstHelper(nums, middle + 1, end);
    return new TreeNode(nums[middle], leftNode, rightNode);
  }
}
```
