# 349. intersections of Two Arrays
https://leetcode.com/problems/intersections-of-two-arrays/

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- nums1, nums2をuniqueにして、片方がの数字がもう片方にあるかを調べていく
- uniqueにする方法として、両方setにする方法をとった
- 効率のため、nums1とnums2のうち、要素数が少ないものをnums1に固定する

計算量
- 時間計算量O(NlogN)
    - ソートにNlogN+MlogM, N個の要素の判定にNlogMステップ
    - NlogN+MlogM+NlogM N=M=1000のとき30000ステップほどなので、c++(10^9/秒)として3マイクロ秒
- 空間計算量O(N)
    - 2種類のsetと答えの格納用の配列

- std::uniqueというものがあった記憶があったのでstd::uniqueを調べた
    - 重複を削除するのでなく、重複した要素を端っこに寄せ、uniqueになった配列の末尾のイテレータを返す（要素を削除するわけではない）

```cpp
#include <set>
#include <vector>

class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        std::set<int> nums1_values(nums1.begin(),nums1.end());
        std::set<int> nums2_values(nums2.begin(),nums2.end());
        if (nums1_values.size() > nums2_values.size()) {
            std::swap(nums1_values,nums2_values);
        }
        vector<int> intersections;
        for (auto nums1_value: nums1_values) {
            if (nums2_values.contains(nums1_value)) {
                intersections.push_back(nums1_value);
            }
        }
        return intersections;
    }
};
```

