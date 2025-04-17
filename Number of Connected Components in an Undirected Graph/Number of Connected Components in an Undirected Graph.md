# Number of Connected Components in an Undirected Graph

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 与えられた配列を、隣接リストに変換する
- DFSで隣接リストを参照し、頂点を探索
- BFS, Union-Findでも求めることはできそう

計算量
- 時間計算量 O(N^2) 辺の数の上限が(N^2)/2 であるため
- ステップ数は N + (N^2)/2   N = 100のとき、5100ステップ 5.1マイクロ秒と見積り
- 空間計算量 O(N) 

```cpp
#include <set>
#include <vector>

class Solution {
    public:
        int countComponents(int n, const std::vector<std::vector<int>>& edges) {
            std::set<int> visited_vertexes;
            std::vector<std::vector<int>> adjacency_list(n);
            for (const auto& edge : edges) {
                auto from = edge[0];
                auto to = edge[1];
                adjacency_list[from].push_back(to);
                adjacency_list[to].push_back(from);
            }
            auto connected_components = 0;
            for (int vertex = 0; vertex < n; vertex++) {
                if (visited_vertexes.count(vertex)) {
                    continue;
                }
                connected_components++;
                TraverseVertexes(vertex, adjacency_list, visited_vertexes);
            }
            return connected_components;
        }
    private:
        void TraverseVertexes(int vertex, const std::vector<std::vector<int>>& adjacency_list,
                                std::set<int>& visited_vertexes) {
            visited_vertexes.insert(vertex);
            for (auto next_vertex : adjacency_list[vertex]) {
                if (visited_vertexes.contains(next_vertex)) {
                    continue;
                }
                TraverseVertexes(next_vertex, adjacency_list, visited_vertexes);
            }
        }
    };
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/Fuminiton/LeetCode/pull/19/files
- https://github.com/ryoooooory/LeetCode/pull/22/files
- https://github.com/olsen-blue/Arai60/pull/19/files
- https://github.com/Hurukawa2121/leetcode/pull/19/files
- https://github.com/colorbox/leetcode/pull/33/files

- ドキュメント系

teachers' eye
- 自然言語での説明と変数の名前　https://github.com/Fuminiton/LeetCode/pull/19/files#r1986520019
- 再帰のスタックオーバー　https://github.com/Fuminiton/LeetCode/pull/19/files#r1986953033
- 同じものを指しているのに表記が違うとよくない　https://github.com/olsen-blue/Arai60/pull/19/files#r1921050516
- 変数名に対する感覚　https://github.com/olsen-blue/Arai60/pull/19/files#r1919880884


#### 感想
- vertex という単語はあまり使わなそう、 nodeでよいか。
- TraverseConnectedNodes とかにしよう
- Componentsもnodeで統一
- Union_find,Path Splittingの最適化は初めて聞いた
- visited判定をsetでなくvectorでする場合、メモリ節約のため、std::uint8_tを使うのはなるほどと思った
- 今後負の数が想定されないなら、uintを使ったほうが明確でいいと思った

#### STEP1以外の手法と感想
- Union_Find と BFS 
	- 今回は頂点の数も少ないので再帰のデメリットも少なそうだ
	- Union_Findもかなり相性の良い手法だ。連結成分のカウントができればよいなら、お客さんに出すとしたらUnion_Findを使うかな
	- デバッグなどもしやすそう
- stackを使ったDFSを実装したことがなかったので、やってみよう
- とういうかこの場合はstack(LIFO)だろうがqueue(FIFO)だろうが達成可能だ(queueの場合はBFSと同じ)
- DFSと同じ道順にするならstackにする必要はある
- 帰りがけの処理は不要

stack(FIFO)でもできることを確認してみる
```cpp
#include <queue>
#include <set>
#include <vector>

