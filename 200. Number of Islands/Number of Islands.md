# 200. Number of Islands

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- DFS,BFSをつかって、連続した島を明らかにしていく
- 1探索ごとに見つけた島はチェックをしておく
- 書くマスが、見たことない島なら探索開始
- 探索した回数が島の個数

計算量
- 時間計算量O（M＊N）、空間計算量O(M*N)
    - M = N の時 9.0*10^4 10マイクロ秒と見積もり

DFSで探索する方法
```cpp
#include <vector>

class Solution {
public:
    int numIslands(const std::vector<std::vector<char>>& grid) {
        auto column_num = grid.size();
        auto row_num = grid[0].size();
        auto visited_islands = std::vector(column_num, std::vector<int>(row_num));
        auto island_count = 0;
        for (int i = 0; i < column_num; i++) {
            for (int j = 0; j < row_num; j++) {
                if (isValid(i, j, column_num, row_num, visited_islands, grid)) {
                    island_count++;
                    travelIsland(i, j, column_num, row_num, visited_islands, grid);
                }
            }
        }
        return island_count;
    }
private:
    static constexpr int kColumnDiff[4] = {1, -1, 0, 0};
    static constexpr int kRowDiff[4] = {0, 0, 1, -1};
    // isValidは「まだ探索していない」「島マスである」「indexが範囲内である」ときにtrueを返す
    static bool isValid(int col, int row, int colum_num, int row_num,
                            const std::vector<std::vector<int>>& visited_islands,
                            const std::vector<std::vector<char>>& grid) {
        if (col < 0 || col >= colum_num || row < 0 || row >= row_num) {
            return false;
        }
        if (visited_islands[col][row]) {
            return false;
        }
        if (grid[col][row] == '0') {
            return false;
        }
        return true;
    }
    // travelIslandは、開始地点から上下左右に連続する島マスをDFSによって探索する
    static void travelIsland(int column, int row, int column_num, int row_num,
                                std::vector<std::vector<int>>& visited_islands,
                                const std::vector<std::vector<char>>& grid) {
        
        visited_islands[column][row] = 1;
        for (int i = 0; i < 4; i++) {
            auto next_column = column + kColumnDiff[i];
            auto next_row = row + kRowDiff[i];
            if (isValid(next_column, next_row,
                        column_num, row_num, visited_islands, grid)) {
                travelIsland(next_column, next_row, column_num,
                                row_num, visited_islands, grid);
            }
        }
    }
};
```

