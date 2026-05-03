
# step 1
元々書いていた形：

- 思ったこといくつか：
- ALPHABETSの初期化は、あまり上手くない気がする
  - 1行で簡単に書きたかったが、思いつかずやめた
    - IntStreamはあるがCharStreamはない
    - 追記：サロゲートペアを考えると、`char[]`より`int[]`にすべきで、それなら書けるかも
    - 追記：`int[] ALPHABETS = IntStream.rangeClosed('a', 'z').toArray()`
    - 追記：`alteredAt`も変更が必要。`c`の代わりに`Character.toString(c)`とするなど
    - 追記：ただ、実際にサロゲートペアがあると`ladderLength`自体は壊れそう
    - 追記：`String[] ALPHABETS`という手もあるか
  - static初期化ブロックを初めて使った
- WordAndNumまわりの変数名は迷った
  - はじめ`Queue<WordAndNum> wordsToVisit`を`Queue<String>`で書いていた
  - `wordsAndNumsToVisit`とかはかえってわかりづらいと思い変数名はそのままにした
- 実行時間について
  - ステップ数はだいたい、`wordList.length * ALPHABETS.length * word.length^2`
    - つまり`10^5 * 10^2`ぐらい
    - nextWordの計算で`word.length`がひとつ掛かると思う
  - Javaは1秒`10^6`から`10^7`程度
    - 追記：覚え間違い！ `10^7`から`10^8`程度。クロック周波数が`1G=10^9`オーダーで、そこから1、2桁落ちる
  - `10^7`ステップだと危ういなあと思いつつ書いていた
  - 結果は272ms
- あとAIにレビューさせて気づいたことだが、ALPHABETSの変数名は単数形のほうが自然かも

# メモ
- 双方向BFS
- 高速なハミング距離
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1216123084889788486