class Solution {
    public:
        int countComponents(int n, const std::vector<std::vector<int>>& edges) {
            std::vector<std::vector<int>> adjacency_list(n);
            std::set<int> seen_nodes;
            for (const auto& edge : edges) {
                auto from = edge[0];
                auto to = edge[1];
                adjacency_list[from].push_back(to);
                adjacency_list[to].push_back(from);
            }
            auto num_connected_nodes = 0;
            for (int node = 0; node < n; node++) {
                if (seen_nodes.contains(node)) {
                    continue;
                }
                num_connected_nodes++;
                TraverseConnectedNodes(node, adjacency_list, seen_nodes);
            }
            return num_connected_nodes;
        }
    private:
        void TraverseConnectedNodes(int start, const std::vector<std::vector<int>>& adjacency_list,
                                std::set<int>& seen_nodes) {
            std::queue<int> node_to_visit;
            node_to_visit.push(start);
            while (!node_to_visit.empty()) {
                auto node = node_to_visit.front();
                node_to_visit.pop();
                for (auto next_node : adjacency_list[node]) {
                    if (seen_nodes.contains(next_node)) {
                        continue;
                    }
                    seen_nodes.insert(next_node);
                    node_to_visit.push(next_node);
                }
            }
        }
    };
```
BFS(queueかstackの差しかない)
```cpp
#include <stack>
#include <set>
#include <vector>

class Solution {
    public:
        int countComponents(int n, const std::vector<std::vector<int>>& edges) {
            std::vector<std::vector<int>> adjacency_list(n);
            std::set<int> seen_nodes;
            for (const auto& edge : edges) {
                auto from = edge[0];
                auto to = edge[1];
                adjacency_list[from].push_back(to);
                adjacency_list[to].push_back(from);
            }
            auto num_connected_nodes = 0;
            for (int node = 0; node < n; node++) {
                if (seen_nodes.count(node)) {
                    continue;
                }
                num_connected_nodes++;
                TraverseConnectedNodes(node, adjacency_list, seen_nodes);
            }
            return num_connected_nodes;
        }
    private:
        void TraverseConnectedNodes(int start, const std::vector<std::vector<int>>& adjacency_list,
                                	std::set<int>& seen_nodes) {
            std::stack<int> node_to_visit;
            node_to_visit.push(start);
            while (!node_to_visit.empty()) {
                auto node = node_to_visit.top();
                node_to_visit.pop();
                for (auto next_node : adjacency_list[node]) {
                    if (seen_nodes.count(next_node)) {
                        continue;
                    }
                    seen_nodes.insert(next_node);
                    node_to_visit.push(next_node);
                }
            }
        }
    };
```

## STEP3
### 3回ミスなく書く

stackで
```cpp
#include <set>
#include <stack>
#include <vector>

class Solution {
public:
	int countComponents(int n, const std::vector<std::vector<int>>& edges) {
		std::vector<std::vector<int>> adjacency_list(n);
		for (const auto& edge : edges) {
			auto from = edge[0];
			auto to = edge[1];
			adjacency_list[from].push_back(to);
			adjacency_list[to].push_back(from);
		}
		std::set<int> seen_island;
		auto num_connected_nodes = 0;
		for (int node = 0; node < n; node++) {
			if (seen_island.count(node)) {
				continue;
			}
			TraverseConnectedNodes(node, adjacency_list, seen_island);
			num_connected_nodes++;
		}
		return num_connected_nodes;
	}
private:
	void TraverseConnectedNodes(int start, const std::vector<std::vector<int>>& adjacency_list,
								std::set<int>& seen_island) {
		std::stack<int> node_to_visit;
		node_to_visit.push(start);
		while (!node_to_visit.empty()) {
			auto node = node_to_visit.top();
            node_to_visit.pop();
			for (auto next_node : adjacency_list[node]) {
				if (seen_island.count(next_node)) {
					continue;
				}
				seen_island.insert(next_node);
				node_to_visit.push(next_node);
			}
		}
	}
};
```

9分,6分,6分で3回Accept

#### 2週目の宿題
Halving SplittingでのUnion_findを実装して、適用する
