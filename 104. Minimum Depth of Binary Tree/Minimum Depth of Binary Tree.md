# 104. Minimum Depth of Binary Tree
https://leetcode.com/problems/minimum-depth-of-binary-tree/description/

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- ひとつ前の問題（木の最大の深さを求める）と同様の手法が使えそう
- BFSをして、そのノードのrightもleftもnullなものがあった時点で、その時点の階層を返せばよい

計算量
- 時間計算量 O(N)
    - ステップで言うと頂点＋辺の数なので 3Nステップ
    - N = 10^5のとき0.3ミリ秒と見積り

- 空間計算量 O(N)


```cpp

#include <algorithm>
#include <vector>

class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        std::vector<TreeNode*> current_nodes;
        current_nodes.push_back(root);
        int depth = 1;
        while (!current_nodes.empty()) {
            std::vector<TreeNode*> next_nodes;
            for (auto node : current_nodes) {
                if (!node->right && !node->left) {
                    return depth;
                }
                if (node->left) {
                    next_nodes.push_back(node->left);
                }
                if (node->right) {
                    next_nodes.push_back(node->right);
                }
            }
            depth++;
            std::swap(current_nodes, next_nodes);
        }
        return -1;  //input_error
    }   
};

```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/ichika0615/arai60/pull/16/files
- https://github.com/Fuminiton/LeetCode/pull/22/files
- https://github.com/fuga-98/arai60/pull/22/files
- https://github.com/ryoooooory/LeetCode/pull/25/files

- ドキュメント系

teachers' eye
- どんな入力を想定していますか？　https://github.com/Fuminiton/LeetCode/pull/22/files#r1996628892
- 例外を狭める　https://github.com/Fuminiton/LeetCode/pull/22/files#r1997354335
- 例外処理に関する議論　https://github.com/fuga-98/arai60/pull/22/files#r1995974879
 


#### 感想
- 「入力が木じゃない」ことを考えもしなかった。入力の想定が甘い。
- 関数の最後の行に到達することが想定されない場合はどうすればいいんだろう。
    - 今回は木がループの場合は到達しないが、その場合はコードが終わらない
    - ループを判定するにはseen_nodesのようなものを管理すればよい
    - pythonは返り値なしも許されたり、返り値なし＆エラー投げるとかできるようだ
    - 無限ループにすることで最後の1行が消せることが判明　https://github.com/ryoooooory/LeetCode/pull/25/files#r1981036882
        - GPTに聞いたら、無限ループにすると、必ずwhile中のreturnで関数が終了すると解析できるため、らしい。

#### STEP1以外の手法と感想
- BFS以外にはDFSがある
    - 再帰の上限は今の制約上は問題ない。
    - ただしDFSは全てのノードを探索する必要があり、枝刈りはできない。
    - コードはDFSのほうが簡潔
- 自然言語の説明的にも合致するのはBFSかなと思う。

STEP1の改良（無限ループ版）
```cpp

#include <algorithm>
#include <vector>

class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        std::vector<TreeNode*> current_nodes;
        current_nodes.push_back(root);
        int depth = 1;
        while (true) {
            std::vector<TreeNode*> next_nodes;
            for (auto node : current_nodes) {
                if (!node->right && !node->left) {
                    return depth;
                }
                if (node->left) {
                    next_nodes.push_back(node->left);
                }
                if (node->right) {
                    next_nodes.push_back(node->right);
                }
            }
            depth++;
            std::swap(current_nodes, next_nodes);
        }
    }   
};

```

DFSでも実装
```cpp

#include <algorithm>

class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        return MeasureMinTreeDepth(root, 1);
    }   
private:
    int MeasureMinTreeDepth(TreeNode* node, int depth) {
        if (!node->left && !node->right) {
            return depth;
        }
        int min_depth = INT_MAX;
        if (node->right) {
            min_depth = std::min(MeasureMinTreeDepth(node->right, depth + 1), min_depth);
        }
        if (node->left) {
            min_depth = std::min(MeasureMinTreeDepth(node->left, depth + 1), min_depth);
        }
        return min_depth;
    }
};

```

もっとシンプルに書ける

```cpp

#include <algorithm>

class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        if (!root->left) {
            return 1 + minDepth(root->right);
        }
        if (!root->right) {
            return 1 + minDepth(root->left);
        }
        return 1 + std::min(minDepth(root->right), minDepth(root->left));
    }
};


```


## STEP3
### 3回ミスなく書く
自然なBFSで
```cpp

#include <algorithm>
#include <vector>

class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        std::vector<TreeNode*> current_nodes;
        current_nodes.push_back(root);
        int depth = 1;
        while (true) {
            std::vector<TreeNode*> next_nodes;
            for (auto node : current_nodes) {
                if (!node->left && !node->right) {
                    return depth;
                }
                if (node->left) {
                    next_nodes.push_back(node->left);
                }
                if (node->right) {
                    next_nodes.push_back(node->right);
                }
            }
            depth++;
            std::swap(current_nodes, next_nodes);
        }
    }
};


```

5分,4分,4分で3回Accept

#### 2週目の宿題

- 非再帰DFSでも実装する
- 例外処理の方法についても、何を想定しているか、どんな形で処理するのか、等々、他のDiscordを見たり、自分調べたりして学ぶ
