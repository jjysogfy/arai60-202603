問題：https://leetcode.com/problems/first-unique-character-in-a-string/

# step 1
- 取り組んだことがあると思う

```java
class Solution {
  public int firstUniqChar(String s) {
    Set<Character> seen = new HashSet<>();
    Set<Character> seen_once = new HashSet<>();
    for (char c : s.toCharArray()) {
      if (seen_once.contains(c)) {
        seen_once.remove(c);
        continue;
      }
      if (seen.contains(c)) {
        continue;
      }
      seen_once.add(c);
      seen.add(c);
    }

    for (int i = 0; i < s.length(); ++i) {
      if (seen_once.contains(s.charAt(i))) {
        return i;
      }
    }
    return -1;
  }
}
```

- 6分ぐらい
- camelCaseにすべきだ……

```java
class Solution {
  public int firstUniqChar(String s) {
    var char_seen_once_to_index = new LinkedHashMap<Character, Integer>();
    Set<Character> seen = new HashSet<>();
    for (int i = 0; i < s.length(); ++i) {
      char c = s.charAt(i);
      if (char_seen_once_to_index.containsKey(c)) {
        char_seen_once_to_index.remove(c);
        continue;
      }
      if (seen.contains(c)) {
        continue;
      }
      char_seen_once_to_index.put(c, i);
      seen.add(c);
    }

    if (char_seen_once_to_index.isEmpty()) {
      return -1;
    }
    return char_seen_once_to_index.firstEntry().getValue();
  }
}
```

- LinkedHashSet を使うと簡単になる気がして書き換えてみた
  - かえって複雑になってしまった

```java
class Solution {
  public int firstUniqChar(String s) {
    var seen_once_to_index = new LinkedHashMap<Character, Integer>();
    Map<Character, Integer> seen_to_count = new HashMap<>();
    for (int i = 0; i < s.length(); ++i) {
      final char c = s.charAt(i);

      final int count = seen_to_count.merge(c, 1, Integer::sum);
      switch (count) {
        case 1 -> seen_once_to_index.put(c, i);
        case 2 -> seen_once_to_index.remove(c);
      }
    }

    if (seen_once_to_index.isEmpty()) {
      return -1;
    }
    return seen_once_to_index.firstEntry().getValue();
  }
}
```

- `Set<> seen`の代わりに`Map<> seen_to_count`を使う
- ちょっとすっきりしたのでは


# step 2
- https://github.com/goto-untrapped/Arai60/pull/17/changes
- https://github.com/ryoooooory/LeetCode/pull/20/changes

コメント集
- https://discord.com/channels/1084280443945353267/1201211204547383386/1211166072552816680
  - 「一回だけ見た、とかですかね。あ、unique でいいか。」


# step 3

