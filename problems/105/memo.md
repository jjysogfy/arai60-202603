問題：105. Construct Binary Tree from Preorder and Inorder Traversal
https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

# step 1

- 20分ぐらい
  - 問題は前に見て考えていた
  - スマホで書いていたので、本当ならもう少し短かったかも

```java
class Solution {
  public TreeNode buildTree(int[] preorder, int[] inorder) {
    return buildTreeFromLists(
        Arrays.stream(preorder).boxed().toList(),
        Arrays.stream(inorder).boxed().toList());
  }
  
  TreeNode buildTreeFromLists(List<Integer> preorder, List<Integer> inorder) {
    if (preorder.isEmpty()) return null;
    int rootVal = preorder.get(0);
    int numLeftNodes = inorder.indexOf(rootVal);
    TreeNode left = buildTreeFromLists(
        preorder.subList(1, numLeftNodes + 1),
        inorder.subList(0, numLeftNodes));
    TreeNode right = buildTreeFromLists(
        preorder.subList(numLeftNodes + 1, preorder.size()),
        inorder.subList(numLeftNodes + 1, inorder.size()));
    return new TreeNode(rootVal, left, right);
  }
}
```

- 方針
  - preorderの先頭がrootになっている
  - もっといえばこんな感じ：
  - preorder: `root [ left ] [ right ]`
  - inorder: `[ left ] root [ right ]`
  - `inorder.indexOf(root)`の前後で分割すればいい


# step 2
Javaでたくさん解いている方。
- https://github.com/ryoooooory/LeetCode/pull/32/changes
- https://github.com/goto-untrapped/Arai60/pull/53/changes

思ったこと：
- goto-untrapped氏のstep 1のコードはちょっと大変
  - 変数 rootIndex が難しい
    - インスタンス変数になっていると変更を追いづらい気がする。Javaだとちょっと書き換えづらいけど（参照が取れないから）
    - `rootIndex--`もわかりづらいと感じる。基本的に進めるだけだが、時々つじつま合わせのために戻す、みたいに感じる
  - https://github.com/goto-untrapped/Arai60/blob/10206faf2ef26abf2cf23f23599753c08cbcba3e/105.%20Construct%20Binary%20Tree%20from%20Preorder%20and%20Inorder%20Traversal/ConstructBinaryTreeFromPreorderAndInorderTraversalStep1.java
- 再帰呼び出しで、配列をコピーしても間に合う
  - どのみちList.indexOfしているので、時間計算量は同じ
  - （subListはビューを作るだけ）
  - Solution2_1 https://github.com/goto-untrapped/Arai60/blob/10206faf2ef26abf2cf23f23599753c08cbcba3e/105.%20Construct%20Binary%20Tree%20from%20Preorder%20and%20Inorder%20Traversal/ConstructBinaryTreeFromPreorderAndInorderTraversalStep2.java

