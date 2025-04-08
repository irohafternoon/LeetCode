# 1. Two Sum

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
方針1
- N <= 10^4だから、O(N^2)かかっても、10^8step/秒処理可能として1秒で処理できる
- なのでindexの組を全探索するのもあり
計算量
- 時間計算量O(N^2)
- 追加の空間計算量なし

方針2
- 各値について、出現したindexをvectorでリスト化するmapを持っておく
- numsの各値numについて、mapを検索する
    - target-numが検索対象(serch_target)
    - mapにserch_targetが存在すれば、対応するvectorの最初のindexが答え
        - num = serch_targetが同じな場合は、vectorのサイズ>=2が必要
計算量
- 時間計算量 ~~O(log(N))~~ O(NlogN) mapへの挿入にlogかかる
- 追加の空間計算量O(N)

//共通する仕様は、「もし該当がなければ空のリストを返す」と「複数該当するならindexが一番若いペアを返す」

indexを2重ループで全探索
```cpp
#include<vector>

class Solution {
public:
    std::vector<int> twoSum(const std::vector<int>& nums, const int target) {
    std::vector<int> pair_of_sum_k;
        for (int first_idx = 0; first_idx < nums.size(); first_idx++) {
            for (int second_idx = first_idx + 1; second_idx < nums.size(); second_idx++) {
                if (nums[first_idx] + nums[second_idx] == target) {
                    pair_of_sum_k.push_back(first_idx);
                    pair_of_sum_k.push_back(second_idx);
                    return pair_of_sum_k;
                }
            }
        }
        return pair_of_sum_k;
    }
};
```

mapを用いる方法
```cpp
#include<map>
#include<vector>

class Solution {
public:
    std::vector<int> twoSum(const std::vector<int>& nums, const int target) {
        std::map<int, std::vector<int>> num_to_index_array;
        for (int idx = 0; idx < nums.size(); idx++) {
            num_to_index_array[nums[idx]].push_back(idx);
        }
        std::vector<int> pair_of_sum_k;
        for (const auto& num : nums) {
            int serch_target = target - num;
            if (!num_to_index_array.contains(serch_target)) {
                continue;
            }

            if (num == serch_target) {
                if (num_to_index_array[num].size() <= 1) {
                    continue;
                }
                pair_of_sum_k.push_back(num_to_index_array[num][0]);
                pair_of_sum_k.push_back(num_to_index_array[num][1]);
                return pair_of_sum_k;
            }
            if (num_to_index_array[num].size() == 0 || num_to_index_array[serch_target].size() == 0) {
                continue;
            }
            pair_of_sum_k.push_back(num_to_index_array[num][0]);
            pair_of_sum_k.push_back(num_to_index_array[serch_target][0]);
            return pair_of_sum_k;
        }
        return pair_of_sum_k;
    }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの
- https://github.com/Ryotaro25/leetcode_first60/pull/12/files
- https://github.com/Mahiro-3612/leetcode/pull/1/files
- https://github.com/Shinkomori19/arai60/pull/2/files
- https://github.com/M-Satou955/leetcode_arai60/pull/1/files
- https://github.com/huyfififi/coding-challenges/pull/1/files

- ドキュメント系
    - https://en.cppreference.com/w/cpp/container/unordered_map/contains (unorderd_mapのcontains)
    - https://cpprefjp.github.io/reference/algorithm/sort.html (std::sort)
        - クイックソートの改良版であるイントロソートというものが使われているようだ
    - https://en.cppreference.com/w/cpp/error/runtime_error (std::runtime_error)
        - 例外オブジェクト、ということのようだ、ちゃんとは理解できなかった
    - https://cpprefjp.github.io/reference/iostream/cerr.html
        - 例外処理版のcoutのイメージ
    - https://cpprefjp.github.io/reference/exception/bad_exception/what.html (".what()" errorの理由の文字列を取得)
    - https://en.cppreference.com/w/cpp/language/value_initialization ("{}"による初期化)
    - https://learn.microsoft.com/ja-jp/cpp/cpp/try-throw-and-catch-statements-cpp?view=msvc-170 (c++の例外処理)
    - https://cpp.rainy.me/026-exception.html#%E4%BE%8B%E5%A4%96%E3%82%92%E6%8A%95%E3%81%92%E3%82%8B (c++の例外処理)

teachers' eye
- 例外処理も、いろいろな幅があり、それを選択する　https://github.com/Mahiro-3612/leetcode/pull/1/files#r1981011124
    - 例外処理をやったことがないので step2でやってみる
- クイックソートの注意点　https://github.com/Shinkomori19/arai60/pull/2#discussion_r1995615765
- 言語のupdate 新しい機能や仕様　https://github.com/huyfififi/coding-challenges/pull/1#discussion_r2002946666
- よく知られているものと別の挙動を指定すると、何かそれがダメな理由があると想起させる　https://github.com/huyfififi/coding-challenges/pull/1/files#r2004191267
- ユーザーの影響を考えて総合的な判断。計算量は絶対ではない　https://github.com/huyfififi/coding-challenges/pull/1/files#r2002955215


#### 感想
- serch_target という変数は分かりにくい。「remain」と書いたほうが分かりやすい
- push_backをいちいち書かず、内容を直接{}でリターンしてよい
- 
- 
STEP1で考えたもの以外の手法＆感想
- hashmap(unorderd_map)を利用し、one-wayで実装する方法
    - 「前回までに見た」物の中から今みているnumの相方となれる数字を探す
    - hashmapを更新しながら探すことで、重複する値がある場合にも対応可能
    - map or unorderd_mapはどちらがいいかは単純ではないようだが、とりあえず今回はunorderd_mapを使ってみる

- sortしてから右、左からのtwo_pointerで挟みながら探索
    - 自分が苦手というのもあるが、pointerの動きを想像する必要があり、読み手の認知負荷はhashmap系の手法より高め？

自分の中で一番良いのはmapを更新しながら探索する手法（計算速度も優れていて、認知負荷も高くないので）

まだまだ複数の手法（今回はソートしてtwo_pointer）などが見えていないので、引き続き他の人の方法をマネするところから頑張る

ソートしてtwo_pointer
```cpp
#include<vector>
#include<algorithm> 

