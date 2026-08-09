問題：300. Longest Increasing Subsequence
https://leetcode.com/problems/longest-increasing-subsequence/

# step 1
```java
class Solution {
  public int lengthOfLIS(int[] nums) {
    // lengths[i] = length of the LIS ending with nums[i]
    int[] lengths = new int[nums.length];
    for (int i = 0; i < nums.length; ++i) {
      lengths[i] = 1;
      for (int k = 0; k < i; ++k) {
        if (nums[k] >= nums[i]) {
          continue;
        }
        lengths[i] = Math.max(lengths[i], lengths[k] + 1);
      }
    }
    return Arrays.stream(lengths).max().orElse(0);
  }
}
```

- 問題文を見てから一週間ぐらいして書いた
  - 前にも見たことがある
- かかった時間：6:01
- ミス：
  - `IntStream.max`がOptionalを返すのを忘れていた
- 方針：
  - （DPと書いてあるし、この問題を見たこともあるので、念頭に置いて考える）
  - `nums[nums.length - 1]`で場合分けしてみる：
    - LISに含まれるか、含まれないか
    - どちらの場合でも、`[0, nums.length - 2]`を見てLISの長さを計算できる
  - 時間計算量`O(N^2)`、`N <= 2500`で`25^2=625`なので、実行時間は60ms--600ms程度と見積もる
    - 41msだった。合っている
- ソートと二分探索とかで高速化できたはず、となんとなく覚えている
  - 高速な解法を思い出せた気でいたが、間違えていた
    - 思い出した気でいたときのメモは`side_notes.md`に残しておく
- （ところで、どんな`nums`のせいでこういうアルゴリズムが必要になるのか、わかっていないと気づく）


# step 2
Javaでたくさん解いている方。
- https://github.com/goto-untrapped/Arai60/pull/18
- https://github.com/ryoooooory/LeetCode/pull/34


## step 2.1 `O(N log N)`の方法
- https://github.com/ryoooooory/LeetCode/pull/34#discussion_r2133973501

`O(N log N)`の方法について、読んだことをもとに、自分でも考えてみる。

step 1のコードを、二分探索を使って高速化する。いろいろと悩んで、次のコードを書いてみた。

```java
// step 2.1 その1
class Solution {
  public int lengthOfLIS(int[] nums) {
    // numToLength.lowerEntry(num).getValue() == num以下に収まるLISの長さ
    NavigableMap<Integer, Integer> numToLength = new TreeMap<>();

    for (int num : nums) {
      Map.Entry<Integer, Integer> lower = numToLength.lowerEntry(num);
      int newLength = lower == null ? 1 : lower.getValue() + 1;

      NavigableMap<Integer, Integer> tail = numToLength.tailMap(num, true);
      Iterator<Integer> iter = tail.values().iterator();
      while (iter.hasNext()) {
        int length = iter.next();
        if (length > newLength) {
          break;
        }
        iter.remove();
      }

      numToLength.put(num, newLength);
    }

    return numToLength
        .sequencedValues()
        .getLast();
  }
}
```

考えたことのメモ：
- `lengths[i + 1]`を高速に計算するにはどうするか
  - `lengths`はstep 1の記号（`lengths[i]`は、`nums[i]`で終わるLISの長さ）
- `nums[i + 1]`の具体的な値は重要ではなく、`nums[0], .., nums[i]`との関係だけが大事
- とくに、`nums[i + 1]`より真に小さいnumsのうち一番大きいものが大事（`nums[k]`とおく）
- 次のようなTreeMap `numToLength`を考える
  - `numToLength.get(nums[k])`は、`nums[k]`以下の数からなるLISの長さ
- 例：
  - numsが100, 400, 200, 300とする
  - numToLengthは次のように遷移する
    - `{100: 1}` ->
    - `{100: 1, 400: 2}` ->
    - `{100: 1, 200: 2}` ->
    - `{100: 1, 200: 2, 300: 3}`
  - 安直には次のようにやりそうになってしまうが、それは不都合
    - `{100: 1}` ->
    - `{100: 1, 400: 2}` ->
    - `{100: 1, 200: 2, 400: 2}` ->
    - `{100: 1, 200: 2, 300: 3, 400: 2}`
    - たとえば次に500が来ると、`400: 2`でなく`300: 3`まで見ないと正しい結果を計算できなくなる
      - （`400: 3`と更新しておいたらどうかとも思った）
      - （しかし、これだと無駄な計算がかなり増えてしまう。やっぱり`400`をremoveするのがいい。）

書いてから思ったこと：
- 1時間以上かけて書いた。混乱した
- 実際には、`iter`でループを回す必要はない
  - ループは高々1回しか回らない
