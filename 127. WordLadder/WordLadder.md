# 127. Word Ladder

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 単語を頂点としたグラフを考える
- WordListにある各単語を1文字ずつ変えていって、他の単語になれば、単語間に辺を貼る
- BFSで最短距離を求める
- endWordが辞書にないパターンを考慮できずWrongAnserになった

計算量
- wordlist.length = N, word.length = M として
- 時間計算量
	- ボトルネックはmapに隣接リストを登録する箇所でO(N * M * log(N^2))か
		- 辺の数は(N^2)/2で抑えられるため
	- 実際はアルファベットを探索する分の26倍はかかる　
		- N = 10, M = 5000として26倍を考慮すると 3.2 * 10^7 ステップ 32ミリ秒と見積り
空間計算量 
- map作成のためO(N)

```cpp
#include <queue>
#include <map>
#include <set>
#include <string>
#include <vector>

class Solution {
public:
    int ladderLength(std::string beginWord, string endWord,
                     std::vector<std::string>& wordList) {
        std::map<std::string, std::vector<std::string>> adjacency_list;
        std::set<std::string> word_list(wordList.begin(), wordList.end());
        if (!word_list.contains(endWord)) {
            return 0;
        }
        word_list.insert(beginWord);
        for (const auto& original_word : word_list) {
            for (int index = 0; index < original_word.size(); index++) {
                for (int diff = 0; diff < 26; diff++) {
                    auto candidate_word = original_word;
                    candidate_word[index] = char('a' + diff);
                    if (word_list.contains(candidate_word)) {
                        adjacency_list[original_word].push_back(candidate_word);
                    }
                }
            }
        }
        std::queue<std::pair<std::string, int>> word_to_visit;
        word_to_visit.push({beginWord, 1});
        std::set<std::string> seen_words;
        seen_words.insert(beginWord);
        while (!word_to_visit.empty()) {
            auto [word, distance] = word_to_visit.front();
            word_to_visit.pop();
            if (word == endWord) {
                return distance;
            }
            for (const auto& next_word : adjacency_list[word]) {
                if (seen_words.count(next_word)) {
                    continue;
                }
                word_to_visit.push({next_word, distance + 1});
                seen_words.insert(next_word);
            }
        }
        return 0;
    }
};
```
- 解いた後に思ったが、隣接単語を探すやり方は、wordlist内の別の単語と1個づつ比べて、差（ハミング距離のようなもの）が1かを判定すればよかった
	- その代わりmap作成にN^2のオーダーがかかるのか。

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/ichika0615/arai60/pull/14/files
- https://github.com/fuga-98/arai60/pull/20/files
- https://github.com/Fuminiton/LeetCode/pull/20/files
- https://github.com/SuperHotDogCat/coding-interview/pull/45
- https://github.com/tshimosake/arai60/pull/11/files

他人のコードを読むのはむずかしい、大変だ。自然に「他人のコードを読もう」と思える姿勢は大切に、ただ他人のコードさっと読み解けることは今の目標ではない
「読んだけど、よくわからないなあ」でも今は良しとする

- ドキュメント系

teachers' eye
- 表したい意味とコードから読み取れるもの https://github.com/ichika0615/arai60/pull/14/files#r1991234699
- 関数の名前から何が起こるか予想しやすいと良い  https://github.com/ichika0615/arai60/pull/14/files#r1991234699
- 管理する要素をindexすることで処理を軽くできる　https://github.com/SuperHotDogCat/coding-interview/pull/45/files#r1980747151
- 意味のあるメソッドで区切る　https://github.com/tshimosake/arai60/pull/11/files


#### 感想
- 26というマジックナンバーを使わない方が良かった、やるなら定数として小文字の配列を持った方がいいか
- word_listという名前をsetで使っていたが、あまり良くないか
    - 今回はしょうがなくword_setという名前を使っている例も,
    - 使用可能な単語という意味で、availavle_wordsというのはどうだろうか


#### STEP1以外の手法と感想
- O(N^2)かけて隣接リストをつくる方法もある
- 別に隣接リストを作らなくても、BFSしながら隣の文字を調べればよい 
	- 隣接リストが要らない分　N個の単語分の空間計算量を抑えられてよい
- 1文字違いを生み出す関数を作ってもよい（もしくは1文字違いか判定する関数）
    - アルファベット26種類を探索するやり方は、N^2かけて探索するよりも計算ステップは少なくてすむ
    - ただし、入力にアルファベット小文字以外が入ると困る

- 個人的には、コードの明確さ、柔軟性の観点から、BFSの各探索ステップで、available_wordの中から次に進めるものを見つける方法が良い

STEP1の改良（アルファベットをベタ打ちする）
```cpp
#include <queue>
#include <map>
#include <set>
#include <string>
#include <vector>

class Solution {
public:
    int ladderLength(std::string beginWord, string endWord,
                     std::vector<std::string> wordList) {
        const size_t word_length = beginWord.size();
        std::set<std::string> available_words(wordList.begin(), wordList.end());
        available_words.insert(beginWord);
        if (!available_words.contains(endWord)) {
            return 0;
        }
        std::queue<std::pair<std::string, int>> word_to_visit;
        word_to_visit.push({beginWord, 1});
        std::set<std::string> seen_words;
        seen_words.insert(beginWord);
        while (!word_to_visit.empty()) {
            auto [word, distance] = word_to_visit.front();
            word_to_visit.pop();
            if (word == endWord) {
                return distance;
            }
            for (int i = 0; i < word_length; i++) {
                auto candidate_word = word;
                for (auto ch : kLowerCase) {
                    candidate_word[i] = ch;
                    if (!available_words.contains(candidate_word)) {
                        continue;
                    }
                    if (seen_words.contains(candidate_word)) {
                        continue;
                    }
                    word_to_visit.push({candidate_word, distance + 1});
                    seen_words.insert(candidate_word);
                }
            }
        }
        return 0;
    }
    private:
        static constexpr std::string_view kLowerCase = "abcdefghijklmnopqrstuvwxyz";
};
```

