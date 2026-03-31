- `email.charAt(i)`を利用
  - 勝手な勘違いで、Unicodeの関係で遅いと思っていた
    - 勘違いを言葉にしておきたい
    - まずUnicodeをよくわかってないので、基本的なことをChatGPTに訊いた（そのうちもう少しちゃんと勉強すべきかも）
    - 前から`i`番目のコードポイントは、時間計算量`O(len)`でしか求まらなさそう、と思った
    - しかしこれは別の話だった：`charAt(i)`はコードポイントのことではないし、`codePointAt(i)`の`i`も「前から`i`番目」という意味ではなかった




```java
// `String`のメソッドを活用
// `indexOf`を使う
class Solution {
  public int numUniqueEmails(String[] emails) {
    Set<String> canonicalizedEmails = new HashSet<>();
    for (String email : emails) {
      canonicalizedEmails.add(this.canonicalize(email));
    }
    return canonicalizedEmails.size();
  }

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
}
```

```java
// `String`のメソッドを活用
// `split`を使う
import java.util.regex.Pattern;

class Solution {
  public int numUniqueEmails(String[] emails) {
    Set<String> canonicalizedEmails = new HashSet<>();
    for (String email : emails) {
      canonicalizedEmails.add(this.canonicalize(email));
    }
    return canonicalizedEmails.size();
  }

  public String canonicalize(String email) {
    String[] splitEmail = email.split("@");
    String localName = splitEmail[0];
    String domainName = splitEmail[1];

    // String formattedLocalName = localName.split("\\+")
    // String formattedLocalName = localName.split("\\Q+\\E")[0]
    String formattedLocalName = localName.split(Pattern.quote("+"))[0]
      .replace(".", "");

    return formattedLocalName + "@" + domainName;
  }
}
```

