# 141. Linked List Cycle

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 一度調べたノードはsetにメモしておく
- もし今見ているノードがすでに見たものであれば、tureを返し終了
- 何もなければfalseを返す
- 時間計算量・空間計算量ともにO(N)

#### 過程
- 入出力の形式について、AtCoderのものしか見たことがなく、戸惑った
- 最終的にLinked Listのheadが渡されるのだと納得した
- 入力として与えられる数字は本当は「リストに格納される数字」であるが、「ノード番号」と勘違いした
- よく考えたらポインタのNULLとは何？という疑問が生じた
- setの型を" ListNode* " が正しいところ "ListNode" にしてしまうミス
 
```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
#include <set>
class Solution {
public:
    bool hasCycle(ListNode *head) {
        std::set<ListNode*> visited;
        while (! head == NULL){
            if (visited.count(head)) return true;
            visited.insert(head);
            head = head->next;
        }
    return false;
    }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 参照したもの
- https://discord.com/channels/1084280443945353267/1239148130679783424/1348986745844142202
- https://discord.com/channels/1084280443945353267/1239148130679783424/1346058253104185438
- https://discord.com/channels/1084280443945353267/1239148130679783424/1343976322837385228
- https://discord.com/channels/1084280443945353267/1239148130679783424/1340251944358248578
- https://discord.com/channels/1084280443945353267/1239148130679783424/1338425356427989004

- コメント集の本題の部分

- ドキュメント系
    - set
    https://cpprefjp.github.io/reference/set/set.html
    contains() で所属判定がc++20から可能
    - NULL
    https://cpprefjp.github.io/reference/cstddef/null.html
    C++11以降はnullptrを使用すべき
    - nullptr
    https://cpprefjp.github.io/lang/cpp11/nullptr.html
    自分の理解としてはポインタにおけるNULL,nullptrとは「メモリアドレスの0番地」だが、正しいのだろうか

#### 感想
- 変数名関連
    - 「head」は与えられたリストの先頭という意味があるため、headでない別の変数を持って更新したほうがいい
    - current という名前がよさそう
    - setの「Visited」も「Visited_Nodes」にした方が良いかもしれない
- 「ぶらさがり文」は避けたほうが良い
- "while (! head == NULL)" は " while(head) " の部分は不要か
- Formatterの存在を知った。構文エラーや統一されていないインデント等の見栄えを修正してくれるもの
    -Formatterによって"hasCycle(ListNode \*head)"は"hasCycle(ListNode\* head)"に\*の位置を修正
- フロイドの方法というものがあるらしいが、あまりこだわる必要はないようだ

```cpp
#include<set>

struct Solution{
public:
    bool hasCycle(ListNode* head){
        std::set<ListNode*> Visited_Nodes;
        ListNode* current = head;
        while (current){
            if (Visited_Nodes.contains(current)){
                return true;
            }
            Visited_Nodes.insert(current);
            current = current->next;
        }
        return false;
    }
};
```

## STEP3
### 3回ミスなく書く

```cpp
# include<set>

struct Solution{
public:
    bool hasCycle(ListNode* head){
        std::set<ListNode*> Visited_Nodes;
        ListNode* current = head;
        while(current){
            if(Visited_Nodes.contains(current)){
                return true;
            }
            Visited_Nodes.insert(current);
            current = current->next;
        }
        return false;
    }
};
```

8分で3回Accept
