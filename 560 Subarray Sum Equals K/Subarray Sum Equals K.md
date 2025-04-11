# 560 Subarray Sum Equals K

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 同じ問題を見たことがあるので方法は浮かんだ 
- 累積和をmapに登録しながら、現在調べているindx = rightについて、leftからrightまでの区間和がtargetになるようなleftがあればその数を足していく
- 最初にmapを全て作ってしまうと　left > rightとなるようなleftも見つかってしまうので、良くない（その場合はある累積和に対応するindexの配列を持って、その中をさらに二分探索することになる）

- mapを使わずに累積和の配列を作成して2重ループをする方法もある
  - 累積和は、元の配列numsの右半開区間[L,R)までの累積和をcumlative_sum[R] - cumlative_sum[L] で表すようにする
  - Rは1からnums.size()まで、各Rに対してLは0からR-1までを探索 　

計算量
- mapを使う方法
  - 時間計算量　O(NlogN) 空間計算量　　O(N)
    - mapへの追加にlogN,検索にlogNなので、2*NlogNステップ
      - N = 2*10^4の時、5.7*10^5ステップ　0.57ミリ秒と見積もり
- 2重ループの方法
  - 時間計算量 O(N^2) 空間計算量　O(N)
  - 累積和の作成にNステップ、二重ループにN^2ステップの　N+N^2ステップ
  - N = 2*10^4の時 4.0*10^8ステップ　0.4秒と見積もり（Leetcode上では1.4秒）

二重ループで
```cpp
#include <vector>

class Solution {
public:
    int subarraySum(const std::vector<int>& nums, int k) {
        auto num_of_subarrays = 0;
        std::vector<int> cumulative_sum (nums.size() + 1); 
        for (int i = 0; i < nums.size(); i++) {
            cumulative_sum[i+1] = cumulative_sum[i] + nums[i];
        }
        for (int right = 1; right < nums.size() + 1; right++) {
            for (int left = 0; left < right; left++) {
                if (cumulative_sum[right] - cumulative_sum[left] == k) {
                    num_of_subarrays++;
                } 
            }
        }
        return num_of_subarrays;
    }
};
```
mapを使う方法で
```cpp
#include <map>
#include <vector>

class Solution {
public:
    int subarraySum(const std::vector<int>& nums, int k) {
        std::map<int, int> cumulative_sum_to_frequency;
        auto num_of_subarrays = 0;
        auto cumulative_sum = 0;
        cumulative_sum_to_frequency[0]++;
        for (int i = 0; i < nums.size(); i++) {
            cumulative_sum += nums[i];
            auto compliment = cumulative_sum - k;
            num_of_subarrays += cumulative_sum_to_frequency[compliment];
            cumulative_sum_to_frequency[cumulative_sum]++;
        }
        return num_of_subarrays;
    }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/mura0086/arai60/pull/20/files
- https://github.com/fuga-98/arai60/pull/17/files
- https://github.com/quinn-sasha/leetcode/pull/16/files
- https://github.com/t0hsumi/leetcode/pull/17/files
- https://github.com/Fuminiton/LeetCode/pull/16/files

- ドキュメント系
    - std::upper_bound https://cpprefjp.github.io/reference/algorithm/upper_bound.html
        - 指定した要素より大きいものが初めて現れるイテレータを返す

teachers' eye
- 他人のコードを読んで感情を育てよう　https://github.com/fuga-98/arai60/pull/17/files#r1984363619
- `num_of_subarrays += cumulative_sum_to_frequency[compliment];`　[]oreratorはキーがないと辞書を作るので、キーの存在確認をしたほうがいいか
    - ただキーの数が2倍になってもlogNがlogN+1と1増える程度で、ステップ数が10^4 マイクロ秒単位での差しかないのでどちらでもよいと判断
- たまにマニュアルを読もう　https://github.com/Fuminiton/LeetCode/pull/16/files#r1980996304

#### 感想
- num_of_subarraysより、num_subarrays_with_sum_kという変数名のほうが意味が分かる
- 抽象的な概念を具体例で理解する　https://github.com/quinn-sasha/leetcode/pull/16/files
    - これは多分コメントを見て「へーこの説明分かりやすいな」と思うだけでは能力が育たない気がする。
    - 日ごろから具体例などで考えて落とし込めないかは気にしよう
        - 抽象的な理解ができるているから具体に落とし込めるという両輪もあると思う
- 高低差の例がわかりやすい 
    - mapに最初に0を登録するのは、1個めの駅の前に、海抜0メートルのスタート地点があって、スタート地点の高さを記録しているから。

- left,rightを決め、left-right間の累積和を計算　→ O(N^3) 
- leftを決め、leftから端まで累積和を計算しながら進む　→O(N^2)
- 累積和を求めてから、left - rightのペアの全探索　→O(N^2)
- 今までの累積和をメモしながら、累積和を更新していく　→O(NlogN)
- 累積和のmap`<int, vector<int>>`を作ってから、二分探索 →O(NlogN)


#### STEP1以外の手法と感想
- leftを決め、leftから端まで累積和を計算しながら進む　→O(N^2)
    - 今回はNlogNの手法とN^2の手法で1000倍近く違うので、累積和+mapを用いるのが良いか
    - ただ、今回の制約で、日に1回しか計算しないとかであれば、どちらでもよいが、いずれにせよ累積和をつかうので、累積和+mapがよいと感じた


- あんまりやっている人を見ないので、累積和のmap`<int, vector<int>>`を作ってから、二分探索の方法を実装
```cpp
#include <algorithm>
#include <map>
#include <vector>
//  元の配列numsの右半開区間[L,R)までの累積和をcumlative_sum[R] - cumlative_sum[L]で表す (なお、0 < R <= nums.size())
//  最終的にLを固定したときの条件を満たすRの数を数えるため、mapにはcumlative_sum[R]と,その値を取るRのindexが列挙されている配列がある
class Solution {
public:
    int subarraySum(const std::vector<int>& nums, int k) {
        std::map<int, std::vector<int>> cumulative_sum_to_indexes;
        std::vector<int> cumulative_sum (nums.size() + 1); 
        for (int i = 0; i < nums.size(); i++) {
            cumulative_sum[i+1] = cumulative_sum[i] + nums[i];
            cumulative_sum_to_indexes[cumulative_sum[i+1]].push_back(i+1);
        }
        auto num_subarrays_with_sum_k = 0;
        for (int i = 0; i < nums.size(); i++) {
            auto complement = k + cumulative_sum[i];
            auto* indexes = &cumulative_sum_to_indexes[complement];
            num_subarrays_with_sum_k += (*indexes).end() - std::upper_bound((*indexes).begin(), (*indexes).end(), i);
        }
        return  num_subarrays_with_sum_k;
    }
};

```


## STEP3
### 3回ミスなく書く
累積和＋mapの方法
```cpp
#include <algorithm>
#include <map>
#include <vector>

class Solution {
public:
    int subarraySum(const std::vector<int>& nums, int k) {
        std::map<int, int> cumulative_sum_to_frequency;
        auto cumulative_sum = 0;
        cumulative_sum_to_frequency[0]++;
        auto num_subarrays_with_sum_k = 0;
        for (auto num : nums) {
            cumulative_sum += num;
            auto complement = cumulative_sum - k;
            num_subarrays_with_sum_k += cumulative_sum_to_frequency[complement];
            cumulative_sum_to_frequency[cumulative_sum]++;
        }
        return num_subarrays_with_sum_k;
    }
};
```

10分で3回Accept

#### 2週目の宿題
leftを決め、leftから端まで累積和を計算しながら進む方法の実装
二分探索を実際に書く
