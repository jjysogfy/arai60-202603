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
- https://github.com/goto-untrapped/Arai60/pull/8/changes
  - 文字列のメソッドを活用
  - `split`、`replace`、`replaceAll`など
  - ドキュメントを見なければ
    - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/lang/String.html
  - 一字ずつ見る方法（`UniqueEmailAddressesStep4_2.java`）では、フラグ変数を使っている
  - 変数名：`uniqueEmails`、`formattedEmail`（上の`result`相当）

- https://github.com/ryoooooory/LeetCode/pull/19/changes
  - `email.charAt(i)`を利用
    - 勝手な勘違いで、Unicodeの関係で遅いと思っていた
      - 勘違いを言葉にしておきたい
      - まずUnicodeをよくわかってないので、基本的なことをChatGPTに訊いた（そのうちもう少しちゃんと勉強すべきかも）
      - 前から`i`番目のコードポイントは、時間計算量`O(len)`でしか求まらなさそう、と思った
      - しかしこれは別の話だった：`charAt(i)`はコードポイントのことではないし、`codePointAt(i)`の`i`も「前から`i`番目」という意味ではなかった
  - `email.toCharArray()`と`email.charAt(i)`のどちらが良いのかはよくわからなかった
  - 「ホスト部に＠が来る場合もある」とのこと
    - Wikipediaにある例：`"very.(),:;<>[]\".VERY.\"very@\\ \"very\".unusual"@strange.example.com`

- https://github.com/seal-azarashi/leetcode/pull/14/changes
  - 「ユーザーがゴミを1つ突っ込んできたら例外投げて動かないコードでいいんですか」とのコメント
  - 正規表現を使う方法

