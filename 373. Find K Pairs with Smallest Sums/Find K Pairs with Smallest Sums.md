# 373. Find K Pairs with Smallest Sums

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- いろいろ考えたがどうしても作成可能なペアを全探索するO(N^2)の方法しか思いつかなかった
- 制約の上限だと10^10ステップ必要で、c++(10^8ステップ/秒)で100秒掛かる見積りなので、現実的でない
- give up して友人にhelpを求める
- 「A,Bのそれぞれi,j番目の和が最小の時、次に小さいのはi+1,jの組か、i,j+1のどっちかなのでpriority_queueで管理する」方針に
- 同じindexのペアを追加しないように、管理するsetが必要

計算量
- K回取り出して、1回につき2つの候補をpriority_queueに入れるので
- 時間計算量　O(Klog(K))　空間計算量 O(K)

```cpp
struct PairSumAndIndexes {
    int pair_sum;
    int index1;
    int index2;
    // operaor< for min_heap
    bool operator<(const PairSumAndIndexes& other) const {
        if (pair_sum == other.pair_sum) {
            if (index1 == other.index1){
                return index2 > other.index2;
            }
            return index1 > other.index1;
        }
        return pair_sum > other.pair_sum;
    }
    
    bool operator==(const PairSumAndIndexes& other) const {
        return index1 == other.index1 && index2 == other.index2;
    }
};

class Solution {
public:
    std::vector<std::vector<int>> kSmallestPairs(const std::vector<int>& nums1, const std::vector<int>& nums2, const int& k) {
        vector<vector<int>> k_smallest_sum_pairs;
        if (nums1.empty() || nums2.empty() || k <= 0) return k_smallest_sum_pairs;

        std::priority_queue<PairSumAndIndexes> smallest_pair_sum_candidates;//min_heap
        std::set<PairSumAndIndexes> seen;
        //pushのための関数を定義
        auto push_candidate = [&] (int idx1, int idx2) -> void {
            PairSumAndIndexes candidate = {nums1[idx1]+nums2[idx2], idx1, idx2};
            if (!seen.contains(candidate)) {
                smallest_pair_sum_candidates.emplace(candidate);
                seen.insert(candidate);
            }
        };
        push_candidate(0, 0);
        int count = k;
        //万が一kがcandidateの数より大きい場合は、全てのペアを昇順に返すことにする
        while (count && !smallest_pair_sum_candidates.empty()) {
            auto [_, index1, index2] = smallest_pair_sum_candidates.top();
            smallest_pair_sum_candidates.pop();
            k_smallest_sum_pairs.push_back({nums1[index1], nums2[index2]});
            if (index1 + 1 < nums1.size()) {
                push_candidate(index1 + 1, index2);
            }
            if (index2 + 1 < nums2.size()) {
                push_candidate(index1, index2 + 1);
            }
            count--;
        }
        return k_smallest_sum_pairs;
    }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの
- https://github.com/Ryotaro25/leetcode_first60/pull/11/files
- https://github.com/Fuminiton/LeetCode/pull/10/files
- https://github.com/SuperHotDogCat/coding-interview/pull/44/files
- https://github.com/fuga-98/arai60/pull/11/files
- https://github.com/plushn/SWE-Arai60/pull/10/files
- https://github.com/mura0086/arai60/pull/15/files


- ドキュメント系

teachers' eye
- pair,tuppleより構造体の方が分かりやすい https://github.com/Ryotaro25/leetcode_first60/pull/11/files#r1627666649
    - 簡単ならpairでもいいなとは思った、今回が情報が複雑なので、構造体がいい
- ライブラリを使うなら、調べましょう　https://github.com/fuga-98/arai60/pull/11/files#r1970944491
- build-in-fanctionの名前を避けよう　https://github.com/plushn/SWE-Arai60/pull/10/files#r2021367811
    - c++ は予約語を避けるということと理解
- if の条件分岐は分かりやすい形に変形できないか（もしくは条件ごとに判定） https://github.com/plushn/SWE-Arai60/pull/10/files#r2021373173
- ラムダ式を用いて暗黙に変数を渡す https://github.com/mura0086/arai60/pull/15/files#r2024513913


#### 感想
- seenに入れる型は構造体でなくpair型で良さそう
    - i,jのindexのペアが入っているくらいであればわざわざそのための構造体を別に作らずにpairで良いという自分の感覚
- 次の候補のindexが収まっているかの判定までpush_candidate関数に入れたほうが簡潔
- push_candidates関数に追加しないケースがある場合変数にそれを明示したほうが分かりやすい、という意見
    - 過度に説明的なのはまどろっこしいと思うけど、try_を先頭につけるくらいならいいかも


#### STEP1の手法以外のものと感想
- O(N^2)の計算量なかで枝刈りをする
    - priority_queueで(N^2)ペア入れても間に合うのか。
    - 枝刈りの方法は(idx1,idx2)の組を全探索するとき、あるidxの組(i,j)の組ですでにpriority_queueからあふれる場合(i,j+1)以降のjの探索は不要
    - numsは昇順にソートされているため、jを進めれば和は必ず大きくなる
    - 全探索なので管理する数が少なければ(N<=10^3ほど)むしろ分かりやすい（場合によっては枝刈りすら不要）
    - 数が多いと(N=10^6以上など)枝刈りでも計算量がかさんでしまう

- 最初にnum1の最小値とnum2の値を全てのペアをpriority_queueに格納する
    - popした(i, j)に対して、(i+1, j)をpriority_queueに入れる
    - はじめに全てのjについて候補が入っているので、拡張がi方向のみでよい
    - また、seenが不要
    - ただ、このロジックをコードだけ見て解読するのは難しそう
    - 個人的には2パターンpriority_queueに挿入&seenで重複管理がいいかな、と思う


nums1,nums2のペア全探索＆枝刈り(LeetCode上で114ms)
```cpp
struct PairSumAndIndexes {
    int pair_sum;
    int index1;
    int index2;
    // operaor< for max_heap
    bool operator<(const PairSumAndIndexes& other) const {
        if (pair_sum == other.pair_sum) {
            if (index1 == other.index1){
                return index2 < other.index2;
            }
            return index1 < other.index1;
        }
        return pair_sum < other.pair_sum;
    }
};

