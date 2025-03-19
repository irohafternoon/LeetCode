# 83. Remove Dulpicates from Sorted List

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- ノードを見た後、setに今あるノードに格納された数字をメモしておく
- 今あるノードの次の行先を調べるため、2重のwhileループを回す
- 次のノードを調べるための変数 next_nodeを作成(必要ではないが、current->next->nextのよう名表記が無いほうが読みやすいとだろう思って導入した)
- 「次のノードに格納されている値がすでに見た値なら、次のノードになれないので、さらに１つ先を見る
- 新しい数字のノードが現れたら、current_nodeの行先を決定し、ノードを進める

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
#include <set>

class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    std::set<int> seen_numbers;
    ListNode* current_node = head;
    while (current_node) {
      seen_numbers.insert(current_node->val);
      ListNode* next_node = current_node->next;
      // next_nodeが見たことのない数字になるまで
      while (next_node && seen_numbers.contains(next_node->val)) {
        next_node = next_node->next;
      }
      current_node->next = next_node;
      current_node = next_node;
    }
    return head;
  }
};
```
## STEP2
### プルリクやドキュメントを参照
### 問題が解けた云々より他人のコードを読んだりコメントをするほうがよっぽど大事
#### 参照したもの
- https://github.com/shintaro1993/arai60/pull/6
- https://github.com/5ky7/arai60/pull/4
- https://github.com/myzn0806/leetcode-2/pull/3
- https://github.com/hayashi-ay/leetcode/pull/20
- https://github.com/TORUS0818/leetcode/pull/5/files (再帰で解いている)
- https://github.com/mura0086/arai60/pull/9/files
- https://github.com/fuga-98/arai60/pull/4 

#### 感想等
- 他人のコードを読み解くのは大変（思考過程コメントがあっても大変）
- ループは1回で十分(結局ポインタを進めるのには変わらないため)
- ただ、2重ループだからと言って可読性が低いとは限らない
- while() の()の部分が長いのは読みにくいかも
    - ただ、while以下の処理が短いなら、多少長くても良い気がする
- 入力がsortされているなら、今回のようにsetを用意する必要はない
- continueを使ったelseの省略　読みやすさと自分がコードを書く際の書きやすさも向上しそう
- 重複を飛ばしたり、重複を判定する操作をいちいち関数にするのは冗長過ぎる
- 再帰で書くのはさすがに可読性が低いと思うが、他人のコードを読み解く練習になった
- 

#### 2重ループ　2変数
```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    ListNode* current_node = head;
    while (current_node) {
      ListNode* next_node = current_node->next;
      while (next_node && next_node->val == current_node->val) {
        current_node->next = next_node->next;
        next_node = next_node->next;
      }
      current_node = current_node->next;
    }
    return head;
  }
};
```

#### 単ループ 2変数
```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    ListNode* current_node = head;
    ListNode* next_node = head;
    while (current_node && next_node) {
      if (current_node->val == next_node->val) {
        current_node->next = next_node->next;
        next_node = next_node->next;
        continue;
      }
      current_node = current_node->next;
    }
    return head;
  }
};
```
2変数だと2つのノードを進める操作を記述する必要があるので、1変数のほうがすっきりしそう

#### 2重ループ　1変数
```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    ListNode* node = head;
    while (current_node) {
      while (node->next && node->val == node->next->val) {
        node->next = node->next->next;
      }
      node = node->next;
    }
    return head;
  }
};
```

#### 単ループ 1変数
```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    ListNode* node = head;
    while (node && node->next) {
      if (node->val == node->next->val) {
        node->next = node->next->next;
        continue;
      }
      node = node->next;
    }
    return head;
  }
};
```
これくらいの処理なら、単ループ・1変数が一見やすく、可読性もあるのではないか

#### 再帰関数の実装（おまけ）
- 関数fの内容
  - 入力されたリストを「"先頭"と"それ以降"に分ける」
  - "先頭"と"それ以降"の先頭（＝2番目）が同じならば、"先頭"は不要なので、f("それ以降")を新たな先頭とする
  - "先頭"と"それ以降"の先頭（＝2番目）が異なるなら、先頭の次にはf("それ以降")がつながる

```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    if (!head->next) {
      return head;
    }
    if (head->val == head->next->val) {
      head = deleteDuplicates(head->next);
    }
    else{
      head->next = deleteDuplicates(head->next);
    }
    return head;
  }
};
```

#### STEP3
変数は１つで　単ループと2重ループで

単ループ
```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    ListNode* node = head;
    while (node && node->next) {
      if (node->val == node->next->val) {
        node->next = node->next->next;
        continue;
      }
      node = node->next;
    }
    return head;
  }
};
```
2重ループ
```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    ListNode* node = head;
    while (node) {
      while (node->next && node->val == node->next->val) {
        node->next = node->next->next;
      }
      node = node->next;
    }
    return head;
  }
};
```
8分で3回AC
