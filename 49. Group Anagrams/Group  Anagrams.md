# 49. Group Anagrams

https://leetcode.com/problems/group-anagrams/

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 各stringをソートしたものをキーとすれば、mapにグループごとに格納できる
- 最後に、mapの各グループをvectorに格納する
- 名前空間を汚さないため,全てstd::をつけているが、かなり見づらいこともある気がしてきた。
- 実務はusing namespase stdを使わず、基本的にstd::を逐一つけるんだろうか
- 「名前の変更」のためにstd::moveを使ってみた。このような使い方は一般的なのだろうか
    - コピーせずに名前を変更したいシーンはあるように思う
    - 今回のような短いstringは、コピーのコストが小さいため、わざわざ使わないのかもと想定

- ソートしたstring以外も、各文字の出現頻度のmapをキーとすることも考えたが、複雑すぎると思った

計算量
strs.lengthをN, 各文字の長さをMとすると
- 時間計算量O(NlogN) (ソートにMlogM,挿入にlogN) なのでN*(MlogM+logN)ステップ程度と見積り
    - N = 10^4, M = 10^2 で6,800,000ステップほど
    - c++は10^9ステップ/秒と仮定して、7ms程と見積る
- 空間計算量O(N) N個の要素を格納するため

```cpp
#include <map>
#include <vector>

class Solution {
public:
    std::vector<std::vector<std::string>> groupAnagrams(const std::vector<std::string>& strs) {
        std::map<std::string, std::vector<std::string>> string_to_group;
        for (const auto& original_str : strs) {
            auto str_for_sort = original_str;
            sort(str_for_sort.begin(), str_for_sort.end());
            auto sorted_str = std::move(str_for_sort);
            string_to_group[sorted_str].push_back(original_str);
        }
        std::vector<std::vector<std::string>> anagram_groups;
        for(const auto& [_, group] : string_to_group) {
            anagram_groups.push_back(group);
        }
        return anagram_groups;
    }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/mura0086/arai60/pull/16/files
- https://github.com/HitoshiKoba/Arai60-public/pull/5/files
- https://github.com/Shinkomori19/arai60/pull/3/files
- https://github.com/Mahiro-3612/leetcode/pull/2/files
- https://github.com/fuga-98/arai60/pull/13/files

- 

- ドキュメント系
- std::move https://cpprefjp.github.io/reference/utility/move.html
    - 左辺値、右辺値など　https://qiita.com/luftfararen/items/1de032bc6e3eb69ca672
    - 右辺値参照・ムーブセマンティクス　https://cpprefjp.github.io/lang/cpp11/rvalue_ref_and_move_semantics.html
    - std::moveは左辺値を右辺値参照にキャストする
        - コピーのコストが高いクラスなどでは、コントラクタに右辺値参照を渡すことでムーブコントラクタを呼ぶことに利点がある。
        - 長い文字列等をvectorにpush_backする場合は、push_back(std::move(str))のように使える

teachers' eye
- 意図しない入力の可能性　https://github.com/mura0086/arai60/pull/16/files#r2026151443
- 文字コードの大変さを理解する　https://github.com/HitoshiKoba/Arai60-public/pull/5/files#r2020661663
    - 一度時間をとって文字コードについて少し調べてみよう。「サロゲートペア」「結合文字」などを追加キーワードに
- 操作をメソッドにする　https://github.com/Shinkomori19/arai60/pull/3/files#r2019123777
- 自然言語で説明するときに一呼吸置くなら、わける　https://github.com/Mahiro-3612/leetcode/pull/2/files
- 何回も使うなら、変数に置く　https://github.com/Mahiro-3612/leetcode/pull/2/files
- アルファベットが連続していない環境もある/小文字以外がある場合　https://github.com/mura0086/arai60/pull/16/files#r2026151443


#### 感想
- string,str系の名前はwordに置き換えたほうが良さそう
- sorted_strのような名前を（後にソートするからという理由で）ソート前の変数として名づけるのが違和感というコメントもあった
    - 自分もそう思ってstd::moveを使った
- 「ソートされた単語がkeyでアナグラムが同じ単語のグループがvalueの辞書」の名前はどの方のPRを見てもしっくりとこなかった
    - 直訳すると sorted_word_to_anagram_groupになるが、sorted_word_to_groupとかで良いかな


step1 以外の手法とその感想
各文字の出現頻度の配列をキーとする方法
- 各文字の出現頻度のmapをキーという手法をstep1の時に思ったが、mapでなくvectorでよい（アルファベットが高々26種類なので）
- ソートしたほうがコードも簡潔で分かりやすいと感じる
- ただ、例えば構成する要素の数が有限でかつ、ソート不可能なオブジェクトを分類するようなタスクの場合はこちらが使える


各文字の出現頻度の配列をキーとする方法
```cpp
#include <iostream>
#include <map>
#include <stdexcept>
#include <vector>