BFSで探索する方法
```cpp
#include <queue>
#include <vector>

struct Position {
    int column;
    int row;
    Position(int column, int row) : column(column), row(rowm) {}
};

class Solution {
public:
    int numIslands(const std::vector<std::vector<char>>& grid) {
        auto column_num = grid.size();
        auto row_num = grid[0].size();
        auto seen_islands = std::vector(column_num, std::vector<int>(row_num));
        auto island_count = 0;
        for (int i = 0; i < column_num; i++) {
            for (int j = 0; j < row_num; j++) {
                if (isValid(i, j, column_num, row_num, seen_islands, grid)) {
                    island_count++;
                    travelIsland(i, j, column_num, row_num, seen_islands, grid);
                }
            }
        }
        return island_count;
    }
private:
    static constexpr int kColumnDiff[4] = {1, -1, 0, 0};
    static constexpr int kRowDiff[4] = {0, 0, 1, -1};
    // isValidは「まだ探索していない」「島マスである」「indexが範囲内である」ときにtrueを返す
    static bool isValid(int col, int row, int colum_num, int row_num,
                            const std::vector<std::vector<int>>& seen_islands,
                            const std::vector<std::vector<char>>& grid) {
        if (col < 0 || col >= colum_num || row < 0 || row >= row_num) {
            return false;
        }
        if (seen_islands[col][row]) {
            return false;
        }
        if (grid[col][row] == '0') {
            return false;
        }
        return true;
    }
    // travelIslandは、開始地点から上下左右に連続する島マスをBFSによって探索する
    static void travelIsland(int start_column, int start_row,
                                int column_num, int row_num,
                                std::vector<std::vector<int>>& seen_islands,
                                const std::vector<std::vector<char>>& grid) {
        std::queue<Position> next_island_position;
        next_island_position.emplace(start_column, start_row);
        while (!next_island_position.empty()){
            auto [column, row] = next_island_position.front();
            next_island_position.pop();
            for (int i = 0; i < 4; i++) {
                auto next_column = column + kColumnDiff[i];
                auto next_row = row + kRowDiff[i];
                if (isValid(next_column, next_row, column_num,
                            row_num, seen_islands, grid)) {
                    seen_islands[next_column][next_row] = 1;
                    next_island_position.emplace(next_column, next_row);
                }
            }
        }
    }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/mura0086/arai60/pull/21
- https://github.com/WNomunomu/leetcode/pull/1/files
- https://github.com/quinn-sasha/leetcode/pull/18/files
- https://github.com/fuga-98/arai60/pull/18/files
- https://github.com/ichika0615/arai60/pull/9/files

- ドキュメント系

teachers' eye
- 略称は避けよう　https://github.com/mura0086/arai60/pull/21/files#r2033329425
- vector vool は特殊 https://github.com/mura0086/arai60/pull/21/files#r2033326989
    - https://qiita.com/voidhoge/items/383244bad2d728a18dbe
        - boolをintと同じように1byteにすると無駄が多いため、各要素が1bitになるようにしてメモリを節約
        - 通常と同じように参照できないので、参照を指すproxy patternを介するのだそう。
- num_ , sum_, max_, min_,のような略語は一般的　https://qiita.com/voidhoge/items/383244bad2d728a18dbe
    - num_　は numbers_ofの略だったのか, numberの略と思っていた
- 等号の向きをそろえて一直線上に書く　https://github.com/mura0086/arai60/pull/21/files#r2033346904
- 自然言語で表したときに自然な変数名に　https://github.com/mura0086/arai60/pull/21/files#r2033278200
- union-findの設計　https://github.com/ichika0615/arai60/pull/9/files#r1954436002
    - 自分もとりあえずは「一般的な」union_findを書いてそれを使う方法を考えてしまいそうだ
        - ただ、具体的にやりたい操作を思い浮かべれば、もっと良いunion_findを設計できるはず


#### 感想
- STEP1のコードはgridが空の時にエラーが起きる（grid[0].size()がとれない）

- traverse (横切る、ジグザグに上る)という単語もあるのか、travelとの選択は好みの範疇か
- '0'を'sea'、'1'を'island'となずけるのはなるほど必要だと思った。「'0','1'」だけ書いてあっても何かわからない
- "isValid"　は"isNewIsland"とかのほうが良いか
- for文中のi,jもcolumn, rowの方が良い
- kColumnDiff, kRowDiffはprivate定数でなく関数内定数にしたほうがいいのか
    - クラスの中に他にも関数があって、別の関数もkColumnDiff, kRowDiff使うならprivate変数にするべきだ。そうでないならどちらでもよいと思った。
- next_island_position は island_to_visitのほうが分かりやすい命名と思った
- 範囲内かの判定と、既に見た島であるかの判定も別にすべきか
    - isNewIsland という名前の関数であれば、まとめてもよいと思った


#### STEP1以外の手法と感想
- Union-find　をして連結成分の個数を数える
    - 操作が直感的になるので良さそうだと感じた（想定手法ではないだろうけど）
    - 2週目の宿題にしよう
- DFSかBFSの選択の話
    - 再帰だとデバッグの難しさや、再帰回数の上限の問題があり、BFSを選択することで著しく何かを犠牲にするのでなければBFSが良さそうだ
- 入力を破壊して空間計算量を抑える
    - マスを見たときに、それを書き換えていくことでis_visited_islandの2次元配列やmapを作らずにすむ
    - メモリの制約が厳しい時は、採用すべきと思った
    - 一般的には別に管理するデータを持った方が良いか

DFSのbrush_up
```cpp
#include <vector>

