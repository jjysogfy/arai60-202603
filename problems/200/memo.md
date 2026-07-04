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

- 取り組んだことがある
- 30分かかったが、一発でちゃんと通った
  - Javaについて調べたりしつつだった（2D配列の初期化、record型など）
- DFS
- 迷ったところ：`p`が範囲内の点か、などチェックするタイミング？
  - ループのはじめでチェックしてみた
  - stackに`push`するとき（ループの最後）でも良いが、Javaでどう書くのが簡単かわからない


# step 2
コメント集
- 「（前略）とはいえメソッドなので、雑に扱っても構わないものにしておきたい気持ちがあるんです。」
  - https://discord.com/channels/1084280443945353267/1227073733844406343/1234185601993932891
- 範囲チェックの方法について
  - 「ここの部分は先にチェックするか一回 queue に入れて、後からチェックするかでしょう。」「上下左右方向の座標の差分を表す配列と、範囲外にアクセスしようとしているかどうかを判定する関数を用意し、ループで方向を回して処理したほうが」
  - https://discord.com/channels/1084280443945353267/1206101582861697046/1316539211779801148
- 別の書き方
  - 「全部同じ方法で書き直すよりも、バリエーションを持たせても良さそうに感じました。visitedを使わない、iterative DFS/BFS、union findなど色々あるかと。」「他には、有効な次のrow, colのみをstackに入れるとか、入力は変更するが、関数がreturnする前には元に戻すとかもありますね。」
  - https://github.com/rimokem/arai60/pull/17
- https://github.com/attractal/leetcode/pull/8

Javaでたくさん解いている方々
- https://github.com/goto-untrapped/Arai60/pull/38/changes
- https://github.com/ryoooooory/LeetCode/pull/2/changes

感想：
- UnionFind、BFSなど、いろいろ書き方を試したくはある
  - ArrayDequeなど、Javaの練習にもなりそう
- しかし、時間を取りすぎるので、step 1の方針のままやってみる
- 範囲の条件を、stackに`push`するとき（ループの最後）チェックするようにはしておきたい
  - Javaだと、チェックしてpushするlambdaは書きづらそう
  - かといってメソッドにすると、引数が増えてちょっと邪魔くさい
  - deltaを用意するのが良いか

## 清書
```java
class Solution {
  static final char WATER = '0';
  static final char LAND = '1';

  public int numIslands(char[][] grid) {
    int result = 0;
    int height = grid.length;
    int width = grid[0].length;
    boolean[][] isVisited = new boolean[height][width];

    for (int row = 0; row < height; ++row) {
      for (int col = 0; col < width; ++col) {
        if (grid[row][col] != LAND || isVisited[row][col]) {
          continue;
        }
        traverse(grid, row, col, isVisited);
        ++result;
      }
    }
    return result;
  }

  private void traverse(char[][] grid, int startRow, int startCol, boolean[][] isVisited) {
    int height = grid.length;
    int width = grid[0].length;
    ArrayDeque<Point> landsToVisit = new ArrayDeque<>();
    landsToVisit.push(new Point(startRow, startCol));

    while (!landsToVisit.isEmpty()) {
      Position position = landsToVisit.pop();
      if (isVisited[position.row()][position.col()]) {
        continue;
      }
      isVisited[position.row()][position.col()] = true;

      List<Position> directions = List.of(
        new Position(1, 0), new Position(-1, 0), new Position(0, 1), new Position(0, -1)
      );
      for (Position d : directions) {
        Position toVisit = new Position(position.row() + d.row(), position.col() + d.col());
        if (!isInRectangle(height, width, toVisit)) {
          continue;
        }
        if (grid[toVisit.row()][toVisit.col()] != LAND) {
          continue;
        }
        if (isVisited[toVisit.row()][toVisit.col()]) {
          continue
        }

        landsToVisit.push(toVisit);
      }
    }

    private boolean isInRectangle(int height, int width, Position p) {
      return 0 <= p.row() && p.row() < height
        && 0 <= p.col() && p.col() < width;
    }
  }

  private record Position(int row, int col) {}
}
```

- goto-untrapped氏を参考にした
- 範囲チェックを、stackにpushするときに行う
  - コメントを見て、`traverse`メソッドの（仮）引数も範囲チェックしたいと思ったが、コードの重複も増えるし今回は気が進まない