文字について、ある文字を抜いた（前半、後半）のpairのキーを作る方法
```cpp
#include <queue>
#include <map>
#include <set>
#include <string>
#include <vector>

class Solution {
public:
    int ladderLength(std::string beginWord, std::string endWord,
                     std::vector<std::string> wordList) {
        const size_t word_length = beginWord.size();
        wordList.push_back(beginWord);
        std::map<std::string, std::vector<std::pair<std::string, std::string>>>
                 word_to_keys;
        std::map<std::pair<std::string, std::string>, std::vector<std::string>>
                 key_to_words;
        bool is_valid_input = false;
        for (const auto& word : wordList) {
            if (word == endWord) {
                is_valid_input = true;
            } 
            for (int i = 0; i < word_length; i++) {
                auto key = std::make_pair(word.substr(0, i),
                                          word.substr(i + 1, word_length - i - 1));
                word_to_keys[word].push_back(key);
                key_to_words[key].push_back(word);
            }
        }
        if (!is_valid_input) {
            return 0;
        }
        std::queue<std::pair<std::string, int>> word_to_visit;
        word_to_visit.push({beginWord, 1});
        std::set<std::string> seen_words;
        seen_words.insert(beginWord);
        while (!word_to_visit.empty()) {
            auto [word, distance] = word_to_visit.front();
            word_to_visit.pop();
            if (word == endWord) {
                return distance;
            }
            for (const auto& key : word_to_keys[word]) {
                if (!key_to_words.contains(key)) {
                    continue;
                }
                for (const auto& adjacent_word : key_to_words[key]){
                    if (seen_words.contains(adjacent_word)) {
                        continue;
                    }
                    word_to_visit.push({adjacent_word, distance + 1});
                    seen_words.insert(adjacent_word);                                        
                }
            }
        }
        return 0;
    }
};
```

BFSの各探索ステップで、available_wordの中から次に進めるものを見つける方法
```cpp
#include <queue>
#include <map>
#include <set>
#include <string>
#include <vector>

class Solution {
public:
    int ladderLength(std::string beginWord, string endWord,
                     std::vector<std::string>& wordList) {
        std::set<std::string> available_words(wordList.begin(), wordList.end());
        if (!available_words.contains(endWord)) {
            return 0;
        }
        available_words.insert(beginWord);
        std::queue<std::pair<std::string, int>> word_to_visit;
        word_to_visit.push({beginWord, 1});
        std::set<string> seen_words;
        seen_words.insert(beginWord);
        while (!word_to_visit.empty()) {
            auto [word, distance] = word_to_visit.front();
            word_to_visit.pop();
            if (word == endWord) {
                return distance;
            }
            for (const auto& candidate_word : available_words) {
                if (!is_adjacent_word(word, candidate_word)) {
                    continue;
                }
                if (seen_words.contains(candidate_word)) {
                    continue;
                }
                word_to_visit.push({candidate_word, distance + 1});
                seen_words.insert(candidate_word);
            }
        }
        return 0;
    }
private:
    bool is_adjacent_word(const std::string& word1, const std::string& word2) {
        if (word1.size() != word2.size()) {
            return false;
        }
        int diff_count = 0;
        for (int i = 0; i < word1.size(); i++) {
            if (word1[i] != word2[i]) {
                diff_count++;
                if (diff_count >= 2) {
                    return false;
                }
            }
        }
        return diff_count == 1;
    }
};
```



## STEP3
### 3回ミスなく書く
BFSの各探索ステップで、available_wordの中から次に進めるものを見つける方法で

```cpp
#include <queue>
#include <map>
#include <set>
#include <string>
#include <vector>

class Solution {
public:
    int ladderLength(std::string beginWord, string endWord,
                     const std::vector<std::string> &wordList) {
        std::set<std::string> available_words(wordList.begin(), wordList.end());
        available_words.insert(beginWord);
        if (!available_words.contains(endWord)) {
            return 0;
        }
        std::set<std::string> seen_words;
        seen_words.insert(beginWord);
        std::queue<std::pair<std::string, int>> word_to_visit;
        word_to_visit.push({beginWord, 1});
        while (!word_to_visit.empty()) {
            auto [word, distance] = word_to_visit.front();
            word_to_visit.pop();
            if (word == endWord) {
                return distance;
            }
            for (const auto& candidate_word : available_words) {
                if (!is_adjacent_word(word, candidate_word)) {
                    continue;
                }
                if (seen_words.contains(candidate_word)) {
                    continue;
                }
                word_to_visit.push({candidate_word, distance + 1});
                seen_words.insert(candidate_word);
            }
        }
        return 0;
    }

private:
    bool is_adjacent_word(const std::string& word1, const std::string& word2) {
        if (word1.size() != word2.size()) {
            return false;
        }
        int diff_count = 0;
        for (int i = 0; i < word1.size(); i++) {
            if (word1[i] != word2[i]) {
                diff_count ++;
                if (diff_count >= 2) {
                    return false;
                }
            }
        }
        return diff_count == 1;
    }
};
```

12分,13分,10分で3回Accept

#### ２周目の宿題
- ダイクストラを適用する（BFSとの違いを考えながら）
- pythonのジェネレータやyieldを理解する