class Solution {
public:
    std::vector<std::vector<int>> kSmallestPairs(const std::vector<int>& nums1, const std::vector<int>& nums2, const int& k) {
        std::vector<std::vector<int>> k_smallest_sum_pairs;
        if (nums1.empty() || nums2.empty() || k <= 0) return k_smallest_sum_pairs;

        std::priority_queue<PairSumAndIndexes> smallest_pair_sum_candidates;//max_heap
        //push関数を定義(ある組(idx1,idx2)がpriority_queueからあふれてしまう場合、idx2の探索を打ち切るフラグも立てる)
        auto push_candidate = [&] (int idx1, int idx2) -> bool {
            if (smallest_pair_sum_candidates.size() < k) {
                smallest_pair_sum_candidates.emplace(nums1[idx1] + nums2[idx2], idx1, idx2);
                return false;
            }
            if (smallest_pair_sum_candidates.top().pair_sum > nums1[idx1] + nums2[idx2]) {
                smallest_pair_sum_candidates.pop();
                smallest_pair_sum_candidates.emplace(nums1[idx1] + nums2[idx2], idx1, idx2);
                return false;
            }
            return true;
        };
        
        //2重ループで全探索（idx2は枝刈りをする）
        for (int idx1 = 0; idx1 < nums1.size(); idx1++) {
            for (int idx2 = 0; idx2 < nums2.size(); idx2++) {
                if (push_candidate(idx1, idx2)) {
                    break;
                };
            }
        }
        //max_heapのため、popしながらvectorに入れた後、最後にvectorをreverseする
        while (!smallest_pair_sum_candidates.empty()) {
            auto [_, index1, index2] = smallest_pair_sum_candidates.top();
            smallest_pair_sum_candidates.pop();
            k_smallest_sum_pairs.push_back({nums1[index1], nums2[index2]});
        }
        reverse(k_smallest_sum_pairs.begin(), k_smallest_sum_pairs.end());
        return k_smallest_sum_pairs;
    }
};
```
STEP1のbrush-up
- push関数の改良,名前変更
- seenの型をpair<int,int>に

```cpp
struct PairSumAndIndexes {
    int pair_sum;
    int index1;
    int index2;
    // operaor< for min_heap
    bool operator<(const PairSumAndIndexes& other) const {
        if (pair_sum == other.pair_sum) {
            if (index1 == other.index1){
                return index2 > other.index2;
            }
            return index1 > other.index1;
        }
        return pair_sum > other.pair_sum;
    }
};

