# 695. Max Area of Island

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 前問の 200. Number of Islands (島（連結成分）の数を数える)とほぼ同様の操作を行う
- とりあえず、前回後回しにしたUnionFindでの実装で実行しようと思った
    - gridでのunionFindを実装する
        - 一番シンプルなのは数直線上の要素でのものだが、2次元でも同様のことができる
        - アルゴリズム的には全く同じ、intの代わりにPosition型, parentsの配列も二次元に
- 秋葉拓哉さんのスライドを見てまねた https://www.slideshare.net/slideshow/ss-3578491/3578491
- 経路圧縮のみの場合、だいたいlogのオーダーで操作可能のようだ

- 前問のコードを改良して、DFSやBFSでも解いてみた
  - BFSはの方が島の数え方はシンプルで、重複なく、島マスのみqueueに入ることを保証するなら、queueから取り出した数が島のサイズ
  - DFSは枝分かれ先に対して、その先にある島のサイズを再帰的に返してもらって、それを足し合わせていくイメージ

計算量
- 時間計算量 O(N*Mlog(N*M)) 空間計算量 O(N*M) N = M = 50 のとき 5.6 * 10^4 ステップ　56マイクロ秒と見積り


```cpp
#include <map>
#include <vector>

struct Position {
    int row;
    int column;
    auto operator<=>(const Position&) const = default;

    Position(int row, int column) : row(row), column(column) {}
};

class unionFind {
public:
    unionFind(int num_row, int num_column) : num_row_(num_row), num_column_(num_column),
                                            parents_(num_row_, std::vector<Position>(num_column_, Position{0,0})) {
        for (int r = 0; r < num_row_; r++) {
            for (int c = 0; c < num_column; c++) {
                parents_[r][c] = Position{r, c};
            }
        }
    }
        
    Position find(Position position) {
        if (parents_[position.row][position.column] == position) {
            return position;
        }
        // 再帰的に親をたどって、経路圧縮を行う
        parents_[position.row][position.column] = find(parents_[position.row][position.column]);
        return parents_[position.row][position.column];
    }
    
    //position2の親をposition1の親に繋ぐ
    void merge(Position position1, Position position2) {
        auto parent1 = find(position1);
        auto parent2 = find(position2);
        if (parent1 == parent2) {
            return;
        }
        parents_[parent2.row][parent2.column] = parent1;
    }

    int numElemntsOfMaxGroup () {
        std::map<Position, int> position_to_num_elements;
        for (int row = 0; row < num_row_; row++) {
            for (int column = 0; column < num_column_; column++) {
                auto parent = find(Position{row, column});
                position_to_num_elements[parent]++;
            }
        }
        auto max_size = 0;
        for (const auto& [_, num_elements] : position_to_num_elements) {
            if (num_elements > max_size) {
                max_size = num_elements;
            }
        }
        return max_size;
    }
private:
    const int num_row_;
    const int num_column_;
    std::vector<std::vector<Position>> parents_;
};

class Solution {
public:
    int maxAreaOfIsland(const std::vector<std::vector<int>>& grid) {
        if (grid.empty() || isOnlySea(grid)) {
            return 0;
        }
        auto num_row = grid.size();
        auto num_column = grid[0].size();
        unionFind island_union (num_row, num_column);

        for (int r = 0; r < num_row; r++) {
            for (int c = 0; c < num_column; c++) {
                if (grid[r][c] != kIsland) {
                    continue;
                }
                for (int i = 0; i < 4; i++) {
                    auto next_r = r + kRowDiff[i];
                    auto next_c = c + kColumnDiff[i];
                    if (!(0 <= next_r && next_r < num_row) ||
                        !(0 <= next_c && next_c < num_column)) {
                        continue;
                    }
                    if (grid[next_r][next_c] != kIsland) {
                        continue;
                    }
                    island_union.merge(Position{r, c}, Position{next_r, next_c});
                }
            }
        }
        return island_union.numElemntsOfMaxGroup();
    }
private:
    static constexpr int kRowDiff[4] = {0, 0, 1, -1};
    static constexpr int kColumnDiff[4] = {1, -1, 0, 0};
    static constexpr int kIsland = 1;
    //unionFindを用いると、各グループの最低要素数が1になるため、すべて海だとカウントを誤る
    //このため、事前にgridがすべて海かを確認する
    bool isOnlySea (const std::vector<std::vector<int>>& grid) {
        for (const auto& row : grid) {
            for (auto cell : row) {
                if (cell == kIsland) {
                    return false;
                }
            }
        }
        return true;
    }
};
```

