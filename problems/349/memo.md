問題：[349. Intersection of Two Arrays](https://leetcode.com/problems/intersection-of-two-arrays/)


# ファイルの構成
- memo.md: 主要部分
  - このファイルを見てくだされば大丈夫です
- side_notes.md: 脇にそれた調べごとなど


# step 1
- いろいろやり方はあるか
  - `nums1`を`Set`に変換し、`nums2`の各要素がそこに現れるか調べる
  - 2回出ると困る
  - 重複を除くため、`nums2`も`Set`にする

```java
class Solution {
  public int[] intersection(int[] nums1, int[] nums2) {
     Set<Integer> uniqueNums1 = new HashSet<>();
     for (int num : nums1) {
      uniqueNums1.add(num);
     }
     Set<Integer> uniqueNums2 = new HashSet<>();
     for (int num : nums2) {
      uniqueNums2.add(num);
     }
     List<Integer> intersection = new ArrayList<>();

     for (int num : uniqueNums2) {
      if (uniqueNums1.contains(num)) {
        intersection.add(num);
      }
     }

     var unboxedIntersection = new int[intersection.size()];
     for (int i = 0; i < intersection.size(); ++i) {
      unboxedIntersection[i] = intersection.get(i);
     }
     return unboxedIntersection;
  }
}
```

- `Set`への変換を、適当に`Set.of(nums1)`とか書いて間違えた
  - 何か良いやり方があるだろうと、ChatGPTに訊いたりGoogle検索したり
  - `Stream`などと言われた
    - これは知らなきゃいけないと思ったが、調べると時間がかかるので、一旦飛ばす
    - （そもそも、`Stream`を使う方法も、多少は煩雑に見える……）
  - すべて安易な方法で書いた
- `List<Integer>`から`int[]`への変換も、安直な方法で
  - 変数名が`intersection`、`unboxedIntersection`などと長いこともあり、煩雑

