問題：https://leetcode.com/problems/unique-email-addresses/


# step 1
- 取り組んだことがある
- 各`email`から余計な部分を除いた文字列を作る
  - 一字ずつ見る

```java
class Solution {
  public int numUniqueEmails(String[] emails) {
    Set<String> canonicalizedEmails = new HashSet<>();
    for (String email : emails) {
      canonicalizedEmails.add(this.canonicalize(email));
    }
    return canonicalizedEmails.size();
  }

  private String canonicalize(String email) {
    char[] emailChars = email.toCharArray();
    var result = new StringBuilder();

    int index = 0;
    while (true) {
      if (index == emailChars.length) {
        throw new IllegalArgumentException();
      }

      if (emailChars[index] == '.') {
        ++index;
        continue;
      }

      if (emailChars[index] == '+') {
        while (emailChars[index] != '@') {
          ++index;
          if (index == emailChars.length) {
            throw new IllegalArgumentException();
          }
        }
        break;
      }

      if (emailChars[index] == '@') {
        break;
      }

      result.append(emailChars[index]);
      ++index;
    }
    result.append('@');
    ++index;

    result.append(email.substring(index));
    return result.toString();
  }
}
```

- 15分ぐらい
  - 手を動かした時間だけ。実際にはもっとかかっている
- 一発では通らず
  - `while (true) {`末尾の`++index;`を忘れたなど
- ネストがかなり深くなってしまった
  - 浅くするには？ うまく関数化する、「`+`を読んだあと」というフラグを作る、など？
  - うまくやる方法を思いつかなかったので、そのまま書いた
- 例外`IllegalArgumentException`はよく考えずに書いた
  - 例外の種類をよく調べられていない


# step 2