struct ValueAndIndex {
    int value;
    int idx;

    bool operator< (const ValueAndIndex& other) const {
        if (value == other.value) {
            return idx < other.idx;
        }
        return value < other.value;
    }
};

class Solution {
public:
    std::vector<int> twoSum(std::vector<int>& nums, const int target) {
    std::vector<int> pair_of_sum_k;
    std::vector<ValueAndIndex> value_and_index_array; //(num, index)
    for (int idx = 0; idx < nums.size(); idx++) {
        value_and_index_array.emplace_back(nums[idx], idx);
    }
    std::sort(value_and_index_array.begin(), value_and_index_array.end());
    int left = 0;
    int right = nums.size() - 1;
    while (left < right) {
        while (value_and_index_array[left].value + value_and_index_array[right].value > target) {
            right --;
        }
        if (value_and_index_array[left].value + value_and_index_array[right].value == target) {
            return {value_and_index_array[left].idx, value_and_index_array[right].idx};
        }
        left ++;
    }
    return {};
  }
};
```
unorderd_map で one_way&例外処理
```cpp
#include <iostream>
#include <stdexcept>
#include <unordered_map>
#include <vector>

class Solution {
public:
    std::vector<int> twoSum(const std::vector<int>& nums, const int target) {
        std::unordered_map<int, int> num_to_index;
        for (int idx = 0; idx < nums.size(); idx++) {
            int num = nums[idx];
            int remain = target - num;
            if (num_to_index.contains(remain)) {
                return {idx, num_to_index[remain]};
            }
            num_to_index[num] = idx;
        }
        throw std::runtime_error("no pair found");
    }
};
//呼び出し元の記述(tryでエラーが出たら、catch内に進む)
int main() {
    Solution solver;
    std::vector<int> nums = {2, 7, 11, 15};
    int target = 30;
    try {
        auto result = solver.twoSum(nums, target);
        std::cout << "Indexes: " << result[0] << ", " << result[1] << std::endl;
    } catch (const std::runtime_error& error) {
        std::cerr << "Error: " << error.what() << std::endl;
        return 1;  //異常終了の場合は1をリターンする
    }
    return 0;
}
```

## STEP3
### 3回ミスなく書く
unordered_map,one-way方式で
例外処理の練習で、main関数も書く
step2と同様なので記載は省略

1回目 8分
2回目 5分
3回目 5分

#### 2週目の宿題
クイックソートやstd::sortの仕様についてさらに調べる
例外処理、例外オブジェクトについての理解を深める
