# 127. Word Ladder

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- ノードが揃ってないから、２つ同時に再帰を始めても、2つのノードが同じポジションに行けるとは限らず、困った
- 分からないので答えを見た
- 「nullのノードからはnullが生える」と擬似的に考えればノードを揃えることができる（両方nullなら打ち止めにする）
- こうすることでDFSが可能


計算量
各木のノード数をN,Mとして
- 時間計算量 O(N+M)
	- ステップ数は、正確に見積もる自信がないが、最悪で頂点と辺の数が合わせて3(N+M)ほど
	- N = M = 2000 の時、 12000ステップほど 12マイクロ秒と見積もり
- 空間計算量 O(N+M)
	- 実際は最大の木の高さ分のスタックが積まれるので、平均はlogNくらい

DFS
```cpp
#include <algorithm>
#include <vector>

class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if (!root1 && !root2) {
            return nullptr;
        }
        auto left1 = NextLeft(root1);
        auto left2 = NextLeft(root2);
        auto right1 = NextRight(root1);
        auto right2 = NextRight(root2);
        TreeNode* new_head = new TreeNode(NodeToValue(root1) + NodeToValue(root2));
        new_head->left = mergeTrees(left1, left2);
        new_head->right = mergeTrees(right1, right2);
        return new_head;
    }
private:
    int NodeToValue(TreeNode* node) {
        if (!node) {
            return 0;
        }
        return node->val;
    }

    TreeNode* NextLeft(TreeNode* node) {
        if (!node) {
            return nullptr;
        }
        return node->left;
    }      

    TreeNode* NextRight(TreeNode* node) {
        if (!node) {
            return nullptr;
        }
        return node->right;
    }    
};
```

(マージ元１、マージ元２、マージして新しく作るノード)の3点セットを持ってDFSすることによっても可能

BFS
```cpp
#include <algorithm>
#include <vector>

class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if (!root1 && !root2) {
            return nullptr;
        }
        std::vector<std::tuple<TreeNode*, TreeNode*, TreeNode*>> current_nodes;
        TreeNode* new_head = new TreeNode(NodeToVal(root1) + NodeToVal(root2));
        current_nodes.emplace_back(root1, root2, new_head);
        while (!current_nodes.empty()) {
        	// tuple {merge_base1, merge_base2, merged_node}
            std::vector<std::tuple<TreeNode*, TreeNode*, TreeNode*>> next_nodes;  
            for (auto [node1, node2, merged_node] : current_nodes) {
                auto left1 = NextLeft(node1);
                auto left2 = NextLeft(node2);
                auto right1 = NextRight(node1);
                auto right2 = NextRight(node2);
                if (left1 || left2) {
                    merged_node->left = new TreeNode (NodeToVal(left1) + NodeToVal(left2));
                    next_nodes.emplace_back(left1, left2, merged_node->left);
                }
                if (right1 || right2) {
                    merged_node->right = new TreeNode (NodeToVal(right1) + NodeToVal(right2));
                    next_nodes.emplace_back(right1, right2, merged_node->right);
                }
            }
            std::swap(current_nodes, next_nodes);
        }
        return new_head;
    }
private:
    int NodeToVal(TreeNode* node) {
        if (!node) {
            return 0;
        }
        return node->val;
    }

    TreeNode* NextLeft(TreeNode* node) {
        if (!node) {
            return nullptr;
        }
        return node->left;
    }      

    TreeNode* NextRight(TreeNode* node) {
        if (!node) {
            return nullptr;
        }
        return node->right;
    }    
};
```


## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/fuga-98/arai60/pull/23/files
- https://github.com/Fuminiton/LeetCode/pull/23/files
- https://github.com/seal-azarashi/leetcode/pull/22/files
- https://github.com/ryoooooory/LeetCode/pull/26/files
- https://github.com/t0hsumi/leetcode/pull/23/files


- ドキュメント系