class Solution {
public:
    std::vector<std::vector<std::string>> groupAnagrams(const std::vector<std::string>& strs) {
        std::map<std::vector<int>, std::vector<std::string>>letter_frequency_to_group;
        for (const auto& word : strs) {
            std::vector<int> letter_frequency;
            try {
                letter_frequency = make_letter_frequency(word);
            } catch (const std::runtime_error& error) {
                std::cerr << "Error: " << error.what() <<" in '" << word <<"'"<<std::endl;
                return {};
            }
            letter_frequency_to_group[letter_frequency].push_back(word);
        }
        std::vector<std::vector<std::string>> anagram_groups;
        for (const auto& [_, group] : letter_frequency_to_group) {
            anagram_groups.push_back(group);
        }
        return anagram_groups;
    }

private:
    static std::vector<int> make_letter_frequency(const std::string& str) {
        std::vector<int> letter_frequency(26);
        for (const auto& character : str) {
            if (character < 'a' || character > 'z') {
                throw std::runtime_error("invalid letter");
                return {};
            }
            letter_frequency[character - 'a']++;
        }
        return letter_frequency;
    }
};
```
このコードは英子文字でない文字があるときは終了させるという対応だが、その単語は無視するやり方もあるなと思った。
ケースバイケース

SETP1のbrush-up
```cpp
#include <map>
#include <vector>

class Solution {
public:
    std::vector<std::vector<std::string>> groupAnagrams(const std::vector<std::string>& strs) {
        std::map<std::string, std::vector<std::string>> sorted_word_to_group;
        for(const auto& original_word : strs) {
            auto word_for_sort = original_word;
            sort(word_for_sort.begin(), word_for_sort.end());
            auto sorted_word = std::move(word_for_sort);
            sorted_word_to_group[sorted_word].push_back(original_word);
        }
        std::vector<std::vector<std::string>> anagram_groups;
        for (const auto& [_, group] : sorted_word_to_group) {
            anagram_groups.push_back(group);
        }
        return anagram_groups;
    }
};
```

## STEP3
### 3回ミスなく書く
一番良いと感じているsortしたwordをキーにする方法
```cpp
#include <map>
#include <vector>

class Solution {
public:
    std::vector<std::vector<std::string>> groupAnagrams(const std::vector<std::string>& strs) {
        std::map<std::string, std::vector<std::string>> sorted_word_to_group;
        for (const auto& original_word : strs) {
            auto word_for_sort = original_word;
            sort(word_for_sort.begin(), word_for_sort.end());
            auto sorted_word = std::move(word_for_sort);
            sorted_word_to_group[sorted_word].push_back(original_word);
        }
        std::vector<std::vector<std::string>> anagram_groups;
        for (const auto& [_, group] : sorted_word_to_group) {
            anagram_groups.push_back(group);
        }
        return anagram_groups;
    }
};
```

1-3回目全て4分