DFSを利用する
```cpp
#include <vector>

class Solution {
public:
    int maxAreaOfIsland(const std::vector<std::vector<int>>& grid) {
        if (grid.empty()) {
            return 0;
        }
        auto num_row = grid.size();
        auto num_column = grid[0].size();
        auto is_visited_island = std::vector(num_row, std::vector<int>(num_column));
        auto max_island_size = 0;
        for (int r = 0; r < num_row; r++) {
            for (int c = 0; c < num_column; c++) {
                if (isUnvisitedIsland(r, c, num_row, num_column,
                                        grid, is_visited_island)) {
                    auto island_size = IslandSize(r, c, num_row,
                                                    num_column, grid, is_visited_island);
                    if (max_island_size < island_size) {
                        max_island_size = island_size;
                    }
                }
            }
        }
        return max_island_size;
    }
private:
    static constexpr int kSea = 0;
    static constexpr int kRowDiff[4] = {0, 0, 1, -1};
    static constexpr int kColumnDiff[4] = {1, -1, 0, 0};
    // isUnvisitedIsland「まだ探索していない」「島マスである」「indexが範囲内である」ときにtrueを返す
    static bool isUnvisitedIsland(int row, int column, int num_row, int num_column,
                                    const std::vector<std::vector<int>>& grid,
                                    const std::vector<std::vector<int>>& is_visited_island) {
        if (!(0 <= row && row < num_row) ||
            !(0 <= column && column < num_column)) {
            return false;
        }
        if (is_visited_island[row][column]) {
            return false;
        }
        if (grid[row][column] == kSea) {
            return false;
        }
        return true;
    }
    // IslandSizeは、開始地点から上下左右に連続する島マスをDFSによって探索し、島全体のサイズを返す
    static int IslandSize(int row, int column, int num_row, int num_column,
                                const std::vector<std::vector<int>>& grid,
                                std::vector<std::vector<int>>& is_visited_island) {
        
        is_visited_island[row][column] = 1;
        auto island_size = 1;
        for (int i = 0; i < 4; i++) {
            auto next_row = row + kRowDiff[i];
            auto next_column = column + kColumnDiff[i];
            if (isUnvisitedIsland(next_row, next_column, num_row, num_column,
                                    grid, is_visited_island)) {
                island_size += IslandSize(next_row, next_column,
                                            num_row, num_column,
                                            grid, is_visited_island);
            }
        }
        return island_size;
    }
};
```

BFSで実装する
```cpp
#include <queue>
#include <vector>

struct Position {
    int row;
    int column;

    Position(int row, int column) : row(row), column(column) {}
};

class Solution {
public:
    int maxAreaOfIsland(const std::vector<std::vector<int>>& grid) {
        if (grid.empty()) {
            return 0;
        }
        auto num_row = grid.size();
        auto num_column = grid[0].size();
        auto is_seen_island = std::vector(num_row, std::vector<int>(num_column));
        auto max_size = 0;
        for (int r = 0; r < num_row; r++) {
            for (int c = 0; c < num_column; c++) {
                if (isUnvisitedIsland(r, c, num_row, num_column,
                                        grid, is_seen_island)) {
                    auto island_size = IslandSize(r, c, num_row, num_column,
                                                    grid, is_seen_island);
                    if (island_size > max_size) {
                        max_size = island_size;
                    }
                }
            }
        }
        return max_size;
    }

private:
    static constexpr int kRowDiff[4] = {1, -1, 0, 0};
    static constexpr int kColumnDiff[4] = {0, 0, 1, -1};
    static constexpr int kSea = 0;
    // isUnvisitedIsland「まだ探索していない」「島マスである」「indexが範囲内である」ときにtrueを返す
    static bool isUnvisitedIsland(int row, int column, int num_row, int num_column,
                                    const std::vector<std::vector<int>>& grid,
                                    const std::vector<std::vector<int>>& is_seen_island) {
        if (!(0 <= row && row < num_row) ||
            !(0 <= column && column < num_column)) {
            return false;
        }
        if (is_seen_island[row][column]) {
            return false;
        }
        if (grid[row][column] == kSea) {
            return false;
        }
        return true;
    }
    // IslandSizeは、開始地点から上下左右に連続する島マスをBFSによって探索し、島全体のサイズを返す
    static int IslandSize(int start_row, int start_column, int num_row, int num_column,
                            const std::vector<std::vector<int>>& grid,
                            std::vector<std::vector<int>>& is_seen_island) {
        std::queue<Position> island_to_visit;
        island_to_visit.emplace(start_row, start_column);
        is_seen_island [start_row][start_column] = 1;
        auto island_size = 0;
        while (!island_to_visit.empty()) {
            auto [row, column] = island_to_visit.front();
            island_to_visit.pop();
            island_size++;
            for (int i = 0; i < 4; i++) {
                auto next_row = row + kRowDiff[i];
                auto next_column = column + kColumnDiff[i];
                if (isUnvisitedIsland(next_row, next_column, num_row, num_column,
                                        grid, is_seen_island)) {
                    island_to_visit.emplace(next_row, next_column);
                    is_seen_island[next_row][next_column] = 1;
                }
            }
        }
        return island_size;
    }
};
```


## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/quinn-sasha/leetcode/pull/19
- https://github.com/ichika0615/arai60/pull/13/files
- https://github.com/fuga-98/arai60/pull/19
- https://github.com/Fuminiton/LeetCode/pull/18/files

- ドキュメント系

teachers' eye
- スコープと変数の名前の丁寧さ　https://github.com/quinn-sasha/leetcode/pull/19#discussion_r2006746463
  - 短いなら簡易なものでも許容されるかも 簡単なfor文のiとかは確かに皆使う
- 名前空間とIDとオブジェクト　https://github.com/ichika0615/arai60/pull/13/files#r1989314225
  - pythonだとこんなことになるのか、特定の整数型のidが予約されているとか、+=するとidが変わるとか
  - c++は参照やポインタがあるので、また事情が違うか
- 副作用　https://github.com/olsen-blue/Arai60/pull/29#discussion_r1948645912


#### 感想
- 「島を走査して島のサイズを返す関数」の名前がSTEP1時に自分はあまりしっくりこなかった
  - 皆さんは　count_island_area, traverse_and_measure_island, visit_and_get_area, calculate_areaなど
  - traverse_and_measure_island　が返り値がintであることも一番自然に伝えられるか
- unionFindについて、持つデータ構造は、2次元配列じゃなくてmapで良い
- unionFindのメンバ変数に、「親が束ねる島の大きさ」のデータ構造を持っておけばいいのか
  - 一番大きな島のサイズを求めるのが簡単になる
    
- DFS/BFSの際に、次に調べるマスが有効（未踏、範囲内、島である）かをチェックし、有効なら調べるというのが私のやり方だったが
- とりあえず調べて、そのマスが無効であれば島の数として0個を返すというやり方があってなるほどと思った。
  - 今回自分は有効なマスだけ調べる保証があった方が心配がなくて好みだが、この方法も覚えておこう
  - マスを踏んだ時に確率で島が沈んで無くなる、みたいに前のマスにいる時に次のマスがまだ有効かわからない場合に有効
- 4方向をベタ書きするやり方も結構目にした。方向が少ないなら、diffを別に持ってforで参照しない方が見やすいという感覚を理解した
    - さらに定数を書き下す必要がなく、4方向はほぼコピペですむので早い

- pythonの関数内関数、便利だな、c++はラムダ式が同様の機能だけど、見た目がややこしい

#### STEP1以外の手法と感想
- 関数の引数としてisland_sizeを参照で渡して、それを更新するやり方もあるなと思った　←副作用がバグの温床になる可能性があり、理由がない限りやめた方がいいようだ
- もしくは木DPというんだろうか？　二次元配列を用意して、今のマスを根とした部分木の頂点の数を配列に書き込んでいく
  - 関数の返り値として返すのではなく、DFSの帰りがけの処理で、親が子のマスの値を見て、（子の合計+1）を自分のマスに書き込むというやり方
  - これも同様に副作用がある
- DP?を使うとさらにM*Nの空間計算量が必要なことは気にするべき
- 前問と同様に入力を破壊してgridを書き換えながら走査すればM*Nだけ空間計算量を削減できる

BFSのコードがが長すぎるので、スリムに書く
無効なマスもqueueに入るので、有効なマスの時のみ、island_sizeをインクリメントする