class Solution {
public:
    std::vector<std::vector<int>> kSmallestPairs(const std::vector<int>& nums1, const std::vector<int>& nums2, const int& k) {
        std::vector<std::vector<int>> k_smallest_sum_pairs;
        if (nums1.empty() || nums2.empty() || k <= 0) return k_smallest_sum_pairs;

        std::priority_queue<PairSumAndIndexes> smallest_pair_sum_candidates;//min_heap
        std::set<std::pair<int, int>> seen; //(index1,index2)
        //pushのための関数を定義
        auto try_push_candidate = [&] (int idx1, int idx2) -> void {
            if (idx1 < 0 || idx1 >= nums1.size()) {
                return;
            }
            if (idx2 < 0 || idx2 >= nums2.size()) {
                return;
            }
            auto idx_pair = std::make_pair(idx1, idx2);
            if (!seen.contains(idx_pair)) {
                smallest_pair_sum_candidates.emplace(nums1[idx1]+nums2[idx2], idx1, idx2);
                seen.insert(idx_pair);
            }
        };
        try_push_candidate(0, 0);
        int count = k;
        //万が一kがcandidateの数より大きい場合は、全てのペアを昇順に返すことにする
        while (count && !smallest_pair_sum_candidates.empty()) {
            auto [_, index1, index2] = smallest_pair_sum_candidates.top();
            smallest_pair_sum_candidates.pop();
            k_smallest_sum_pairs.push_back({nums1[index1], nums2[index2]});
            try_push_candidate(index1, index2 + 1);
            try_push_candidate(index1 + 1, index2);
            count--;
        }
        return k_smallest_sum_pairs;
    }
};
```
## STEP3
### 3回ミスなく書く

```cpp
struct PairSumAndIndexes {
    int pair_sum;
    int index1;
    int index2;
    
    bool operator< (const PairSumAndIndexes& other) const {
        if (pair_sum == other.pair_sum) {
            if (index1 == other.index1) {
                return index2 > other.index2;
            }
            return index1 > other.index1;
        }
        return pair_sum > other.pair_sum;
    };
};

class Solution {
public:
    std::vector<std::vector<int>> kSmallestPairs(const std::vector<int>& nums1, const std::vector<int>& nums2, const int& k) {
        std::vector<std::vector<int>> k_smallest_sum_pairs;
        if (nums1.size() == 0 || nums2.size() == 0 || k < 0) {
            return k_smallest_sum_pairs;
        }
        std::priority_queue<PairSumAndIndexes> smallest_pair_sum_candidate;//min_heap
        std::set<std::pair<int, int>> seen;//(index1, index2)
        auto try_push_candidate = [&] (int idx1, int idx2) -> void {
            if (idx1 < 0 || idx1 >= nums1.size()) {
                return;
            }
            if (idx2 < 0 || idx2 >= nums2.size()) {
                return;
            }
            auto index_pair = std::make_pair(idx1, idx2);
            if (!seen.contains(index_pair)) {
                smallest_pair_sum_candidate.emplace(nums1[idx1] + nums2[idx2], idx1, idx2);
                seen.insert(index_pair);
            }           
        };
        try_push_candidate(0,0);
        int count = k;
        while (count && !smallest_pair_sum_candidate.empty()) {
            auto [_, idx1, idx2] = smallest_pair_sum_candidate.top();
            smallest_pair_sum_candidate.pop();
            k_smallest_sum_pairs.push_back({nums1[idx1], nums2[idx2]});
            try_push_candidate(idx1, idx2 + 1);
            try_push_candidate(idx1 + 1, idx2);
            count--;
        }
        return k_smallest_sum_pairs;
    }
};
```

1回目 13分
2回目 10分
3回目 10分

#### 2週目の課題
multimap/setを管理する方法・最初にnum1の最小値とnums2のすべての組み合わせをqueueに入れる方法を実装