std::uniqueを使う方法
```cpp
#include <set>
#include <vector>

class Solution {
public:
    std::vector<int> intersection(std::vector<int> nums1, std::vector<int> nums2) {
        if (nums1.size() > nums2.size()) {
            std::swap(nums1,nums2);
        }
        sort(nums1.begin(), nums1.end());
        auto nums1_new_end = std::unique(nums1.begin(), nums1.end());
        std::set<int> nums2_values(nums2.begin(), nums2.end());
        vector<int> intersections;
        for (auto itr = nums1.begin(); itr != nums1_new_end; itr++) {
            if (nums2_values.contains(*itr)) {
                intersections.push_back(*itr);
            }
        }
        return intersections;
    }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/Fuminiton/LeetCode/pull/13/files
- https://github.com/fuga-98/arai60/pull/14/files
- https://github.com/mura0086/arai60/pull/17/files
- https://github.com/olsen-blue/Arai60/pull/13/files
- https://github.com/quinn-sasha/leetcode/pull/13/files

- ドキュメント系
    - std::unique
        - https://cpprefjp.github.io/reference/algorithm/unique.html
        - 重複を削除するのでなく、重複した要素を端っこに寄せ、uniqueになった配列の末尾のイテレータを返す（要素を削除するわけではない）、なお、端っこに寄せた重複要素の値は未規定になる
        - 計算量O(N)
    - std::swap
        - https://cpprefjp.github.io/reference/utility/swap.html
        - 内部的にはstd::moveを行うのと同等のよう

teachers' eye
- この問題を題材に、追加のシチュエーションが（面接で）与えられる https://github.com/quinn-sasha/leetcode/pull/13/files#r1960884543
- 考えたらキリがない、どこかで所与とするのも１つの態度　https://github.com/olsen-blue/Arai60/pull/13/files#r1929931137
- コードのコメントはチームメンバーに残すと思って　https://github.com/olsen-blue/Arai60/pull/13/files#r1915848634
- コードの見やすさ以外にも、いろんなケースを想定して方法を考える　https://github.com/mura0086/arai60/pull/17/files#r2027894970


#### 感想
- "片方がとても大きくて、片方がとても小さいときには、大きい方を set にするのは大変じゃないでしょうか、特に大きいほうが sort 済みのときにはどうしますか"という問い (https://github.com/fuga-98/arai60/pull/14/files#r1977144526)
    - indexを2つ管理する方法は頭をよぎったのだがsetが簡単だと思ってすぐ思考から外してしまった。
    - まだ問題を解くことで満足している証拠  
    - このようなケースをいろいろ想定していろんな手法を選択する練習をしなくては。
    - でも、「何が足りないか」を自分で認識できたのは収穫。
- 配列が1TBぐらいある場合は、分割して読み込んで、分割して出力していく　←　なるほど。想像もしたことがなかった。
    - こういうのをいろいろ考えられるようになると楽しそう
    - ファイルを読み込むという話になると、CSの知識もある程度いりそう

#### STEP1以外の手法と感想
- nums1,nums2のindexを2つ管理して、片方のindexが示す値が大きければう一方を進める、逆も同じ、同じものを見つけた場合は配列にいれる
    - nums1,nums2は事前にソートする
    - これは配列がメモリに乗らないようなサイズの入力が与えられても実行可能
        - メモリに乗るサイズにファイルを分割して読み込み、分割して結果を書き込みしていけばよい
        - 分割で読み込んでも、常に「最後に追加した値」を覚えておけば大丈夫そう
    - 必要な追加の空間計算量が実質O(1)(答えがすべて入った配列を用意するという出力の指定がなければ)
    - 時間計算量はO(NlogN)
    - このケースでは総合的に一番優れていると思った

- 入力される値が0以上1000以下であることを利用して、長さ1001の配列にnums1にある値を記録する方法
    - 時間計算量O(N)
    - 追加の空間計算量は実質O(1000)
    - 入力される値が有限で、想定される種類数が小さく、値の幅も小さい（nums1,もしくはnums2を要約した配列のサイズがメモリに乗る）場合は、要素数が非常に大きい場合も対応可能
    - ただ、その場合はsetを用いる方法も使用可能で、差異は時間計算量のlogがかかる部分
        - メモリの上限を仮に10GB,入力がint型(4bite)とすれば、要素数が2.6*10^8程度なのでlogをとると30程
        - c++のステップ数を10^9程と見積ると、logがないと0.26秒 logがつくと8秒
        - 同様の計算を何度も繰り返すなら、30倍の差は大きい思われる。そうでないなら、setのほうが柔軟と思うのでsetが良いと思った
    - 要素数が少なくても、要素の値がとりえる幅が大きいなら配列を用意できないのでsetを選択する
    - もしくは数字以外で同様のことをしたい場合もsetを選択



2つのindexを管理する方法で実装

大体のPRは nums1[idx1] == nums2[idx2] の時に違う数字になるまでインクリメントしていた。
一度にインクリメントを何度も進めてしまうと、途中でファイルをまたぐときにややこしいかな、と思ったので

whileの1ターンの中ではidx1,idx2は高々1回しか進まないようにした

分割ファイルを扱って、出力用の配列を新しくするとき、最後に追加したものだけは新しい配列に引き継ぐイメージ

```cpp
class Solution {
public:
    std::vector<int> intersection(std::vector<int> nums1, std::vector<int> nums2) {
        sort(nums1.begin(), nums1.end());
        sort(nums2.begin(), nums2.end());
        auto idx1 = 0;
        auto idx2 = 0;
        std::vector<int> intersections;
        while (true) {
            if (idx1 == nums1.size() || idx2 == nums2.size()) {
                break;
            }
            if (nums1[idx1] == nums2[idx2]) {
                auto maybe_push_num = nums1[idx1];
                if (intersections.empty() || !(intersections.back() == maybe_push_num)) {
                    intersections.push_back(maybe_push_num);
                }
                idx1++;
                idx2++;
                continue;
            }
            if (nums1[idx1] < nums2[idx2]) {
                idx1++;
                continue;
            }
            if (nums1[idx1] > nums2[idx2]) {
                idx2++;
            }
        }
        return intersections;
    }
};
```

まだ時間ををかけすぎているが、とりあえず考えたこと書くとこれくらいになってしまう。
もう少しなれたら、簡潔にできないか考えよう

## STEP3
### 3回ミスなく書く

2つのindexを管理する方法で

```cpp
#include <vector>

class Solution {
public:
    std::vector<int> intersection(std::vector<int> nums1, std::vector<int> nums2) {
        sort(nums1.begin(), nums1.end());
        sort(nums2.begin(), nums2.end());
        std::vector<int> intersections;
        int idx1 = 0;
        int idx2 = 0;
        while(true) {
            if (idx1 == nums1.size() || idx2 == nums2.size()) {
                break;
            }
            if (nums1[idx1] == nums2[idx2]) {
                auto maybe_push_num = nums1[idx1];
                if (intersections.empty() || !(intersections.back() == maybe_push_num)) {
                    intersections.push_back(maybe_push_num);
                }
                idx1++;
                idx2++;
                continue;
            }
            if (nums1[idx1] < nums2[idx2]) {
                idx1++;
                continue;
            }
            if (nums1[idx1] > nums2[idx2]) {
                idx2++;
            }
        }
        return intersections;
    }
};
```

1-3回目全て4分

#### 2週目の宿題
- 2分探索を用いる手法もあるので、それを実装する
- どのようにしてファイルを分割して読み込むのか、具体的な手法を調べる