- ループを書くとして、Iteratorはあまり読みやすくない気がするが **他の方法を思いつかない**
- numToLengthのvalueは1つずつ増える
  - ここまでくると、`{1: 100, 2: 200, 3: 300}`のように逆向きの対応をさせればいい、と思いつくかもしれない
- 変数名newLengthはあまり **わかりやすくない** 気がする
  - 単にlengthでいいかな、でも+1したことは変数名に入れておきたい、ぐらいの気持ち

Iteratorのループは高々1回しか回らないことを使った書き換えをしておく。

```java
// step 2.1 その1' （「その1」の書き換え）
class Solution {
  public int lengthOfLIS(int[] nums) {
    // numToLength.lowerEntry(num).getValue() == num以下に収まるLISの長さ
    NavigableMap<Integer, Integer> numToLength = new TreeMap<>();

    for (int num : nums) {
      Map.Entry<Integer, Integer> lower = numToLength.lowerEntry(num);
      int newLength = lower == null ? 1 : lower.getValue() + 1;

      Integer higher = numToLength.ceilingKey(num);
      if (higher != null) {
        numToLength.remove(higher);
      }

      numToLength.put(num, newLength);
    }

    return numToLength
        .sequencedValues()
        .getLast();
  }
}
// step 2.1 その1' 終わり
```

「逆向きの対応」を使ってみる。つまり、LISの末尾の数のうち最小のもの、を管理する方法で書く。

```java
// step 2.1 その2
class Solution {
  public int lengthOfLIS(int[] nums) {
    // lengthToLastNum.get(length - 1): 長さlengthのLISの末尾のうち、最小のもの
    List<Integer> lengthToLastNum = new ArrayList<>();

    for (int num : nums) {
      int searchResult = Collections.binarySearch(lengthToLastNum, num);
      int insertionPoint = searchResult >= 0 ? searchResult : -(searchResult + 1);

      int newLength = insertionPoint + 1;

      if (newLength - 1 == lengthToLastNum.size()) {
        lengthToLastNum.add(num);
        continue;
      }
      lengthToLastNum.set(newLength - 1, num);
    }

    return lengthToLastNum.size();
  }
}
// step 2.1 その2 終わり
```

- 20分ぐらいで書いた
- `lengthToLastNum`は1はじまりではなく0はじまり
  - `lengthToLastNum.get(length - 1)`と-1が必要
  - 頭が混乱した
  - 初めは1はじまりにしようとしたが、0番目をどう扱うか困ってやめてしまった
    - `Arrays`なら、1はじまりも書きやすそう。清書はそうしてみる。
- 二分探索も自分で書くと良さそうだが、すでに時間がかかっているので、今回はやめる
- JavaのbinarySearchの返り値はわかりづらい
  - https://docs.oracle.com/javase/jp/25/docs/api/java.base/java/util/Collections.html#binarySearch(java.util.List,T)


## step 2.2 清書
「step 2.1 その2」の方針。LISの末尾のうち最小のものを管理する方法。

```java
// step 2.2 清書
class Solution {
  public int lengthOfLIS(int[] nums) {
    int[] lengthToLastNum = new int[nums.length + 1];
    int longest = 0;

    for (int num : nums) {
      int length = Arrays.binarySearch(lengthToLastNum, 1, longest + 1, num);
      length = length >= 0 ? length : -(length + 1);

      longest = Math.max(longest, length);
      lengthToLastNum[length] = num;
    }

    return longest;
  }
}
```


# step 3
1回目（ミスあり）：12:54、2回目（ミスあり）：6:39、3回目（ミスあり）：7:10、
4回目（ミスあり）：5:50、5回目：2:20、6回目：1:49、7回目：2:00

- 生じたミス
  - スペルミス（lognest）
  - Arrays.binarySearchの引数の順番
  - lengthToLastNumにすべきところをnumsとした
  - `return lengthToMinTail[longest]`としてしまった
- 少し時間が空くとどこかミスしてしまう
  - 理解できてないのだと思う
  - 時間を空けなければ、暗記で書けてしまう

```java
class Solution {
  public int lengthOfLIS(int[] nums) {
    int[] lengthToMinTail = new int[nums.length + 1];
    int longest = 0;

    for (int num : nums) {
      int length = Arrays.binarySearch(lengthToMinTail, 1, longest + 1, num);
      length = length >= 0 ? length : -length - 1;

      longest = Math.max(longest, length);
      lengthToMinTail[length] = num;
    }

    return longest;
  }
}
```

- 変数名`lengthToMinTail`は https://github.com/chryschron/codings/pull/29 を参考にした

