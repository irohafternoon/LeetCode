# 82. Remove Dulpicates from Sorted List II

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- おそらく与えられたリストを繋ぎ変えて答えを出すのだろうが、やり方がわからない
- しょうがないのでリストを一周して値をメモし、１回しか登場していないもので新しくListを作り直す解法
- LinkListを新しく作る方法を知らなかったので生成AIに聞くなどしてとりあえず作成
- この方法は入力を破壊しない(する方法も思いついてどちらのメリット・デメリットを考えられるのが理想)
- 関数内の処理ではないがこの方法だとメモリ開放が必要

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
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    if (!head) {
      return nullptr;
    }
    // 各値の出現回数をカウント
    map<int, int> number_count;
    ListNode* cur = head;
    while (cur) {
      number_count[cur->val]++;
      cur = cur->next;
    }
    // 出現回数が1回のみの値を set に追加
    set<int> unique_nums;
    for (auto [key, value] : number_count) {
      if (value == 1) {
        unique_nums.insert(key);
      }
    }
    // 新しいリンクリストの作成
    ListNode* new_head = nullptr;
    ListNode* new_tail = nullptr;
    for (auto value : unique_nums) {
      // 初回だけはheadを更新
      if (!new_head) {
        new_head = new ListNode(value);
        new_tail = new_head;
        continue;
      } 
      // 初回以外はtailを伸ばしていく
      new_tail->next = new ListNode(value);
      new_tail = new_tail->next;
    }
    return new_head;
  }
};

//＊注意：関数を呼び出した際にメモリの開放が必要
//examle//
ListNode* new_list = Solution.deleteDuplicates(head);
ListNode* node = new_list;
while (node) {
  ListNode* gabage = node;
  node = node->next;
  delete (gabage);
}
```

## STEP2
### プルリクやドキュメントを参照
### 問題が解けた云々より他人のコードを読んだりコメントをするほうがよっぽど大事
#### 参照したもの
- https://github.com/katataku/leetcode/pull/3#discussion_r1836843562
- https://github.com/fuga-98/arai60/pull/5
- https://github.com/plushn/SWE-Arai60/pull/4
- https://github.com/Fuminiton/LeetCode/pull/4
- https://github.com/quinn-sasha/leetcode/pull/4
- https://github.com/yus-yus/leetcode/pull/4
- https://github.com/lilnoahhh/leetcode/pull/4
- https://github.com/konnysh/arai60/pull/4
- https://github.com/t0hsumi/leetcode/pull/4#discussion_r1856455749

#### 「引継ぎ」について
- https://discord.com/channels/1084280443945353267/1322513618217996338/1327647773918625863 
- https://github.com/seal-azarashi/leetcode/pull/38#discussion_r1837422519
- https://discord.com/channels/1084280443945353267/1195700948786491403/1197102971977211966 (列車)
- https://discord.com/channels/1084280443945353267/1231966485610758196/1239417493211320382 (鎖)

####　感想等
- 2種類か3種類の点を考慮する必要があり、コードを読むのが難しくなりやすい
- A = B->next　のような場合、がB->nextが直接Cを指すと分かるなら A = Cとするべき
- 条件分けをすれば、dummyも不要
    - dummyが必要なのは、headが答えの先頭になるとは限らないため
    - ただ、「headを重複がないノードとなるように前処理する」がほぼこの問題を解くようなものなので、二度手間のように感じる
- 「リストの最後の値」をuniqueやnodeという変数で管理しているが、追加するか未確定の状態でunique->nextを更新するコードは、個人的には混乱した（->nextは「候補」として更新、という制御をするコード）
- 見たコードの大半がこの方法なので、自分の感覚を修正したほうがいいと思った。
- この方法では最後にnullptrを加えるというステップを省略できる
- なぜなら、どの処理も仮候補であるunique->nextは最終的にnullに更新され、その後ループが回らないため



#### 自分の中で固めた「引継ぎ」のイメージ
- 翌日に引き継がれる情報は「確定したリストの最後」 = last_unique_node と「明日調査を開始するノード」　= node
- 仕事内容 
  - 「今日調査するノード」としてnullが渡された人は最後の締めとして、last_unique_nodeの次をnullにして、全体の仕事を終了

  - 今日開始するノードに格納されている数字 = target_value を記録する
  - 次のノードがnull もしくは　次のノードの数字が記録した数字と違うなら
    - 今日の開始ノードをリストの末尾に繋ぐ
    - 引継ぎとして明日の開始ノードを更新して終了
  - 次のノードの数字が今日の記録した数字と違うなら
    - 今日記録した数字と同じ数字が続く限りノードを進める
    - 進めた先がそのまま明日の開始ノード
  
仕事イメージに忠実なパターン
```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    ListNode dummy(0, head);
    ListNode* last_unique_node = &dummy;
    ListNode* node = head;
    while (true) {
      if (!node) {
        last_unique_node->next = nullptr;
        break;
      }
      int target_value = node->val;
      if (!node->next || node->next->val != target_value) {
        last_unique_node->next = node;
        last_unique_node = node;
        node = node->next;
        continue;
      }
      while (node && node->val == target_value) {
        node = node->next;
      }
    }
    return dummy.next;
  }
};
```
自分の手なりで書くと、無限ループを使わず、下記のように最後にnullptrを付け足すが、意味的には上記が分かりやすい
```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    ListNode dummy(0, head);
    ListNode* last_unique_node = &dummy;
    ListNode* node = head;
    while (node) {
      int target_value = node->val;
　　　 //省略//
    }
    last_unique_node->next = nullptr;
    return dummy.next;
  }
};
```
最後の node = node->next の処理を共通化せず、if-continueを用いてそれぞれのパターンの処理を独立にしたほうが、意味が伝わりやすいだろうか、個人的には好みの範疇かな、と思う

#### 他のコードで主流の方法のイメージ
- 翌日に引き継がれる情報は、明日開始するノードであり「確定したリストの末尾の「候補」」 = last_unique_node->next(=nodeとして引き継がれる)
- 仕事内容 
  - 今日の開始ノードの次を見る
  - もし数が異なったら(null含む)、昨日の「候補」は今日確定に変わるので実際にリストにつなげる
  - もし数字が同じなら
    - null もしくは　開始ノードの数と異なる数が初めて出てくる場所(nullも含む)が新しい「候補」
  - どちらの場合も、最後に引継ぎとして「明日の開始ノード」を更新(node = node->next)して終了


```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    ListNode dummy;
    ListNode* last_unique_node = &dummy;
    last_unique_node->next = head;
    ListNode* node = head;
    while (node) {
      if (!node->next || node->val != node->next->val) {
        last_unique_node = node;
        node = node->next;
        continue;
      }
      while (node->next && node->next->val == node->val) {
        node = node->next;
      }
      last_unique_node->next = node->next;
      node = node->next;
    }
    return dummy.next;
  }
};
```
## STEP3
#### 3回ミスなく書く
仮更新方式の定着のため

```cpp
class Solution {
 public:
  ListNode* deleteDuplicates(ListNode* head) {
    ListNode dummy;
    ListNode* last_unique_node = &dummy;
    last_unique_node->next = head;
    ListNode* node = head;
    while (node) {
      if (!node->next || node->val != node->next->val) {
        last_unique_node = node;
        node = node->next;
        continue;
      }
      while (node->next && node->val == node->next->val) {
        node = node->next;
      }
      node = node->next;
      last_unique_node->next = node;
    }
    return dummy.next;
  }
};
```
9分で3回Accept


STEP4以降で再帰,stackでの解法を実装（備忘）