class Solution {
public:
    int numIslands(const std::vector<std::vector<char>>& grid) {
        if (grid.empty()) {
            return 0;
        }
        auto num_column = grid.size();
        auto num_row = grid[0].size();
        auto is_visited_island = std::vector(num_column, std::vector<int>(num_row));
        auto island_count = 0;
        for (int column = 0; column < num_column; column++) {
            for (int row = 0; row < num_row; row++) {
                if (isNewIsland(column, row, num_column, num_row,
                                is_visited_island, grid)) {
                    traverseIsland(column, row, num_column, num_row,
                                    is_visited_island, grid);
                    island_count++;
                }
            }
        }
        return island_count;
    }
private:
    static constexpr char kSea = '0';
    static constexpr int kColumnDiff[4] = {1, -1, 0, 0};
    static constexpr int kRowDiff[4] = {0, 0, 1, -1};
    // is_validは「まだ探索していない」「島マスである」「indexが範囲内である」ときにtrueを返す
    static bool isNewIsland(int column, int row, int num_column, int num_row,
                            const std::vector<std::vector<int>>& is_visited_island,
                            const std::vector<std::vector<char>>& grid) {
        if (!(0 <= column && column < num_column) || !(0 <= row && row < num_row)) {
            return false;
        }
        if (is_visited_island[column][row]) {
            return false;
        }
        if (grid[column][row] == kSea) {
            return false;
        }
        return true;
    }
    // traverse_islandは、開始地点から上下左右に連続する島マスをDFSによって探索する
    static void traverseIsland(int column, int row, int num_column, int num_row,
                                std::vector<std::vector<int>>& is_visited_island,
                                const std::vector<std::vector<char>>& grid) {
        
        is_visited_island[column][row] = 1;
        for (int i = 0; i < 4; i++) {
            auto next_column = column + kColumnDiff[i];
            auto next_row = row + kRowDiff[i];
            if (isNewIsland(next_column, next_row, num_column, num_row,
                            is_visited_island, grid)) {
                traverseIsland(next_column, next_row, num_column, num_row,
                                is_visited_island, grid);
            }
        }
    }
};
```
## STEP3
### 3回ミスなく書く
BFSのbrush_upを3回

```cpp
#include <queue>
#include <vector>

struct Position {
    int column;
    int row;

    Position(int column, int row) : column(column), row(row) {}
};

class Solution {
public:
    int numIslands(const std::vector<std::vector<char>>& grid) {
        if (grid.empty()) {
            return 0;
        }
        auto num_column = grid.size();
        auto num_row = grid[0].size();
        auto is_seen_island = std::vector(num_column, std::vector<int>(num_row));
        auto island_count = 0;
        for (int column = 0; column < num_column; column++) {
            for (int row = 0; row < num_row; row++) {
                if (isNewIsland(column, row, num_column, num_row,
                                is_seen_island, grid)) {
                    traverseIsland(column, row, num_column, num_row,
                                    is_seen_island, grid);
                    island_count++;
                }
            }
        }
        return island_count;
    }

private:
    static constexpr int kColumnDiff[4] = {0, 0, 1, -1};
    static constexpr int kRowDiff[4] = {1, -1, 0, 0};
    static constexpr char kSea = '0';

    static bool isNewIsland(int column, int row, int num_column, int num_row,
                            std::vector<std::vector<int>>& is_seen_island,
                            const std::vector<std::vector<char>>& grid) {
        if (!(0 <= column && column < num_column) || !(0 <= row && row < num_row)) {
            return false;
        }
        if (is_seen_island[column][row]) {
            return false;
        }
        if (grid[column][row] == kSea) {
            return false;
        }
        return true;
    }

    static void traverseIsland(int start_column, int start_row, int num_column, int num_row,
                                std::vector<std::vector<int>>& is_seen_island,
                                const std::vector<std::vector<char>>& grid) {
        std::queue<Position> island_to_visit;
        island_to_visit.emplace(start_column, start_row);
        while (!island_to_visit.empty()) {
            auto [column, row] = island_to_visit.front();
            island_to_visit.pop();
            for (int i = 0; i < 4; i++) {
                auto next_column = column + kColumnDiff[i];
                auto next_row = row + kRowDiff[i];
                if (isNewIsland(next_column, next_row, num_column, num_row,
                                is_seen_island, grid)) {
                    island_to_visit.emplace(next_column, next_row);
                    is_seen_island[next_column][next_row] = 1;
                }
            }
        }
    }
};
```

- 何回か練習したが、どうしても15分ほどかかってしまう
- ラムダ式で関数内関数のようにかけば、引数の多さを削減できるが、コードの見やすさは損なわれる
- ここら辺の感覚も今後つかんでいきたい

#### 2週目の宿題
Union_findでの実装、
