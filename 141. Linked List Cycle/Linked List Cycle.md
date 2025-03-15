# 141. Linked List Cycle

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 一度調べたノードはsetにメモしておく
- もし今見ているノードがすでに見たものであれば、tureを返し終了
- 何もなければfalseを返す

#### 過程
- 入出力の形式について、AtCoderのものしか見たことがなく、戸惑った
- 最終的にLinked Listのheadが渡されるのだと納得した
- 入力として与えられる数字は本当は「リストに格納される数字」であるが、「ノード番号」と勘違いした
- よく考えたらポインタのNULLとは何？という疑問が生じた

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



