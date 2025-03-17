# 142. Linked List Cycle II

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- ノードを見た後、setにメモしておく
- もし今見ているノードがすでに見たものであれば、その今見ているノードを返し終了
- 重複したノードがサイクルの始点であり終点であるため
- 何もなければfalseを返す
- 時間計算量・空間計算量ともにO(N)
- 「nodeがnullptrでない限り」はwhile (node) より while (node != nullptr)のほうが事故がなくて良いらしいので後者を採用してみる

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
 #include<map>

class Solution {
public:
    ListNode* detectCycle(ListNode* head) {
        int idx = 0;
        std::set<ListNode*> visited_nodes;
        ListNode* node = head;
        while (node != nullptr){
            if (visited_nodes.contains(node)){
                return node;
            }
            visited_nodes.insert(node);
            node = node->next;
        }
        return nullptr;
    }
};
```
前回の141. Linked List Cycleとほぼ同様のため、ほとんど時間はかからなかった

## STEP2
### プルリクやドキュメントを参照
#### 参照したもの
- https://github.com/shintaro1993/arai60/pull/5
- https://github.com/5ky7/arai60/pull/3
- https://github.com/myzn0806/leetcode-2/pull/2
- https://github.com/fuga-98/arai60/pull/3
- https://github.com/mura0086/arai60/pull/7

- 本問のコメント集(fast,slowを使った解法)

ドキュメント等
set と unordered_setの違いの話
- https://chromium.googlesource.com/chromium/src/+/HEAD/base/containers/README.md#std_unordered_map-and-std_unordered_set
これを読むのは「最終ゴール」ということだったが、現時点で見てもさっぱりわからない
これを「読むの大変そうだけど、まあ読むか」くらいにするにはどうすればいいのだろうか
- なんとなく低レイヤーというか、CSの基礎知識も必要そうな気がする
- 少なくとも、問題を解くやアルゴリズムを身に着けるといった方向とはまた別


#### fast slow を使った解法
#### 解法の整理（自然言語で説明）
- 1歩ずつ進むslow 2歩ずつ進むfastの二つを用意する
- 2つが同じノードに止まるか、片方がnullまで行くまで進める
- 同じノードに止まる場合
    - slowをスタート地点にもどして、fast,slowともに1つづつ進める
    - fast,slowが同じ場所に止まったところがサイクルの開始（＝分岐）地点

#### 苦労したポイントや、調べたこと
- fast == slowをいつ判定するか。whileループの初めにしてしまうと初期状態で止まってしまう
- ループの書き方。無限ループをとりあえず書いて終了条件でbreakしたほうが、個人的には考えやすい
- 無限ループを書くときはfastとfast->nextがnullであるか調べればよい
    - fast->next->nextは調べなくてよい
- 二回目のループは、最初に同じ場所にある場合も考慮するためにfast==slowの判定位置が最初のループと異なる
- slowをheadに戻すとき、別の変数を使うべきか→好みの範疇？
    - 個人的には「カメを戻す」という意味合いを込めて使い回そうかなと思った。
    - 3匹目の動物が出てくるわけでなないため

#### コードの整え方
- https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.9kpbwslvv3yv
- 今回のは「ループを止める条件」をとりあえず先に書いて、止めないなら・・・といった説明をするイメージ

```cpp
class Solution {
public:
    ListNode* detectCycle(ListNode* head) {
        ListNode* fast = head;
        ListNode* slow = head;
        while (true){
            if (fast == nullptr || fast->next == nullptr){
                return nullptr;
            }
            fast = fast->next->next;
            slow = slow->next;
            if (fast == slow){
                break;
            }
        }
        slow = head;
        while (true){
            if (fast == slow){
                return fast;
            }
            fast = fast->next;
            slow = slow->next;
        }
    }
};
```

## STEP3
### 3回ミスなく書く
自然に理解→暗記を実感するため、fast-slowのコードで3回

```cpp
class Solution{
public:
    ListNode* detectCycle(ListNode* head){
        ListNode* fast = head;
        ListNode* slow = head;
        while (true){
            if (fast == nullptr || fast->next == nullptr){
                return nullptr;
            }
            fast = fast->next->next;
            slow = slow->next;
            if (fast == slow){
                break;
            }
        }
        slow = head;
        while (true){
            if (fast == slow){
                return fast;
            }
            fast = fast->next;
            slow = slow->next;
        }
    }
};
```
10分で3回
