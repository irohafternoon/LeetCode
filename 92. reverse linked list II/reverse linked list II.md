# 92. reverse linked list II

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- とりあえずleftまでノードを進める
- 最終的に、left-1がright、leftがright+1のノードつながるため、left-1,leftに名前をつけておく(left_joint,right_joint)
- nodeがrightになるまで進めながら、以下をする
  - 次のノードがrightになるまで、今と次のノードを逆につなぐ
  - 引き継ぎ必要な内容、①明日の開始ノード(node)②明日の開始ノードの次のノード(next_node)
  - ②を別に記録するのは、繋ぎ変える都合上、->nextで次のノードが取れないため
- 最後にひっくり返した部分をジョイントする
```cpp
class Solution {
 public:
  ListNode* reverseBetween(ListNode* head, int left, int right) {
    ListNode dummy(0, head);
    ListNode* prev_node = &dummy;
    ListNode* current_node = head;
    int index = 1;
    while (index < left) {
      current_node = current_node->next;
      prev_node = prev_node->next;
      index++;
    }
    // currentがleft prevがleft-1にある状態
    ListNode* left_joint = prev_node;
    ListNode* right_joint = left_joint->next;
    ListNode* next_node = current_node->next;
    while (index < right) {
      ListNode* two_next_node = next_node->next;
      next_node->next = current_node;
      current_node = next_node;
      next_node = two_next_node;
      index++;
    }
    // currentがright, nextがright+1(null含む)にある
    right_joint->next = next_node;
    left_joint->next = current_node;
    return dummy.next;
  }
};
```


## STEP2
### プルリクやドキュメントを参照
#### 参照したもの
類題のreverse linked list のPRから
- https://github.com/plushn/SWE-Arai60/pull/7/files
- https://github.com/mura0086/arai60/pull/12/files
- https://github.com/HitoshiKoba/Arai60-public/pull/3/files
- https://github.com/fuga-98/arai60/pull/8/files
- https://github.com/Fuminiton/LeetCode/pull/7/files

- chatgptに見せてみる
#### 感想
- どうせindexを数えるのだから、素直にfor文を回せばいい（ループを回す回数は、left,rightから計算可能）
- 次の次のノードというのは dummy_nodeとしているのが多い
- dummyは使っているので、temporary_node とかか
- 現在、次、次を次を見るよりか、一つ前、現在、次　で管理する人が多い　→　自分もそっちのほうが良さそう
- というか、最初のループで current,prevを見ていたのに、次フェーズでnext と　two_nextを持つのは整合性がない
- ただ、prev,current,nextで固定すると、切り替えのところで途中で1つ進めなくてはいけず、うまくいかない
- reverse linked listが再帰でかけるので、この問題も再帰でかけるか
    - 部分的に再帰でひっくり返そうだけど、ひっくり返した部分の接合まで含めると思い浮かばなかった
- 接着個所の名前はあまり納得言っていないので、ほかに解いた人がいれば気になる

とりあえず最初の方針をbrush-up
```cpp
class Solution {
 public:
  ListNode* reverseBetween(ListNode* head, int left, int right) {
    ListNode dummy(0, head);
    ListNode* prev_node = &dummy;
    ListNode* current_node = head;
    for (int idx = 1; idx < left; idx++) {
      current_node = current_node->next;
      prev_node = prev_node->next;
    }
    // currentがleft prevがleft-1にある状態
    // jointとなる部分を記録
    ListNode* joint_to_left = prev_node;
    ListNode* joint_to_right = current_node;
    ListNode* next_node = current_node->next;
    for (int idx = left; idx < right; idx++) {
      ListNode* temporary_node = next_node->next;
      next_node->next = current_node;
      current_node = next_node;
      next_node = temporary_node;
    }
    // currentがright, nextがright+1(null含む)にある
    joint_to_left->next = current_node;
    joint_to_right->next = next_node;
    return dummy.next;
  }
};
```

## STEP3
### 3回ミスなく書く

```cpp
class Solution {
 public:
  ListNode* reverseBetween(ListNode* head, int left, int right) {
    ListNode dummy(0, head);
    ListNode* prev_node = &dummy;
    ListNode* current_node = head;
    for (int idx = 1; idx < left; idx++) {
      current_node = current_node->next;
      prev_node = prev_node->next;
    }
    // current = left, prev = left-1
    ListNode* joint_to_left = prev_node;
    ListNode* joint_to_right = current_node;
    ListNode* next_node = current_node->next;
    for (int idx = left; idx < right; idx++) {
      ListNode* temporary_node = next_node->next;
      next_node->next = current_node;
      current_node = next_node;
      next_node = temporary_node;
    }
    joint_to_left->next = current_node;
    joint_to_right->next = next_node;
    return dummy.next;
  }
};
```

## STEP4
繋ぎ変えの3種類のうち、残りの2種類を実装

ひっくりかえす前の鎖と後の鎖を用意して、前のやつの先頭を後のやつの先頭につけていく方法

```cpp
class Solution {
 public:
  ListNode* reverseBetween(ListNode* head, int left, int right) {
    ListNode dummy(0, head);
    ListNode* last_node_before_reverse = &dummy;
    for (int i = 0; i < left - 1; i++) {
      last_node_before_reverse = last_node_before_reverse->next;
    }
    // last_node_before_reverse = left-1
    ListNode* head_of_reverse = last_node_before_reverse->next;
    last_node_before_reverse->next = nullptr;
    // last_node_before_reverse and head_of_reverse are fixed from now
    auto node_to_reverse = head_of_reverse;
    ListNode* tail_of_reverse = nullptr;
    // node_to_reverse should be set after tail_of_reverse
    for (int i = 0; i < right - left + 1; i++) {
      auto next_node_to_reverse = node_to_reverse->next;
      node_to_reverse->next = tail_of_reverse;
      tail_of_reverse = node_to_reverse;
      node_to_reverse = next_node_to_reverse;
    }
    last_node_before_reverse->next = tail_of_reverse;
    head_of_reverse->next = node_to_reverse;
    return dummy.next;
  }
};
```
先頭の前にダミーをつけて、先頭の次のノードをダミーの後ろに挿入していく方法

```cpp
class Solution {
 public:
  ListNode* reverseBetween(ListNode* head, int left, int right) {
    ListNode dummy(0, head);
    ListNode* last_node_before_reverse = &dummy;
    for (int i = 0; i < left - 1; i++) {
      last_node_before_reverse = last_node_before_reverse->next;
    }
    // last_node_before_reverse = left-1
    ListNode* head_of_reverse = last_node_before_reverse->next;
    // last_node_before_reverse and head_of_reverse are fixed from now
    ListNode* push_position = head_of_reverse;
    // node_to_push should be set before push_position
    for (int i = 0; i < right - left; i++) {
      auto node_to_push = head_of_reverse->next;
      auto next_node_to_push = node_to_push->next;
      last_node_before_reverse->next = node_to_push;
      node_to_push->next = push_position;
      push_position = node_to_push; 
      head_of_reverse->next = node_node_to_push;
    }
    return dummy.next;
  }
};
```
