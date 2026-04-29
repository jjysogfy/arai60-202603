問題（695. Max Area of Island）：https://leetcode.com/problems/max-area-of-island/

# step 1
```java
// step 1 その1 Union-Find
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
// step 1 その2 BFS
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


## 清書
せっかくなのでUnion-findでやってみる。

```java
// step 2 清書
class Solution {
  static final int LAND = 1;

  public static int maxAreaOfIsland(int[][] grid) {
    int height = grid.length;
    int width = grid[0].length;

    UnionFind<Cell> islands = new UnionFind<>(height * width,
        cell -> width * cell.row() + cell.col());

    for (int row = 0; row < height; ++row) {
      for (int col = 0; col < width; ++col) {
        if (grid[row][col] != LAND) {
          continue;
        }
        if (row + 1 < height && grid[row + 1][col] == LAND) {
          islands.union(new Cell(row, col), new Cell(row + 1, col));
        }
        if (col + 1 < width && grid[row][col + 1] == LAND) {
          islands.union(new Cell(row, col), new Cell(row, col + 1));
        }
      }
    }

    int maxArea = 0;
    for (int row = 0; row < height; ++row) {
      for (int col = 0; col < width; ++col) {
        if (grid[row][col] != LAND) {
          continue;
        }
        int area = islands.sizeOfComponent(new Cell(row, col));
        maxArea = Math.max(maxArea, area);
      }
    }
    return maxArea;
  }

  record Cell(int row, int col) {
  }
}

class UnionFind<T> {
  private int size;
  private int[] parents;
  private int[] sizes;
  public final ToIntFunction<T> itemToIndex;

  public UnionFind(int size, ToIntFunction<T> itemToIndex) {
    this.size = size;
    this.parents = IntStream.range(0, size).toArray();
    this.sizes = new int[size];
    Arrays.fill(this.sizes, 1);
    this.itemToIndex = itemToIndex;
  }

  public int size() {
    return this.size;
  }

  public int sizeOfComponent(T x) {
    int root = find(x);
    return this.sizes[root];
  }

  public int find(T x) {
    int index = itemToIndex.applyAsInt(x);
    return findFromIndex(index);
  }

  private int findFromIndex(int index) {
    int parent = this.parents[index];
    if (parent == index) {
      return index;
    }
    int root = findFromIndex(parent);
    this.parents[index] = root;
    return root;
  }

  public void union(T x, T y) {
    int rootX = find(x);
    int rootY = find(y);
    if (rootX == rootY) {
      return;
    }
    int smaller;
    int larger;
    if (this.sizes[rootX] <= this.sizes[rootY]) {
      smaller = rootX;
      larger = rootY;
    } else {
      smaller = rootY;
      larger = rootX;
    }
    this.parents[smaller] = larger;
    this.sizes[larger] += this.sizes[smaller];
  }
}
```


# step 3
1回目（ミスあり）：19分半、2回目（ミスあり）:17分、3回目（ミスあり）：18分、
4回目（ミスあり）：11分、5回目（ミスあり）：12分、6回目（ミスあり）：13分

バグらせたところ
- maxAreaOfIslandの1回目のループで、`== LAND`を`!= LAND`と書いた
- 2回目のループで`!= LAND`チェックを忘れた
  - このUnionFindはWATERもサイズ1と扱ってしまう
  - ここ、UnionFindの実装を変える手もあるが、ちょっと複雑になりそうなので、とりあえずやめておく
- return忘れ
- `UnionFind<Cell> islands`の宣言忘れ
- `new UnionFind<>()`の`<>`忘れ
  - raw type `new UnionFind()`だと、引数のlambdaで型エラーが出る
- `find`の呼び出しを`findByIndex`と書いた
- スペルミス（sizesをsizeと書いた）
- `union`で`sizes[ ]`付け忘れ（`sizes[larger] += smaller;`）
- ミスはしていないが、`union`で`parents[smaller] = larger;`を書くのにちょっと時間がかかる
  - `sizes`の更新と、あとひとつ何をするんだっけ、みたいに考えてしまう