teachers' eye
- メインで注目する変数に接頭辞をつけない方が好み　https://github.com/fuga-98/arai60/pull/23/files#r2001007482
- 生々しく、どのように使われるか想像しませんか　https://github.com/seal-azarashi/leetcode/pull/22/files#r1778932434

- 特殊ケース(null等)が扱いづらい場合に番兵を使う https://github.com/Fuminiton/LeetCode/pull/23/files#r1997660631
- 場合分け（繰り返し）が多い場合、なんとかならないか https://github.com/ryoooooory/LeetCode/pull/26/files#r1980815887
- どのような場合に、どのような状況でTLEになるのか考える https://github.com/hroc135/leetcode/pull/22/files#r1797638201
	- これは自分は甘い。徹底しなければ



#### 感想
- やっぱり三項演算子は認知不可の観点から積極的に使われていない（1行で分岐処理がかけるけど）
- NextLeft, NextRightなど書かなくても、TreeNode（番兵？）を作れば良いのか
- 自分はDFSもBFSもかなり回りくどく書いてしまっていて、どうすればシンプルに記述できるようになるのだろう


#### STEP1以外の手法と感想
- root1を上書きしていくような手法は、入力を破壊するのでユーザーが驚くと思い、避けたい
- 基本的には再帰（非再帰DFS）を用いるか、BFSを用いるかには分かれる　再帰上限は問題ない

- 現実のユースケースを想像するのは難しい。全然考えても出てこないのでGPTに聞いてみた
	- その中で組織合併というようなワードがあってなるほどとなった。
	- 市町村、県、地方、全国のような階層で営業所を持つ2つの会社があって、合併によって各階層の営業所の売り上げを合算したいというイメージ(X市にはA社しか営業所がないならば、A社のみの売り上げを記録する)
		- 途中経過で階層ごとの情報とかも収集したいのであれば、各階層ごとに処理をするBFS式の方がユーザー使いやすいのかなと思うので、私はBFSを推すことにする

DFSをシンプルに書いたもの
```cpp
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if (!root1 && !root2) {
            return nullptr;
        }
        if (!root1) {
            return root2;
        }
        if (!root2) {
            return root1;
        }
        return new TreeNode(root1->val + root2->val,
                            mergeTrees(root1->left, root2->left),
                            mergeTrees(root1->right, root2->right));
    }
};
```
DFS(番兵をたてても良い)
```cpp
class Solution {
public:
    TreeNode* mergeTrees(const TreeNode* root1, const TreeNode* root2) {
        if (!root1 && !root2) {
            return nullptr;
        }
        if (!root1) {
        	root1 = kSentinel;
        }
        if (!root2) {
        	root2 = kSentinel;
        } 
        return new TreeNode(root1->val + root2->val,
                            mergeTrees(root1->left, root2->left),
                            mergeTrees(root1->right, root2->right));
    }

private:           
    static const TreeNode* const kSentinel;
};

const TreeNode* const Solution::kSentinel = new TreeNode(0);  
```
- TreeNode* 型のポインタをkSentinelとして定数に置きたかったのだが、これでよかったのか自信がない
- KSentinelはポインタも中身も変更したくないので、const TreeNode* const　にした

