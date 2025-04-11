# 387. First Unique Character in a String

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 添え字i,jを2重ループ+枝刈りする方法
    - 枝刈りは、内側のループでs[i] == s[j]となれば打ち切り
    - 外側のループでは、答えが見つかれば打ち切り
    - 時間計算量O(N^2)だが、アルファベットの種類が高々26種類なので実際のステップ数はN^2よりもかなり小さそう

-  配列を1週してmapを作り、2週目で登場カウントが1のものを出力する方法

計算量
- 2重ループで全探索する方法
    - 時間計算量O(N^2) 追加の空間計算量なし
    - N = 10^5 のとき　10^10ステップ　10秒 の見積り(ただし枝刈りは効く)

- 配列を1週してmapを作り、2週目で登場カウントが1のものを出力する方法
    - 時間計算量O(N) 追加の空間計算量O(1) （アルファベットの数のみなので、mapの挿入・検索にかかるのはlog26=定数）
    - map作成にNlog26,  2週目の判定にNlog26 の計 2*Nlog26ステップ
    - N = 10^5 の時 6.5*10^5ステップ　0.65ミリ秒と見積もり


2重ループで全探索する方法

```cpp
#include <map>
#include <string>
#include <vector>

class Solution {
public:
    int firstUniqChar(const std::string& s) {
        for (int index1 = 0; index1 < s.size(); index1++) {
            bool is_unique = true;
            for (int index2 = 0; index2 < s.size(); index2++) {
                if (index1 != index2 && s[index1] == s[index2]) {
                    is_unique = false;
                    break;
                }
            }
            if (is_unique) {
                return index1;
            }
        }
        return -1;
    }
};
```

配列を1週してmapを作り、2週目で登場カウントが1のものを出力する

```cpp
#include <map>
#include <string>
#include <vector>

class Solution {
public:
    int firstUniqChar(const std::string& s) {
        std::map<char ,int> character_to_frequency;
        for (const auto& character : s) {
            character_to_frequency[character]++;
        }
        for (int index = 0; index < s.size(); index++) {
            auto character = s[index];
            if (character_to_frequency[character] == 1) {
                return index;
            }
        }
        return -1;
    }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/Fuminiton/LeetCode/pull/15/files
- https://github.com/onyx0831/leetcode/pull/2/files
- https://github.com/fuga-98/arai60/pull/16/files
- https://github.com/colorbox/leetcode/pull/29/files
- https://github.com/mura0086/arai60/pull/19/files
- https://github.com/ichika0615/arai60/pull/12/files

- ドキュメント系
  - std::find_if() https://cpprefjp.github.io/reference/algorithm/find_if.html
    - 条件を満たす最初のイテレータを返す
  - constexpr https://cpprefjp.github.io/lang/cpp11/constexpr.html
    - コンパイル時に確定する値に使う、コンパイル時に確定しないものにはつけられない
  - std::min_element https://cpprefjp.github.io/reference/algorithm/min_element.html
    - 最小値のイテレータを返す

teachers' eye
- UTF-8のエンコード方法は 知っている人も多い　https://github.com/ichika0615/arai60/pull/12/files#r1986240112
    - 2週目のどこかか、暇なときに文字コードについてしらべてみよう
- const& 参照　https://github.com/colorbox/leetcode/pull/29/files#r1861423426
  - 参照は場所を検索するステップがかかるため、レジスタ幅（アーキテクチャによるが、4byte or 8byte）に乗るなら参照しないほうがいい
  - 参照は「名前」のようなものという話　https://github.com/colorbox/leetcode/pull/29/files#r1861617719
    - あんまり理解できていない、けど要は場所を実体の名前にするということだろうか。コンビニのタバコを本来は棚の場所を意味する「3番」みたいに指定することに似てる？
- constexpr https://github.com/colorbox/leetcode/pull/29/files#r1861424164
- 実際の.hヘッダファイル https://github.com/colorbox/leetcode/pull/29/files#r1861038507
    - https://source.chromium.org/gn/gn/+/main:src/base/files/file.h;l=155?q=%5C%20%5C%20%2F%2F.*-1%20file:.h$


#### 感想
- -1 がどのような意味を持つのかコードを見るだけだと分からない
    - たしかに、コメントに残すのが良い
- "character" は　"c"でも許容されるようだ

#### STEP1以外の手法と感想
- mapを削除する方法
  - 2度以降出現するものは、mapから消す
  - 3度目に復活してしまうので、消したものを覚えておく
  - 残ったものの中からindexが一番小さいものが答え

  - mapが追加した順番を記憶するのであればこれも良い（先頭だけ見れば良いので）が、そうではないので、結局2週しなければならない
    
- queueに(char,index)のペアをいれながら進む方法
  - 各時点で登録しているmapから、可能な限り削除する
  - step1は最後にまとめて判定するが、この方法は判定しながら進むイメージ
  - one-wayで処理できることがメリット
  - 「a,b,b,b,b,b,......」のような時はqueueがずっと伸びるので、空間計算量はかかる

- double-linked-listを持って情報を管理する
  - mapでcharに対するノードの場所を覚えておき、カウントが2以上になれば、そのノードを切り離して脱落させ、prevをnextに繋ぎ直す
  - こうすると最後に先頭を参照するだけでわかる
  - 実装のコストがかかるが、ノード数も高々26にしかならず、one-wayで実行できる

- 配列を1週してmapを作り、2週目で登場カウントが1のものを出力する方法が簡潔で良いかなと思うが、double-linked-listも良いデータ構造と思った

mapから削除する方法
```cpp
#include <algorithm>
#include <map>
#include <set>
#include <string>
#include <vector>

