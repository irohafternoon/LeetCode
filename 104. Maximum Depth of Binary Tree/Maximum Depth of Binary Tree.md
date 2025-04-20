# 104. Maximum Depth of Binary Tree

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- DFSで深さの情報を保持しながら進む
- 本当はleft right　がnullかどうかで場合分けをしたいが、煩雑なので、TreeNodeがnullであることを終了条件にする
- その代わり、Nullのノードの深さは含まないので、返すあたりはdepth - 1 とする

時間計算量
- O(N)	
	- ステップ数はNodeの数+辺の数で3Nほど、N = 10^4 のとき 0.12ミリ秒程と見積もり
空間計算量
- 再帰分のO(N)

- 二分探索木をソラで書く問題と思って身構えていたが、そうではなかった。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
#include <algorithm>

class Solution {
public:
    int maxDepth(TreeNode* root) {
        return MeasureTreeDepth(root, 1);
    }   
private:
    int MeasureTreeDepth(TreeNode* node, int depth) {
        if (!node) {
            return depth - 1;  //nullのノードは深さに数えないので、1引いたものを返す
        }
        return std::max(MeasureTreeDepth(node->right, depth + 1),
                        MeasureTreeDepth(node->left, depth + 1 ));
    }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/ichika0615/arai60/pull/15/files
- https://github.com/Fuminiton/LeetCode/pull/21/files
- https://github.com/fuga-98/arai60/pull/21/files
- https://github.com/quinn-sasha/leetcode/pull/17/files
- https://github.com/ryoooooory/LeetCode/pull/24/files

- ドキュメント系
	- std::max https://cpprefjp.github.io/reference/algorithm/max.html
		- 2つものものにしか適用できないと思ってたが、比較できれば配列の中身でもできるのか

teachers' eye
- 配列の中身を統一させる　https://github.com/ichika0615/arai60/pull/15/files#r1995489030
- 変数名でなく、コメントでの情報　https://github.com/ichika0615/arai60/pull/15/files#r1997494323
>　データの整合性が取れているということは、読んでいる人からすると全部読み終わらないと分からないと思いますね。
- https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.ho7q4rvwsa1g
	- データの整合性というのが理解できたか怪しい。「配列の中は、ある境目に現階層と次階層のNodeがキレイに分けれてかれている」という書く側の前提は、読み手からは分からないということだろうか。
	- 最後まで見て「あ、これで動くのね。ということは入力がちゃんとキレイに分かれるようになってるってことなのね。」と思うということだと推測している


#### 感想
- 自然言語での詳細なステップごとの説明をコメントに含めている方がいて良いなと思った。
- あまりに長いものは難しいが、できるものは書いていきたい
- 再帰でも、depthをメモしなくてもいいということを理解した。(1+max(右の部分木の深さ, 左の部分木の深さ))
	- その代わり、nullのノードは0を返すように設定する

#### STEP1以外の手法と感想
- BFSによる手法
	- 1つのノードに次見るべきノードを次々入れる方法と
	- 現在の階層が入っている配列, 次の階層が入る配列の2つを用意しておき、、現在の階層の個々のノードからたどれる次の改装のノードを次階層の配列に入れる方法
	- 最後に現階層と次階層の配列をswapするイメージ
		- 後者の方法はinplace-dp(これは一般的な用語か分からない)に近い着想だなと思った。
	- コードは明らかにDFSが簡潔、再帰の限界はCが10M程度のようなので、今回の制約ではDFSを用いるのが素直か。

STEP1のbrush-up
```cpp
#include <algorithm>

class Solution {
public:
    int maxDepth(TreeNode* root) {
        return MeasureTreeDepth(root);
    }   
private:
    int MeasureTreeDepth(TreeNode* node) {
        if (!node) {
            return 0;
        }
        return 1 + std::max(MeasureTreeDepth(node->right),
                            MeasureTreeDepth(node->left));
    }
};
```

BFS(２つの配列を用意する方法)
```cpp
#include <algorithm>
#include <vector>

class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        std::vector<TreeNode*> current_nodes;
        current_nodes.push_back(root);
        int depth = 0;
        while (!current_nodes.empty()) {
            depth++;
            std::vector<TreeNode*> next_nodes;
        	for (auto node : current_nodes) {
        		if (node->left) {
        			next_nodes.push_back(node->left);
        		}
        		if (node->right) {
        			next_nodes.push_back(node->right);
        		}
        	}
        	std::swap(current_nodes, next_nodes);
        }
        return depth;
    }   
};
```

## STEP3
### 3回ミスなく書く

BFSで
```cpp
#include <algorithm>
#include <vector>

class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        std::vector<TreeNode*> current_nodes;
        current_nodes.push_back(root);
        int depth = 0;
        while (!current_nodes.empty()) {
            depth++;
            std::vector<TreeNode*> next_nodes;
            for (auto node : current_nodes) {
                if (node->right) {
                    next_nodes.push_back(node->right);
                }
                if (node->left) {
                    next_nodes.push_back(node->left);
                }
            }
            std::swap(current_nodes, next_nodes);
        }
        return depth; 
    }
};
```

6分,4分,3分で3回Accept

#### 2週目の宿題
- 配列（vectorを自分で作ってみる）今回とは関係ないが、やっていないなと思ったので
- そもそも二分探索木の実装をしてみる（BITなど）
- 二週目はもっといっぱいの人のコードを"読める"ようになりたい
	- そのためには最初は自分で他人のコードを実際に変えて動かしたり実験したりするのもいいかも
