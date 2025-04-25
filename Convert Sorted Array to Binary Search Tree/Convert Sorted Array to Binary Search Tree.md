# 108. Convert Sorted Array to Binary Search Tree

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 右、左の区間の中央の値をどんどんとってノードに再帰的に書き込んでいく
- 再帰関数を書くのが自然に思う
- left, rightを再帰の引数にして、今の根の右左に応じて、新しいleft rightを付けて再帰
- left と rightの差が1の時は右にだけ木が生えて終了


計算量
- 時間計算量
    - 1 + 2 + 4 +.....Nになるので項数が約logN,初項1 公比2の等比数列の和でO(N)
    - というか、各ノード分をO(1)で作るので普通にO(N)
    - N = 10^4の時、10マイクロ秒と見積もり

- 空間計算量 
    - 木の階層分 ワーストケースでO(N)

```cpp
#include <vector>

class Solution {
public:
    TreeNode* sortedArrayToBST(const std::vector<int>& nums) {
        return SortedArrayToBSTHelper(nums, 0, nums.size() - 1);
    }
private:
    TreeNode* SortedArrayToBSTHelper(const std::vector<int>& nums, int left, int right) {
        int mid = (left + right) / 2;
        auto node = new TreeNode(nums[mid]);
        if (left == right) {
            return node;
        }
        if (left +1 == right) {
            node->left = nullptr;
            node->right = new TreeNode(nums[right]);
            return node;
        }
        node->left = SortedArrayToBSTHelper(nums, left, mid -1);
        node->right = SortedArrayToBSTHelper(nums, mid + 1, right);
        return node;
    }
};
```

終わった後、終了条件をleft > rightに統一できると気づいた
```cpp
#include <vector>

class Solution {
public:
    TreeNode* sortedArrayToBST(const std::vector<int>& nums) {
        return SortedArrayToBSTHelper(nums, 0, nums.size() - 1);
    }
private:
    TreeNode* SortedArrayToBSTHelper(const std::vector<int>& nums, int left, int right) {
        if (left > right) {
            return nullptr;
        }
        int mid = (left + right) / 2;
        auto node = new TreeNode(nums[mid]);
        node->left = SortedArrayToBSTHelper(nums, left, mid -1);
        node->right = SortedArrayToBSTHelper(nums, mid + 1, right);
        return node;
    }
};
```



## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/ichika0615/arai60/pull/17/files
- https://github.com/tarinaihitori/leetcode/pull/24/files  
- https://github.com/Fuminiton/LeetCode/pull/24/files
- https://github.com/fuga-98/arai60/pull/24/files


- ドキュメント系

teachers' eye
- 無駄な変数や引数がないか（引数にしなくても情報が取れるパターンもある） https://github.com/ichika0615/arai60/pull/17/files#r2019815076
- 「手の運動」 https://github.com/ichika0615/arai60/pull/17/files#r2019099632
    - とりあえず整理のために紙に書いてみるとか、手計算してみる、実験してみるみたいなニュアンスと思った
- while true で縦長のコードだと読み手が辛い　https://github.com/ichika0615/arai60/pull/17/files#r2020991639
    - 「コードの整え方」、まだあんまり腹落ちできてないので、定期的に読み返そう
- 動くことを考えるために、考慮する要素　https://github.com/Fuminiton/LeetCode/pull/24/files#r2004001700
    - これはかなり自分に不足しているなと感じる、「なんか動いた」の域から脱せていない
    - これがないと人のコードも「読めない」
    - 今回だと、閉区間で
        - 全てのインデックスはカバーされる
        - left == rightのときはそのindexを返せばいい
            - left == rightから更に進むと、必ずleftとrightが逆転するのでnullptrを返すのは合っている
        - left = right = 0や右端の時も問題ない
        - 木が深くなればleft-rightは近づき、いつか必ず逆転するので無限ループはない
        後付けでもこれくらいしか考えつかない


#### 感想
-　indexのleftとTreeNodeのleftが紛らわしいというのは確かにと思ったleft_indexのようにする
-　非再帰で、「入れる前に有効か確認」式の人が多いのかな、と感じた。自分もそうなのだが、とりあえず入れて取り出した時にダメなら捨てる方式の方がすっきりする場合があるので、step2でこの方式を試そう


#### STEP1以外の手法と感想
-　手法的には、再帰かstackを使う人が多かった。
    - 再帰の引数としてスライスした配列を持つ方法もあるが、これはindexをもてば良いのであえて採用する理由はないか
-　再帰の上限は、今回の制約では問題ない
-　実際のユースケース、何かあるだろうか
    - 「データ構造（平衡二分木とか）の元」に使われるイメージ
- 再帰のが自然で良いと感じた


- BST・「POPした時に有効か判定」式でダブルポインタを利用して。
- ダブルポインタの認知負荷はあるが、コード自体は場合分けがほぼなくすっきりする
```cpp
#include <queue>
#include <vector>

class Solution {
public:
    TreeNode* sortedArrayToBST(const std::vector<int>& nums) {
        //tuple = (left_index, right_index, ptr_to_node)
        std::queue<std::tuple<int, int, TreeNode**>> nodes_for_BST;
        TreeNode* root = new TreeNode();
        nodes_for_BST.emplace(0, nums.size() - 1, &root);
        while (!nodes_for_BST.empty()) {
            auto [left_index, right_index, ptr_to_node] = nodes_for_BST.front();
            nodes_for_BST.pop();
            if (!(left_index <= right_index)) {
                *ptr_to_node = nullptr;
                continue;
            }
            int mid_index = (left_index + right_index) / 2;
            (*ptr_to_node)->val = nums[mid_index];
            (*ptr_to_node)->left = new TreeNode();
            nodes_for_BST.emplace(left_index, mid_index - 1, &((*ptr_to_node)->left));
            (*ptr_to_node)->right = new TreeNode();
            nodes_for_BST.emplace(mid_index + 1, right_index, &((*ptr_to_node)->right));
        }
        return root;
    }
};
```

## STEP3
### 3回ミスなく書く

```cpp
#include <vector>

class Solution {
public:
    TreeNode* sortedArrayToBST(const std::vector<int>& nums) {
        return ContructBST(nums, 0, nums.size() - 1);
    }
private:
    TreeNode* ContructBST(const std::vector<int>& nums, int left_index, int right_index) {
        if (!(left_index <= right_index)) {
            return nullptr;
        }
        int mid_index = (left_index + right_index) / 2;
        TreeNode* node = new TreeNode(nums[mid_index]);
        node->left = ContructBST(nums, left_index, mid_index - 1);
        node->right = ContructBST(nums, mid_index + 1, right_index);
        return node;
    }
};
```

5分,4分,3分で3回Accept

#### ２周目の宿題
- 「コードの整え方」を再度見返してみる
