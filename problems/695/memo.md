問題（695. Max Area of Island）：https://leetcode.com/problems/max-area-of-island/

# step 1
```java
class Solution {
  static final int LAND = 1;

  public static int maxAreaOfIsland(int[][] grid) {
    int height = grid.length;
    int width = grid[0].length;

    UnionFind islands = new UnionFind(height * width);

    for (int row = 0; row + 1 < height; ++row) {
      for (int col = 0; col < width; ++col) {
        if (grid[row][col] != LAND || grid[row + 1][col] != LAND) {
          continue;
        }
        islands.union(positionToIndex(grid, row, col), positionToIndex(grid, row + 1, col));
      }
    }
    for (int row = 0; row < height; ++row) {
      for (int col = 0; col + 1 < width; ++col) {
        if (grid[row][col] != LAND || grid[row][col + 1] != LAND) {
          continue;
        }
        islands.union(positionToIndex(grid, row, col), positionToIndex(grid, row, col + 1));
      }
    }

    int area = 0;
    for (int row = 0; row < height; ++row) {
      for (int col = 0; col < width; ++col) {
        if (grid[row][col] != LAND) {
          continue;
        }
        area = Math.max(area, islands.sizeOfComponent(positionToIndex(grid, row, col)));
      }
    }
    return area;
  }

  private static int positionToIndex(int[][] grid, int row, int col) {
    return row + grid.length * col;
  }
}

class UnionFind {
  private int size;
  private int[] parents;
  private int[] sizes; // rootのみ意味のある値

  public UnionFind(int size) {
    this.size = size;
    this.parents = IntStream.range(0, size).toArray();
    this.sizes = IntStream.generate(() -> 1).limit(size).toArray();
  }

  public int size() {
    return this.size;
  }

  public int sizeOfComponent(int n) {
    n = find(n);
    return this.sizes[n];
  }

  public int find(int n) {
    int parent = this.parents[n];
    if (parent == n) {
      return n;
    }
    int root = find(parent);
    this.parents[n] = root;
    return root;
  }

  public void union(int m, int n) {
    m = find(m);
    n = find(n);
    if (m == n) {
      return;
    }
    int smaller;
    int larger;
    if (this.sizes[m] <= this.sizes[n]) {
      smaller = m;
      larger = n;
    } else {
      smaller = n;
      larger = m;
    }
    this.parents[smaller] = larger;
    this.sizes[larger] += this.sizes[smaller];
  }
}
```

- 前の問題はDFSで解いたので、今回はUnion-Findで書いてみた。25分
  - unionで`if (m == n) { return; }`を忘れるなどミスした
- BFSも書いておく

```java
// step 1の二つ目のコード
class Solution {
  static final int LAND = 1;

  public static int maxAreaOfIsland(int[][] grid) {
    int height = grid.length;
    int width = grid[0].length;
    boolean[][] seen = new boolean[height][width];

    int area = 0;
    for (int row = 0; row < height; ++row) {
      for (int col = 0; col < width; ++col) {
        if (grid[row][col] != LAND || seen[row][col]) {
          continue;
        }
        area = Math.max(area, computeArea(grid, row, col, seen));
      }
    }
  }

  private static int computeArea(int[][] grid, int startRow, int startCol, boolean[][] seen) {
    int height = grid.length;
    int width = grid[0].length;
    Deque<Position> landsToVisit = new ArrayDeque<>();
    landsToVisit.offer(new Position(startRow, startCol));

    int area = 0;
    while (!landsToVisit.isEmpty()) {
      Position p = landsToVisit.poll();
      if (!(0 <= p.row() && p.row() < height && 0 <= p.col() && p.col() < width)) {
        continue;
      }
      if (grid[p.row()][p.col()] != LAND) {
        continue;
      }
      if (seen[p.row()][p.col()]) {
        continue;
      }
      seen[p.row()][p.col()] = true;
      ++area;

      landsToVisit.offer(new Position(p.row() + 1, p.col()));
      landsToVisit.offer(new Position(p.row() - 1, p.col()));
      landsToVisit.offer(new Position(p.row(), p.col() + 1));
      landsToVisit.offer(new Position(p.row(), p.col() - 1));
    }
    return area;
  }

  private record Position(int row, int col) {}
}
```

- 13分。`return area;`忘れで1ミス
- （このようにキューから取り出すときにseenなどチェックするほうがコードが短いが、キューに追加する前にチェックしたほうが挙動は自然な気もする）
- どこにprivate/staticを付けるかは悩む


# step 2 : 読む
- 「（注：変数名`lands_queue`について）`lands`と書かれていますが実際には海の座標も入りうるので」
  - https://github.com/Hurukawa2121/leetcode/pull/17#discussion_r1898871975
  - たしかに。前回はチェックしたものだけをスタックに追加していたから`landsToVisit`で良かったが、今回は違う
- 「UnionFind側の実装の都合が露出してしまってる」「union() に２つのセルのrow, columnを渡すというインターフェースにしたほうが」
  - https://github.com/hroc135/leetcode/pull/18#discussion_r1770740750

