```java
class Solution {
  static final int LAND = 1;

  public static int maxAreaOfIsland(int[][] grid) {
    int height = grid.length;
    int width = grid[0].length;
    boolean[][] seen = new boolean[height][width];

    int result = 0;
    for (int row = 0; row < height; ++row) {
      for (int col = 0; col < width; ++col) {
        if (grid[row][col] != LAND || seen[row][col]) {
          continue;
        }
        result = Math.max(result, computeArea(grid, row, col, seen));
      }
    }
    return result;
  }

  private static int computeArea(int[][] grid, int startRow, int startCol, boolean[][] seen) {
    int area = 0;
    Deque<Position> landsToVisit = new ArrayDeque<>();
    landsToVisit.push(new Position(startRow, startCol));
    while (!landsToVisit.isEmpty()) {
      var p = landsToVisit.pop();
      if (seen[p.row()][p.col()]) {
        continue;
      }
      seen[p.row()][p.col()] = true;
      ++area;
      pushIfLand(grid, seen, p.row() + 1, p.col(), landsToVisit);
      pushIfLand(grid, seen, p.row() - 1, p.col(), landsToVisit);
      pushIfLand(grid, seen, p.row(), p.col() + 1, landsToVisit);
      pushIfLand(grid, seen, p.row(), p.col() - 1, landsToVisit);
    }
    return area;
  }

  private static void pushIfLand(
      int[][] grid, boolean[][] seen, int row, int col, Deque<Position> landsToVisit
  ) {
    int height = grid.length;
    int width = grid[0].length;
    if (!(0 <= row && row < height && 0 <= col && col < width)) {
      return;
    }
    if (grid[row][col] != LAND) {
      return;
    }
    if (seen[row][col]) {
      return;
    }
    landsToVisit.push(new Position(row, col));
  }

  record Position(int row, int col) {}
}
```

- 16分。1回で通った