class Solution {
public:
    int firstUniqChar(const std::string& s) {
        std::map<char, int> unique_character_to_index;
        std::set<int> removed_characters;
        for (int index = 0; index < s.size(); index++) {
            auto c = s[index];
            if (removed_characters.contains(c)) {
                continue;
            }
            if (unique_character_to_index.contains(c)) {
                unique_character_to_index.erase(c);
                removed_characters.insert(c);
                continue;
            }
            unique_character_to_index[c] = index;
        }
        if (unique_character_to_index.empty()) {
            return -1;  //not found
        }
        std::vector<int> unique_character_indexes;
        for (auto [_, index] : unique_character_to_index) {
            unique_character_indexes.push_back(index);
        }
        return *std::min_element(unique_character_indexes.begin(), unique_character_indexes.end());
    }
};
```

queueに格納していく方法
```cpp
#include <map>
#include <queue>
#include <string>
#include <vector>

struct CharacterAndIndex {
    char c;
    int index;
};

class Solution {
public:
    int firstUniqChar(const std::string& s) {
        std::map<char, int> character_to_frequency;
        std::queue<CharacterAndIndex> characters;
        for (int index = 0; index < s.size(); index++) {
            auto c = s[index];
            character_to_frequency[c]++;
            characters.push({c, index});
            while (!characters.empty() && character_to_frequency[characters.front().c] >= 2) {
                characters.pop();
            }
        }
        if (characters.empty()) {
            return -1;  //not found
        }
        return characters.front().index;
    }
};
```

double-linked-listの実装は2周目の宿題

## STEP3
### 3回ミスなく書く

```cpp
#include <map>
#include <string>
#include <vector>

class Solution {
public:
    int firstUniqChar(const std::string& s) {
        std::map<char, int> character_to_frequency;
        for (auto c : s) {
            character_to_frequency[c]++;
        }
        for (int index = 0; index < s.size(); index++) {
            auto c = s[index];
            if (character_to_frequency[c] == 1) {
                return index;
            }
        }
        return -1;  //not found
    }
};
```

9分で3回Accept

#### 2周目の宿題
- double-linked-listの実装
- constexprをもっと調べる
- 文字エンコードについて調べる
