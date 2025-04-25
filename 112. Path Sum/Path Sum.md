# 112. Path Sum
https://leetcode.com/problems/path-sum/description/

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- その時点での合計の数値を持ってBFSをする。
- 葉ノードでかつ条件を満たすならtrueを返して終了、queueが回り切ったらfalse


計算量
- 時間計算量 
    - 頂点＋辺の数で3N N = 5000の時1.5 * 10^4ステップ、15マイクロ秒と見積もり

- 空間計算量
    - O(N) 一番深い階層で約N/2個のノードが入るため

BFS
```cpp
#include <queue>

class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) {
            return false;
        }
        // pair = (node, total_value)
        std::queue<std::pair<TreeNode*, int>> node_to_visit;
        node_to_visit.push({root, 0});
        while (!node_to_visit.empty()) {
            auto [node, total] = node_to_visit.front();
            node_to_visit.pop();
            if (!node) {
                continue;
            }
            total += node->val;
            if (IsLeaf(node) && total == targetSum) {
                return true;
            }
            node_to_visit.push({node->left, total});
            node_to_visit.push({node->right, total});
        }
        return false;
    }
private:
    bool IsLeaf(TreeNode* node) {
        return !node->left && !node->right;
    }
};
```

DFSでも解いてみた
- 葉ノードでかつ、totalがtargetならtrueを返す
- 葉以外のノードは、右か左からtrueが返ってくるならtrueを返す
    - どちらもfalseならfalseを返す 

- 時間計算量O(N),　空間計算量O(N) (最悪ケース)

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) {
            return false;
        }
        return HasPathSumHelper(root, targetSum, 0);
    }
private:
    bool HasPathSumHelper(TreeNode* node, int targetSum, int total) {
        if (!node) {
            return false;
        }
        total += node->val;
        if (IsLeaf(node) && total == targetSum) {
            return true;
        }
        return HasPathSumHelper(node->left, targetSum, total) ||
               HasPathSumHelper(node->right, targetSum, total);
    }
    bool IsLeaf(TreeNode* node) {
        return !node->left && !node->right;
    }
};
```


## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/tarinaihitori/leetcode/pull/25/files
- https://github.com/Fuminiton/LeetCode/pull/25/files
- https://github.com/fuga-98/arai60/pull/25/files
- https://github.com/ryoooooory/LeetCode/pull/28/files
- https://github.com/t0hsumi/leetcode/pull/25/files


- ドキュメント系

teachers' eye
- 責任範囲 https://github.com/Fuminiton/LeetCode/pull/25/files#r2007901179
- 一回しか使わない短い処理を関数にする意味 https://github.com/Fuminiton/LeetCode/pull/25/files#r2018169618
    - 自分は操作を抽象化して表現できるので作ってもいいかなと思っています
- 演算子を手前に持ってくる　https://github.com/Fuminiton/LeetCode/pull/25/files#r2009103687
- 意味的に変わってほしくない変数 https://github.com/ryoooooory/LeetCode/pull/28/files#r1986257063
- 再帰の感覚、1000は大丈夫、10000はまずそう https://github.com/ryoooooory/LeetCode/pull/28/files#r1986234552


#### 感想
- stackの変数名はnodes_and_totalsがいいか
- "||"のような記号の後に改行するよりか、その前で改行した方が見やすいのか https://peps.python.org/pep-0008/#should-a-line-break-before-or-after-a-binary-operator
    - 下が揃っていた方が見やすいという人もいるのかな
- totalを加算するタイミング
    - 取り出した時にすでに加算されているのが違和感だったので取り出してから足すようにしたが、「nodeについてのこと」のまとまりという解釈がある https://github.com/Fuminiton/LeetCode/pull/25/files#r2007901179
    - 逆に取り出してから加算することに違和感がある方もいるようだ。好みの問題なのかも
    - けど追加する時点で足しておくと、いちいち追加前にnullでないか確認するので煩雑だなあという気持ち
    - 自分が採用するなら取り出した後で加算

- 再帰上限について勘違いしていた。1Mというは100万回でなく、スタックメモリが1Mということだった。 ちゃんとコメントを読んでいなかった。
    - https://discord.com/channels/1084280443945353267/1235829049511903273/1236256946403807323
    - https://tech.tinybetter.com/Article/f157609c-b43c-6ad1-4637-3a016608c7ab/View
    - スタックフレーム
        - https://learn.microsoft.com/ja-jp/cpp/mfc/memory-management-frame-allocation?view=msvc-170
        > スタック フレームは、関数のローカルに定義された変数や関数への引数を一時的に保持するメモリ領域です。


#### STEP1以外の手法と感想
- 基本的にBFSかDFSになっている
- N = 5000だと、スタックフレームを考えると再帰は微妙、だけど引数がintとポインタなので大丈夫か
    - leetcodeは64bit環境（sizeof(int*）が8だった)みたいなので
    - ListNode* が8byte intが4byte * 2 の計16byte,戻りアドレスも8byteで24byteと見積もり
    - 30byteと概算して スタックメモリ1MBとして30000回は可能？
- でもスタックオーバーフローが一応心配だからBFSにしよう。




再帰で、totalをあらかじめ足しておく方法
```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) {
            return false;
        }
        return HasPathSumHelper(root, targetSum, root->val);
    }
private:
    bool HasPathSumHelper(TreeNode* node, int targetSum, int total) {
        if (IsLeaf(node) && total == targetSum) {
            return true;
        }
        if (node->left && HasPathSumHelper(node->left, targetSum, total + node->left->val)) {
            return true;
        }
        if (node->right && HasPathSumHelper(node->right, targetSum, total + node->right->val)) {
            return true;
        }
        return false;
    }
    bool IsLeaf(TreeNode* node) {
        return !node->left && !node->right;
    }
};
```

## STEP3
### 3回ミスなく書く
BFSで

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        std::queue<std::pair<TreeNode*, int>> nodes_and_totals;
        nodes_and_totals.emplace(root, 0);
        while (!nodes_and_totals.empty()) {
            auto [node, total] = nodes_and_totals.front();
            nodes_and_totals.pop();
            if (!node) {
                continue;
            }
            total += node->val;
            if (IsLeaf(node) && total == targetSum) {
                return true;
            }
            nodes_and_totals.emplace(node->left, total);
            nodes_and_totals.emplace(node->right, total);
        }
        return false;
    }
private:
    bool IsLeaf(TreeNode* node) {
        return !node->left && !node->right;
    }
};
```

4分,4分,3分で3回Accept

#### ２周目の宿題
- 末尾再帰について調べる
- コンパイラによる最適化について調べる
