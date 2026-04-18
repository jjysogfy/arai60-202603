問題：https://leetcode.com/problems/number-of-islands/

# 取り組み方
基本的に、マニュアルの「標準的な進め方」にしたがう。
ただし、時間を取りすぎないよう、（取り組みが不十分でも）早めに切り上げることにする。


# step 1
```java
class Solution {
  static final char WATER = '0';
  static final char LAND = '1';

  public int numIslands(char[][] grid) {
    int result = 0;
    int height = grid.length;
    int width = grid[0].length;
    boolean[][] isSeen = new boolean[height][width];

    for (int y = 0; y < height; ++y) {
      for (int x = 0; x < width; ++x) {
        if (isSeen[y][x] || grid[y][x] == WATER) {
          continue;
        }
        traverse(y, x, height, width, grid, isSeen);
        ++result;
      }
    }
    return result;
  }

  private void traverse(int y0, int x0, int height, int width, char[][] grid, boolean[][] isSeen) {
    ArrayDeque<Point> nextPoints = new ArrayDeque<>();
    nextPoints.push(new Point(y0, x0));

    while (!nextPoints.isEmpty()) {
      Point p = nextPoints.pop();
      if (!(0 <= p.y() && p.y() < height && 0 <= p.x() && p.x() < width)) {
        continue;
      }
      if (isSeen[p.y()][p.x()] || grid[p.y()][p.x()] == WATER) {
        continue;
      }

      isSeen[p.y()][p.x()] = true;
      nextPoints.push(new Point(p.y() + 1, p.x()));
      nextPoints.push(new Point(p.y() - 1, p.x()));
      nextPoints.push(new Point(p.y(), p.x() + 1));
      nextPoints.push(new Point(p.y(), p.x() - 1));
    }
  }

  private record Point(int y, int x) {}
}
```

- 解いたことがある
- 30分かかったが、一発でちゃんと通った
  - Javaについて調べたりしつつだった（2D配列の初期化、record型など）
- DFS
- `p`がgrid内の点か、などのチェックをするタイミングについて
  - ループのはじめでチェックしてみた
  - `push`するときでも良いが、Javaでどう書くのが簡単かわからない


# step 2
コメント集と、Javaでたくさん解いた人のコードを見る。

- UnionFind
- BFS
- 「（前略）…とはいえメソッドなので、雑に扱っても構わないものにしておきたい気持ちがあるんです。」
  - https://discord.com/channels/1084280443945353267/1227073733844406343/1234185601993932891
- 範囲チェックの方法について
  - 「ここの部分は先にチェックするか一回 queue に入れて、後からチェックするかでしょう。」「上下左右方向の座標の差分を表す配列と、範囲外にアクセスしようとしているかどうかを判定する関数を用意し、ループで方向を回して処理したほうが」
  - https://discord.com/channels/1084280443945353267/1206101582861697046/1316539211779801148
- 「全部同じ方法で書き直すよりも、バリエーションを持たせても良さそうに感じました。visitedを使わない、iterative DFS/BFS、union findなど色々あるかと。」「他には、有効な次のrow, colのみをstackに入れるとか、入力は変更するが、関数がreturnする前には元に戻すとかもありますね。」
  - https://github.com/rimokem/arai60/pull/17
- https://github.com/attractal/leetcode/pull/8

Javaの方
- https://github.com/goto-untrapped/Arai60/pull/38/changes
- https://github.com/ryoooooory/LeetCode/pull/2/changes

感想：
- UnionFind、BFSなど、いろいろ書き方を試したくはある
  - ArrayDequeなど、Javaの練習にもなりそう
- しかし、時間を取りすぎるので、step 1の方針のままやってみる
- 範囲の条件をpush時にチェックするようにはしておきたい
  - Javaだと、こういうlambdaは書きづらそう（lambdaの型を指定しないといけない）
  - かといってメソッドにすると、引数が増えてちょっと邪魔くさい
  - deltaを用意するのが良いか

