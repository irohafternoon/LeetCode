# 2. Add Two Numbers
https://leetcode.com/problems/add-two-numbers/description/

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- Linked_listを筆算の要領で各桁の数字を足し、繰り上がりを管理する
- 各リストの長さが異なる場合は、ノードを進めたり足し算をする処理について考慮が必要
- ノードがnullの場合は数字を0とする
- ノードがnullでない限り、二つのノードを進める
- ループ後に繰り上がりが残っていたら、「1」の入ったノードを追加して終了


#### 過程
- 思考の通りに素直に書くと、addTwoNumbersが煩雑に
- ノードがnullかによって処理が変わる部分は、関数にしておくと良いと思い、①ノードを進める　②ノードの数字を取得する　の2つを切り出した

```cpp
//与えられるノードと関数で作成したリストのノードのメモリは、関数の呼び出し側で開放することを過程
class Solution {
 public:
  void increment_nodes(ListNode*& node_1, ListNode*& node_2) {
    if (node_1) {
      node_1 = node_1->next;
    }
    if (node_2) {
      node_2 = node_2->next;
    }
  }

  int get_node_value(ListNode* node) {
    if (node) {
      return node->val;
    }
    return 0;
  }

  ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode dummy;
    ListNode* last_answer_node = &dummy;
    ListNode* node_1 = l1;
    ListNode* node_2 = l2;
    bool carry = false;
    while (node_1 || node_2) {
      int value_1 = get_node_value(node_1);
      int value_2 = get_node_value(node_2);
      int new_node_value = value_1 + value_2 + carry;
      carry = false;
      if (new_node_value >= 10) {
        new_node_value %= 10;
        carry = true;
      }
      ListNode* new_node = new ListNode(new_node_value);
      last_answer_node->next = new_node;
      last_answer_node = new_node;
      increment_nodes(node_1, node_2);
    }
    if (carry) {
      ListNode* new_node = new ListNode(1);
      last_answer_node->next = new_node;
    }
    return dummy.next;
  }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの
- https://github.com/fuga-98/arai60/pull/6/files (再帰を使う方法)
- https://github.com/mura0086/arai60/pull/10/files (carryを処理に含める)
- https://github.com/plushn/SWE-Arai60/pull/5/files
- https://github.com/Fuminiton/LeetCode/pull/5/files
- https://github.com/quinn-sasha/leetcode/pull/5/files

- コメント集の本題の部分 (主にdummyの使用の有無と再帰)

#### 感想
- public と　private　そしてstatic
    - classについてあまり知識がないので、基本を調べた
    - ヘルパーの関数はクラスの外部でアクセスする必要はないので、privateのほうが良い
    - さらに、ヘルパー関数はクラスのメンバ変数に依存しないので、これを明示するためにもstatic void, static int　で宣言するべき
- while の条件に carryは含めることができるので、最後の処理をしないこともできる
- carryの型は intのほうがいいと思った（2つ以上足す場合は繰り上がりが2以上にもなるという拡張性もある）
- get_node_value 関数は、ポインタがnullの時に値0の番兵ListNodeを用意するのと意味は同じだが、番兵を用意するというやり方もある
- new_node_value　という変数よりtotalのほうがよさそう。totalの1の位をnew_nodeに格納するというイメージ
- このような問題で「再帰」という発想が全く出てこない。この処理に再帰を使うのはかなり発想が飛んでいると思っていた
- しかし、他の人のコードを読んで、要は再帰 = 無限ループ（終了条件を定義して、それ以外は同様の処理を繰り返す）だと理解して、再帰で解くのも自然だと納得した
- 再帰関数でかつ補助関数を2つ使うと、オーバーヘッドはどのくらいになるだろうと思った
    - 関数呼び出しのオーバーヘッドが10クロック程度のようなので、1ノードにつき30クロック = 3GHzのCPUで 10ナノ秒
    - 今回はノードの上限が100なので全く問題ない
    - ノードの数が10^6で10ミリ秒
    - 高速でこの計算を繰り返すことは想定しづらいので、このケースでは関数のオーバーヘッドは気にならないと思った
 

step1の修正
```cpp
class Solution {
 public:
  ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode dummy;
    ListNode* last_answer_node = &dummy;
    ListNode* node_1 = l1;
    ListNode* node_2 = l2;
    int carry = 0;
    while (node_1 || node_2 || carry) {
      int total = get_node_value(node_1) + get_node_value(node_2) + carry;
      ListNode* new_node = new ListNode(total % 10);
      carry = total / 10;
      last_answer_node->next = new_node;
      last_answer_node = new_node;
      increment_nodes(node_1, node_2);
    }
    return dummy.next;
  }

 private:
  static void increment_nodes(ListNode*& node_1, ListNode*& node_2) {
    if (node_1) {
      node_1 = node_1->next;
    }
    if (node_2) {
      node_2 = node_2->next;
    }
  }

  static int get_node_value(ListNode* node) {
    if (node) {
      return node->val;
    }
    return 0;
  }
};
```
 
dummyを使う再帰
```cpp
//increment_nodes関数とget_node_value関数は省略
class Solution {
 public:
  ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode dummy;
    ListNode* last_answer_node = &dummy;
    addTwoNumbers_helper(last_answer_node, l1, l2, 0);
    return dummy.next;
  }

 private:
  //2関数省略
  static void addTwoNumbers_helper(ListNode* last_answer_node, ListNode* node_1,
                                   ListNode* node_2, int carry) {
    if (!node_1 && !node_2 && !carry) {
      return;
    }
    int total = get_node_value(node_1) + get_node_value(node_2) + carry;
    ListNode* new_node = new ListNode(total % 10);
    last_answer_node->next = new_node;
    last_answer_node = new_node;
    increment_nodes(node_1, node_2);
    addTwoNumbers_helper(last_answer_node, node_1, node_2, total / 10);
  }
};
```
dummyを使わない再帰 (初回だけ別対応)
```cpp
//increment_nodes関数とget_node_value関数は省略
class Solution {
 public:
  ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode* head = nullptr;
    ListNode* tail = nullptr;
    addTwoNumbers_helper(head, tail, l1, l2, 0);
    return head;
  }

 private:
  //2関数省略
  static void addTwoNumbers_helper(ListNode*& head, ListNode*& tail, ListNode* node_1,
                                   ListNode* node_2, int carry) {
    if (!node_1 && !node_2 && !carry) {
      return;
    }
    int total = get_node_value(node_1) + get_node_value(node_2) + carry;
    ListNode* new_node = new ListNode(total % 10);
    if (! head){
        head = new_node;
        tail = new_node;
    }
    else{
        tail->next = new_node;
        tail = new_node;
    }
    increment_nodes(node_1, node_2);
    addTwoNumbers_helper(head, tail, node_1, node_2, total / 10);
  }
};
```


## STEP3
### 3回ミスなく書く
自分が一番自然と思ったループで
```cpp
class Solution {
 public:
  ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode dummy;
    ListNode* last_answer_node = &dummy;
    ListNode* node_1 = l1;
    ListNode* node_2 = l2;
    int carry = 0;
    while (node_1 || node_2 || carry) {
      int total = get_node_value(node_1) + get_node_value(node_2) + carry;
      ListNode* new_node = new ListNode(total % 10);
      carry = total / 10;
      last_answer_node->next = new_node;
      last_answer_node = new_node;
      increment_nodes(node_1, node_2);
    }
    return dummy.next;
  }

 private:
  static void increment_nodes(ListNode*& node_1, ListNode*& node_2) {
    if (node_1) {
      node_1 = node_1->next;
    }
    if (node_2) {
      node_2 = node_2->next;
    }
  }

  static int get_node_value(ListNode* node) {
    if (node) {
      return node->val;
    }
    return 0;
  }
};
```

25分で3回Accept

備忘　STEP4でstackを用いた再帰を書く