BFSの修正
```cpp
#include <algorithm>
#include <vector>

class Solution {
public:
    TreeNode* mergeTrees(const TreeNode* root1, const TreeNode* root2) {
    	if (!root1 && !root2) {
    		return nullptr;
    	}
        if (!root1) {
        	root1 = kSentinel;
        }
        if (!root2) {
        	root2 = kSentinel;
        }
        std::vector<std::tuple<const TreeNode*, const TreeNode*, TreeNode*>> current_nodes;
        TreeNode* new_head = new TreeNode();
        current_nodes.emplace_back(root1, root2, new_head);
        while (!current_nodes.empty()) {
        	// tuple {merge_base1, merge_base2, merged_node}
            std::vector<std::tuple<const TreeNode*, const TreeNode*, TreeNode*>> next_nodes;  
            for (auto [node1, node2, merged_node] : current_nodes) {
            	if (!node1) {
            		node1 = kSentinel;
            	}
            	if (!node2) {
            		node2 = kSentinel;
            	}
            	merged_node->val = node1->val + node2->val;
            	if (node1->left || node2->left) {
            		merged_node->left = new TreeNode();
            		next_nodes.emplace_back(node1->left, node2->left, merged_node->left);
            	}
            	if (node1->right || node2->right) {
            		merged_node->right = new TreeNode();
            		next_nodes.emplace_back(node1->right, node2->right, merged_node->right);
            	}
            }
            std::swap(current_nodes, next_nodes);
        }
        return new_head;
    }
private:           
    static const TreeNode* const kSentinel;
};

const TreeNode* const Solution::kSentinel = new TreeNode(0); 
```



## STEP3
### 3回ミスなく書く
BFSで

```cpp
#include <algorithm>
#include <vector>

class Solution {
public:
	TreeNode* mergeTrees(const TreeNode* root1, const TreeNode* root2) {
		if (!root1 && !root2) {
			return nullptr;
		}
		if (!root1) {
			root1 = kSentinel;
		}
		if (!root2) {
			root2 = kSentinel;
		}
		TreeNode* new_head = new TreeNode();
		std::vector<std::tuple<const TreeNode*, const TreeNode*, TreeNode*>> current_nodes;
		current_nodes.emplace_back(root1, root2, new_head);
		while (!current_nodes.empty()) {
			std::vector<std::tuple<const TreeNode*, const TreeNode*, TreeNode*>> next_nodes;
			for (auto [node1, node2, merged_node] : current_nodes) {
				if (!node1) {
					node1 = kSentinel;
				}
				if (!node2) {
					node2 = kSentinel;
				}
				merged_node->val = node1->val + node2->val;
				if (node1->left || node2->left) {
					merged_node->left = new TreeNode();
					next_nodes.emplace_back(node1->left, node2->left, merged_node->left);
				}
				if (node1->right || node2->right) {
					merged_node->right = new TreeNode();
					next_nodes.emplace_back(node1->right, node2->right, merged_node->right);
				}
			}
			std::swap(current_nodes, next_nodes);
		}
		return new_head;
	}
private:
	static const TreeNode* const kSentinel;
};
const TreeNode* const Solution::kSentinel = new TreeNode(0);
```

9分,8分,6分で3回Accept

#### 余談
https://discord.com/channels/1084280443945353267/1196472827457589338/1200081854301225011

```python
a = [0] * 5
a[1] = 7
print(a)

b = [[0] * 5] * 3
b[1][3] = 7
print(b)
```
bの出力が[[0,0,0,7,0],[0,0,0,7,0],[0,0,0,7,0]]
になる話が興味深かった。

pythonのlistはオブジェクトへの参照を持つ構造らしい

c++でどう言うことなのかなと思って自分のイメージを再現して遊んだ
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int*> A;
    int a = 0; 
    for (int i = 0; i < 3; i++) {
    	A.push_back(&a);
    }

    std::vector<std::vector<int*>*> B;
    for (int i = 0; i < 3; i++) {
        B.push_back(&A); 
    }

    for (auto ptr_to_vec : B) {
        for (auto ptr_to_int : *ptr_to_vec) {
            std::cout << *ptr_to_int << " ";
        }
    }
    // 0,0,0,0,0,0,0,0,0
    std::cout << std::endl;

    int d = 1;
    (*B[0])[1] = &d;  

    for (auto ptr_to_vec : B) {
        for (auto ptr_to_int : *ptr_to_vec) {
            std::cout << *ptr_to_int << " ";
        }
    }
    // 0,1,0,0,1,0,0,1,0
}
```

#### ２周目の宿題
- 非再帰DFSでも実装する
- メンバー変数、定数について調べて、さらに理解を深める
