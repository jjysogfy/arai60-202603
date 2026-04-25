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

- Union-Findで書いてみた。25分

