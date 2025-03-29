# 206. Reverse Linked List

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 繋ぎ変える解法
    - headの前にダミーのnullptrを置く
    - 使う変数は、prev,current,nextの3つ
    - current_nodeをheadから走査し、current->nextをprevに繋ぎ変える
- stackを用いる方法や、再帰による方法もありそうなので、過去の取り組みの復習を兼ねてSTEP1で実装
- stackを使う方法
    - 最後の1つ以外を順番にstackに入れる
    - stackから順に取り出し、逆につなげていく
    - 最後をnullptrに繋げる（stackの最初にnullptrを入れる方法もあるが、読み手が分かりにくいと思った）
- 再帰を使う方法
    - ループを再帰に直すのは機械的なので、別のアプローチ
    - 引き継ぐのは、ひっくり返すリストの先頭と、今日のノード繋げるための、ひっくり返している最中のリストの末尾
    - 最後のノードに来たら、ひっくり返すリストの先頭と末尾をそのノードに設定
    - それ以降は帰りがけで、今のノードをtailにつなげていく
    - 最後にtailをnullptrにつなげる

計算量
- 時間計算量
    - いずれも全てO(N)だが、one-wayの分、stackや再帰（行きがけと帰りがけがある）を使う方法よりも繋ぎ変えの方が早い
- 空間計算量
    - stackと再帰はO(N),繋ぎ変えはO(1)

繋ぎ変える方法
```cpp
class Solution {
 public:
  ListNode* reverseList(ListNode* head) {
    ListNode* dummy_node = nullptr;
    auto prev_node = dummy_node;
    auto current_node = head;
    while (current_node) {
      auto next_node = current_node->next;
      current_node->next = prev_node;
      prev_node = current_node;
      current_node = next_node;
    }
    return prev_node;
  }
};
```
stackを用いる
```cpp
class Solution {
 public:
  ListNode* reverseList(ListNode* head) {
    stack<ListNode*> node_stack;
    auto node = head;
    if (!head) {
      return nullptr;
    }
    // 最後の1つ以外を取り出す
    while (node->next) {
      node_stack.emplace(node);
      node = node->next;
    }
    auto reverse_node_head = node;
    auto reverse_node = node;
    while (!node_stack.empty()) {
      auto reverse_next_node = node_stack.top();
      reverse_node->next = reverse_next_node;
      reverse_node = reverse_next_node;
      node_stack.pop();
    }
    reverse_node->next = nullptr;
    return reverse_node_head;
  }
};
```
再帰を使った解法
```cpp
class Solution {
 public:
  ListNode* reverseList(ListNode* head) {
    if (!head) {
      return head;
    }
    ListNode* reverse_list_head = nullptr;
    ListNode* reverse_list_tail = nullptr;
    reverseListHelper(head, reverse_list_head, reverse_list_tail);
    reverse_list_tail->next = nullptr;
    return reverse_list_head;
  }

 private:
  static void reverseListHelper(ListNode* node, ListNode*& reverse_list_head,
                                ListNode*& reverse_list_tail) {
    //最後のノードに来たらreverse_listのheadとtailを設定
    if (!node->next) {
      reverse_list_head = node;
      reverse_list_tail = node;
      return;
    }
    reverseListHelper(node->next, reverse_list_head, reverse_list_tail);
    //帰りがけの処理(reverse_list_tailに今のノードを繋げる)
    reverse_list_tail->next = node;
    reverse_list_tail = node;
  }
};
```
## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの
- https://github.com/yus-yus/leetcode/pull/7/files
- https://github.com/quinn-sasha/leetcode/pull/7/files
- https://github.com/Fuminiton/LeetCode/pull/7/files
- https://github.com/mura0086/arai60/pull/12/files
- https://github.com/HitoshiKoba/Arai60-public/pull/3/files

- コメント集の本題の部分



#### 感想
stackを使った方法について
- node_stack.top() = reverse_next_node と置かなくても、reverse_node->next = node_stack.top()くらいなら大丈夫そう
- 1度全てstackに入れて、dummyを作ってからつなげていくのが、直感的に分かりやすい
- stackに入れるとき、->nextは全てnullにしておく、つまり最後のループでは「取り出すノードは何にもつながっていない」を約束すると、最後の処理が不要(再帰でも、常に末尾をnullptrにしておけばよい)

繋ぎ変えをする方法
- ListNode* dummy_node = nullptr; auto prev_node = dummy_node;　は二度手間。dummy_nodeは今後参照しないので不要
- prev_nodeという名前のノードを操作の結果として返すのは少し違和感がある
- reversed_list_headという名前にするか

再帰をする方法
- void型である必要がない、ListNode* 型でreverse_nodeの先頭を返せばよい

繋ぎ変え
```cpp
class Solution {
 public:
  ListNode* reverseList(ListNode* head) {
    ListNode* reversed_list_head = nullptr;
    auto node = head;
    while (node) {
      auto next_node = node->next;
      node->next = reversed_list_head;
      reversed_list_head = node;
      node = next_node;
    }
    return reversed_list_head;
  }
};

```
stack
```cpp
class Solution {
 public:
  ListNode* reverseList(ListNode* head) {
    if (!head) {
      return head;
    }
    stack<ListNode*> node_stack;
    auto node = head;
    while (node->next) {
      auto next_node = node->next;
      node->next = nullptr;
      node_stack.emplace(node);
      node = next_node;
    }
    auto reverse_list_head = node;
    auto reverse_list_tail = node;
    while (!node_stack.empty()) {
      reverse_list_tail->next = node_stack.top();
      node_stack.pop();
      reverse_list_tail = reverse_list_tail->next;
    }
    return reverse_list_head;
  }
};
```
再帰
```cpp
class Solution {
 public:
  ListNode* reverseList(ListNode* head) {
    if (!head) {
      return head;
    }
    ListNode* reverse_list_head = nullptr;
    ListNode* reverse_list_tail = nullptr;
    return reverseListHelper(head, reverse_list_head, reverse_list_tail);
  }

 private:
  static ListNode* reverseListHelper(ListNode* node,
                                     ListNode*& reverse_list_head,
                                     ListNode*& reverse_list_tail) {
    //最後のノードに来たらreverse_listのheadとtailを設定
    if (!node->next) {
      reverse_list_head = node;
      reverse_list_tail = node;
      return reverse_list_head;
    }
    reverse_list_head =
        reverseListHelper(node->next, reverse_list_head, reverse_list_tail);
    //帰りがけの処理(reverse_list_tailに今のノードを繋げる)
    reverse_list_tail->next = node;
    reverse_list_tail = node;
    reverse_list_tail->next = nullptr;
    return reverse_list_head;
  }
};
```

## STEP3
### 3回ミスなく書く
繋ぎ変えで
```cpp
class Solution {
 public:
  ListNode* reverseList(ListNode* head) {
    ListNode* reverse_list_head = nullptr;
    auto node = head;
    while (node) {
      auto next_node = node->next;
      node->next = reverse_list_head;
      reverse_list_head = node;
      node = next_node;
    }
    return reverse_list_head;
  }
};
```

9分で3回Accept