```cpp
#include <queue>
#include <set>
#include <vector>


class  Solution {
public:
    int maxAreaOfIsland(const std::vector<std::vector<int>>& grid) {
        if (grid.empty()) {
            return 0;
        }
        std::set<std::pair<int, int>> is_seen_island;
        auto max_size = 0;
        auto num_row = grid.size();
        auto num_column = grid[0].size();
        for (int r = 0; r < num_row; r++) {
            for (int c = 0; c < num_column; c++) {
                if (isUnvisitedIsland(r, c, num_row, num_column, grid, is_seen_island)) {
                    auto island_size = 0;
                    std::queue<std::pair<int, int>> island_to_visit;
                    island_to_visit.emplace(r, c);
                    while (!island_to_visit.empty()) {
                        auto [row, column] = island_to_visit.front();
                        island_to_visit.pop();
                        if (!isUnvisitedIsland(row, column, num_row, num_column, grid, is_seen_island)) {
                            continue;
                        } 
                        island_size++;
                        is_seen_island.emplace(row, column);
                        island_to_visit.emplace(row + 1, column);
                        island_to_visit.emplace(row - 1, column);
                        island_to_visit.emplace(row, column + 1);
                        island_to_visit.emplace(row, column - 1);
                    }
                    if (island_size > max_size) {
                        max_size = island_size;
                    }
                }
            }
        }
        return max_size;
    }
private:
    static const int  kSea = 0;
    static bool isUnvisitedIsland(int r, int c, int num_row, int num_column,
                                    const std::vector<std::vector<int>>& grid,
                                    const std::set<std::pair<int, int>>& is_seen_island) {
        if (!(0 <= r && r < num_row) ||!(0 <= c && c < num_column)) {
            return false;
        }
        if (grid[r][c] == kSea) {
            return false;
        }
        if (is_seen_island.count({r, c})) {
            return false;
        }
        return true;
    }
};

```

## STEP3
### 3回ミスなく書く

unionFind
ランク付けとsize_のmapも持つ

```cpp
#include <algorithm>
#include <map>
#include <vector>

struct Position {
    int row;
    int column;

    auto operator <=> (const Position&) const = default;
};

class unionFind {
public:
    unionFind(int num_row, int num_column) : num_row_(num_row), num_column_(num_column) {
        for (int r = 0; r < num_row_; r++) {
            for (int c = 0; c < num_column_; c++) {
                Position position = {r, c};
                parents_[position] = position;
                size_[position] = 1;
            }
        }
    }

    Position find(Position position) {
        if (parents_[position] == position) {
            return position;
        }
        parents_[position] = find(parents_[position]);
        return parents_[position];
    }

    void merge(Position position1, Position position2) {
        auto parent1 = find(position1);
        auto parent2 = find(position2);
        if (parent1 == parent2) {
            return;
        }
        if (size_[parent1] < size_[parent2]) {
            std::swap(parent1, parent2);
        }
        parents_[parent2] = parent1;
        size_[parent1] += size_[parent2];
    }

    int sizeOfMaxGroup() {
        int max_size = 0;
        for (const auto& [_, size] : size_) {
            max_size = std::max(max_size, size);
        }
        return max_size;
    }
private:
    const int num_row_;
    const int num_column_;
    std::map<Position, Position> parents_;
    std::map<Position, int> size_;
};

class Solution {
public:
    int maxAreaOfIsland(const std::vector<std::vector<int>>& grid) {
        if (grid.empty() || isOnlySea(grid)) {
            return 0;
        }
        auto num_row = grid.size();
        auto num_column = grid[0].size();
        unionFind island_union(num_row, num_column);
        for (int r = 0; r < num_row; r++) {
            for (int c = 0; c < num_column; c++) {
                if (grid[r][c] == kSea) {
                    continue;
                }
                for (int i = 0; i < 4; i++) {
                    auto next_r = r + kRowDiff[i];
                    auto next_c = c + kColumnDiff[i];
                    if (!(0 <= next_r && next_r < num_row) ||
                        !(0 <= next_c && next_c < num_column) ||
                        grid[next_r][next_c] == kSea) {
                            continue;
                        }
                    island_union.merge(Position{r, c}, Position{next_r, next_c});
                }
            }
        }
        return island_union.sizeOfMaxGroup();
    }

private:
    static constexpr int kSea = 0;
    static constexpr int kRowDiff[4] = {1, -1, 0, 0};
    static constexpr int kColumnDiff[4] = {0, 0, 1, -1};
    static bool isOnlySea(const std::vector<std::vector<int>>& grid) {
        for (const auto& row : grid) {
            for (auto cell : row) {
                if (cell != kSea) {
                    return false;
                }
            }
        }
        return true;
    }
};
```

- union_findを8分で書けるようになった
- その他の部分は9分程度

#### 2周目の宿題
DFSやBFSのその他の方針の実装