- Positionの変数名`position`と`toVisit`はかなり悩む……
- なんとなく見た目にゴチャゴチャした印象。`pushIfLand(grid, isVisited, row, col)`メソッドを書くほうが良い？
- 他：
  - 変数名：goto-untrapped氏を参考に、Point -> Position
  - step 1ではtraverseの引数にheight, widthを含めたが、そんなに気持ちよくない感じがしたのでやめた
  - 変数名：DFSに使うstackの変数名は？ goto-untrapped氏はconnected。なるほど、操作より意味を見てる感じ（？）
    - 今回は、別の人が使っていたlandsToVisitとしてみた（たとえば https://github.com/quinn-sasha/leetcode/pull/18/changes ）


# step 3
1回目（ミス）：15分、2回目（ミス）：11:45、3回目：12:53、4回目：7:43、5回目：7:47

- 少し時間は10分に収まってないが、今回はこれで良しとしておく。
- `directions`をループの外に出して、変数名を変更してみた
  - これなら一文字変数`p`もそんなに悪くないと、個人的には感じる

```java
class Solution {
  static final char LAND = '1';

  public int numIslands(char[][] grid) {
    int height = grid.length;
    int width = grid[0].length;
    boolean[][] isVisited = new boolean[height][width];

    int result = 0;
    for (int row = 0; row < height; ++row) {
      for (int col = 0; col < width; ++col) {
        if (grid[row][col] != LAND || isVisited[row][col]) {
          continue;
        }
        traverse(grid, row, col, isVisited);
        ++result;
      }
    }
    return result;
  }

  private void traverse(char[][] grid, int startRow, int startCol, boolean[][] isVisited) {
    int height = grid.length;
    int width = grid[0].length;
    var landsToVisit = new ArrayDeque<Position>();
    landsToVisit.push(new Position(startRow, startCol));
    List<Position> directions = List.of(
      new Position(0, 1), new Position(0, -1), new Position(1, 0), new Position(-1, 0)
    );

    while (!landsToVisit.isEmpty()) {
      Position p = landsToVisit.pop();
      if (isVisited[p.row()][p.col()]) {
        continue;
      }
      isVisited[p.row()][p.col()] = true;

      for (Position direction : directions) {
        Position toVisit = new Position(p.row() + direction.row(), p.col() + direction.col());
        if (
          !(0 <= toVisit.row() && toVisit.row() < height
          && 0 <= toVisit.col() && toVisit.col() < width)
        ) {
          continue;
        }
        if (grid[toVisit.row()][toVisit.col()] != LAND) {
          continue;
        }
        if (isVisited[toVisit.row()][toVisit.col()]) {
          continue;
        }
        landsToVisit.push(toVisit);
      }
    }
  }

  record Position(int row, int col) {}
}
```


# step 4
時間を置いてから、見ないで書いてみる。レビューも念頭に。

時間は8:50ぐらい。

```java
public class Solution {
  static final char LAND = '1';

  public int numIslands(char[][] grid) {
    int height = grid.length;
    int width = grid[0].length;
    boolean[][] visited = new boolean[height][width];

    int result = 0;
    for (int row = 0; row < height; ++row) {
      for (int col = 0; col < width; ++col) {
        if (grid[row][col] != LAND || visited[row][col]) {
          continue;
        }
        ++result;
        traverse(grid, row, col, visited);
      }
    }
    return result;
  }

  public record Cell(int row, int col) {
  }

  void traverse(char[][] grid, int startRow, int startCol, boolean[][] visited) {
    int height = grid.length;
    int width = grid[0].length;
    Deque<Cell> cellsToVisit = new ArrayDeque<>();
    cellsToVisit.push(new Cell(startRow, startCol));

    while (!cellsToVisit.isEmpty()) {
      Cell cell = cellsToVisit.pop();
      if (!(0 <= cell.row() && cell.row() < height && 0 <= cell.col() && cell.col() < width)) {
        continue;
      }
      if (grid[cell.row()][cell.col()] != LAND) {
        continue;
      }
      if (visited[cell.row()][cell.col()]) {
        continue;
      }
      visited[cell.row()][cell.col()] = true;

      cellsToVisit.push(new Cell(cell.row() + 1, cell.col()));
      cellsToVisit.push(new Cell(cell.row() - 1, cell.col()));
      cellsToVisit.push(new Cell(cell.row(), cell.col() + 1));
      cellsToVisit.push(new Cell(cell.row(), cell.col() - 1));
    }
  }
}
```

- 結局、popしたあとに範囲チェックするコードに。step 1と同じ。これだとコードが短くはなる。
- レビューを反映し、変数名はisVisitedではなくvisitedに直した
