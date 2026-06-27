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
    - C++のイテレータに合わせてbegin/endが良い、という人も練習会で見かけたと思う
  - first/last（閉区間）
  - left/right, low/high
- 区間の長さが偶数の場合の「中央」の位置
  - 2種類ある
  - `middle = (start + end) / 2`は「右寄り」のほう
    - 気づいてなかった
  - 「左寄り」は`(start + end - 1) / 2`
- `sortedArrayToBST`というメソッド名
  - いくつか気になった点があるけど、目くじら立てるほどじゃないとも思った
  - BstとcamelCaseを徹底したい
  - 配列がsortedである必要はそんなにない（node.valを見てないので）
    - ただBSTでもなくなるので、ちょっと問題を述べづらくなる


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

他に思ったこと：
- 問題はheight-balanced（AVL）で十分だが、解答では "size-balanced" BSTを構成している
  - どのノードの左右の子も、サイズの差が1以内、ということのつもり
- 全然size-balancedじゃない木も作れるかが気になった
- 高さhのheight-balanced木で、サイズ最小のものは[Fibonacci tree](https://en.wikipedia.org/wiki/Fibonacci_sequence#:~:text=A%20Fibonacci%20tree%20is%20a%20binary%20tree%20whose)
- これを踏まえると、全然size-balancedじゃない木も作れる
  - 根の左の子がFibonacci、右の子がsize-balancedという木を考えると、サイズ比がとても大きくなる（`(2 / phi)^h`）
- 

