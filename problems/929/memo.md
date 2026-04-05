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
  - （`while (true) {`末尾の`++index;`を忘れたなど）
- ネストがかなり深くなってしまった
  - 浅くするには？ うまく関数化する、「`+`を読んだあと」というフラグを作る、など？
  - うまくやる方法を思いつかなかったので、そのまま書いた
- 例外`IllegalArgumentException`はよく考えずに書いた
  - 例外の種類をよく調べられていない


# step 2
- https://github.com/goto-untrapped/Arai60/pull/8/changes
  - フラグ変数を使っている（`UniqueEmailAddressesStep4_2.java`）
  - `String`のメソッドを活用する方法もある
  - `split`、`replace`、`replaceAll`など
    - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/lang/String.html
  - 参考になる変数名：`formattedEmail`（上の`result`相当）、`uniqueEmails`

- https://github.com/ryoooooory/LeetCode/pull/19/changes
  - `email.charAt(i)`を利用
    - 勝手な勘違いで、`charAt`はサロゲートペアの関係で遅いと思っていた。そんなことはなさそう
  - `email.toCharArray()`と`email.charAt(i)`のどちらが良いのかはよくわからなかった
  - 「ホスト部に＠が来る場合もある」とのこと
    - Wikipediaにある例：`"very.(),:;<>[]\".VERY.\"very@\\ \"very\".unusual"@strange.example.com`

- https://github.com/seal-azarashi/leetcode/pull/14/changes
  - 「ユーザーがゴミを1つ突っ込んできたら例外投げて動かないコードでいいんですか」とのコメント
  - 正規表現を使う方法

コメント集も軽く見ておいた。
- ユースケースの想定 https://discord.com/channels/1084280443945353267/1251052599294296114/1254245440690589787
  - https://discord.com/channels/1084280443945353267/1355903616032178327/1370028729186910269
  - 想定外の入力について、いつどういう選択をとるべきなのか、よくわかっていない
  - 例外を投げずに続けるとしたらどういう選択があるだろうか。`canonicalize`が`""`を返すとか？
  - Yoshiki-Iwasa氏は`return i32::MIN;`している

```java
// `String`のメソッドを活用
// 例外を投げないことにしてみた
// 変更部分のみ
  public String canonicalize(String email) {
    int indexOfAtSign = email.lastIndexOf("@");
    if (indexOfAtSign == -1) {
      return "";
    }
    String localName = email.substring(0, indexOfAtSign);
    String domainName = email.substring(indexOfAtSign + 1);

    int indexOfPlus = localName.indexOf("+");
    if (indexOfPlus == -1) {
      indexOfPlus = localName.length();
    }
    String formattedLocalName = localName.substring(0, indexOfPlus).replace(".", "");

    return formattedLocalName + "@" + domainName;
  }
```

- seal-azarashi氏を参考に、正規表現を使うコードも：

```java
// 正規表現を利用
// 変更部分のみ
  public String canonicalize(String email) {
    String formatted = email.replaceAll("\\.(?=.*@)", "")
      .replaceAll("\\+.*(?=@)", "");
    return formatted;
  }
```

- ところでこのコード、計算量が二乗かかりそうな気がする
  - 正規表現をよく知らないので自信がない
  - コメント集にあるようにそれでも問題ない場面は多いかもしれない（ https://discord.com/channels/1084280443945353267/1330509715712643112/1363506350927118356 ）


## 清書
```java
class Solution {
  public int numUniqueEmails(String[] emails) {
    Set<String> canonicalizedEmails = new HashSet<>();
    for (String email : emails) {
      canonicalizedEmails.add(this.canonicalize(email));
    }
    return canonicalizedEmails.size();
  }

  public String canonicalize(String email) {
    String[] splitEmail = email.split("@", 2);
    String localName = splitEmail[0];
    String domainName = splitEmail[1];

    String formattedLocalName = localName.split("\\+", 2)[0]
      .replaceAll("\\.", "");
    return formattedLocalName + "@" + domainName;
  }
}
```

- 想定外の入力の処理をしていない
  - @が一つもないときだけ例外`ArrayIndexOutOfBoundsException`
- ところで、`numUniqueEmails`、`canonicalize`はstaticでも良い気がする


# step 3
3回書いて、どれも3分程度で通った。

```java
class Solution {
  public int numUniqueEmails(String[] emails) {
    Set<String> canonicalizedEmails = new HashSet<>();
    for (String email : emails) {
      canonicalizedEmails.add(Solution.canonicalize(email));
    }
    return canonicalizedEmails.size();
  }

  public static String canonicalize(String email) {
    String[] splitEmail = email.split("@", 2);
    String localName = splitEmail[0];
    String domainName = splitEmail[1];

    String formattedLocalName = localName.split("\\+", 2)[0]
      .replaceAll("\\.", "");
    return formattedLocalName + "@" + domainName;
  }
}
```


# 次回について
- 時間がかかっているのと、4月から忙しくなったのとでなかなか進まない
- 次回からは、別の方法 https://discord.com/channels/1084280443945353267/1366778718705553520/1450943270799671337 で試す
- 最低でも週2（土日）は進める
- 次の問題：https://leetcode.com/problems/first-unique-character-in-a-string/

