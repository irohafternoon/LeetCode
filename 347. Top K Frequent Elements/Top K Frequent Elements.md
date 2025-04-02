# 347. Top K Frequent Elements

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- スコアごとに出現回数をカウント
- 昇順にソートされたsetに<-1*出現回数,値>のpair型を入れ、小さい順にK回取り出して、vectorにpushする


計算量
- 時間計算量O(logN)、空間計算量O(N)

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        std::map<int, int> score_count;
        std::set<std::pair<int,int>> count_and_score;
        for (auto score: nums) {
            score_count[score]++;
        }
        for (auto [score, count]: score_count) {
            count_and_score.insert(std::make_pair(-count,score)); 
        }
        int push_count = 0;
        std::vector<int> result;
        for (auto pair: count_and_score) {
            result.emplace_back(pair.second);
            push_count++;
            if (push_count == k) break;
        }
        return result;
    }
};
```


## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/plushn/SWE-Arai60/pull/9/files
- https://github.com/mura0086/arai60/pull/14/files
- https://github.com/fuga-98/arai60/pull/10/files
- https://github.com/Fuminiton/LeetCode/pull/9
- https://github.com/SanakoMeine/leetcode/pull/11/files


- ドキュメント系
setやmapに比較関数を定義するやり方
    - set https://en.cppreference.com/w/cpp/container/set/set
    - map https://en.cppreference.com/w/cpp/container/map/map
    std::greaterというのは比較"オブジェクト"のようだ。宣言の時に、比較オブジェクト型を渡してあげる
    もしくはコントラクタにオブジェクト自体を引数に渡す

teachers' eye
- pair型は中身が何を指しているか分かりにくいので、構造体と大小関係を定義したほうが分かりやすい https://github.com/mura0086/arai60/pull/14/files#r2009675956
- 混同しやすい変数名はつけない　https://github.com/mura0086/arai60/pull/14/files#r2009679122
- 使わない変数に'_'は使用できるが、 __ を含む名前や _ で始まる名前は避けるべき(c++26から _ が言語使用として登場) https://github.com/mura0086/arai60/pull/14/files#r2009727030
- ドキュメントを読む習慣はgood https://github.com/plushn/SWE-Arai60/pull/9/files#r2014608685 
- 1行に処理を詰め込むと目が降られたり可読性が下がる https://github.com/SanakoMeine/leetcode/pull/11/files#r1925814299
- return直前にわざわざreturn用の変数を用意する必要は(基本)ない https://github.com/SanakoMeine/leetcode/pull/11/files#r1925815250

#### 感想
- setやmapにも自作の比較関数やstd::greaterを使って降順にソートすることは可能
    - -1倍して昇順で処理する方法よりもわかりやすい
- pair型が分かりにくいのはその通りと感じた。構造体を作ってみよう
- count という名詞よりfrequencyのほうが分かりやすいと思った
- result もあまり意味のない名前 top_k_scoresが良さそう
- setじゃなくてpriority_queueに要素数kを保つようにpushしていけば良かった

```cpp
struct ScoreAndFrequency {
  int score;
  int frequency;

  ScoreAndFrequency(int score, int frequency)
      : score(score), frequency(frequency) {}

  bool operator>(const ScoreAndFrequency& other) const {
    if (frequency == other.frequency) {
      return score < other.score;
    }
    return frequency > other.frequency;
  }
};

class Solution {
 public:
  vector<int> topKFrequent(vector<int>& nums, const int& k) {
    for (const auto& score : nums) {
      score_to_frequency[score]++;
    }
    for (const auto& [score, frequency] : score_to_frequency) {
      top_k_frequency_heap.emplace(score, frequency);
      if (top_k_frequency_heap.size() > k) {
        top_k_frequency_heap.pop();
      }
    }
    std::vector<int> top_k_scores;
    while (!top_k_frequency_heap.empty()) {
      const auto& [score, _] = top_k_frequency_heap.top();
      top_k_scores.emplace_back(score);
      top_k_frequency_heap.pop();
    }
    return top_k_scores;
  }

 private:
  std::map<int, int> score_to_frequency;
  std::priority_queue<ScoreAndFrequency, std::vector<ScoreAndFrequency>,
                      std::greater<ScoreAndFrequency>>
      top_k_frequency_heap;
};
```
```cpp
bool operator< (const ScoreAndFrequency& other) const {
```
の末尾のconstは、「この関数はobjectを変更しない」という意味
最少ヒープをするには、operator>を定めて、std::greatorを使ったほうが原則に沿っているか
< を本来と逆向き定義するのは混乱させる
自作構造体ではCTADは使えない（ようだ）

## STEP3
### 3回ミスなく書く

```cpp
struct ScoreAndFrequency{
    int score;
    int frequency;
    ScoreAndFrequency(int score, int frequency)
      : score(score), frequency(frequency) {}
    
    bool operator> (const ScoreAndFrequency& other) const {
        if (frequency == other.frequency) {
            return score < other.score;
        }
        return frequency > other.frequency;
    }
};

class Solution{
  public:
    std::vector<int> topKFrequent(std::vector<int>& nums, const int& k) {
        for (const auto& num : nums) {
            score_to_frequency[num]++;
        }
        for (const auto& [score, frequency] : score_to_frequency) {
            top_k_frequency_heap.emplace(score, frequency);
            if (top_k_frequency_heap.size() > k) {
                top_k_frequency_heap.pop();
            }
        }
        std::vector<int> top_k_scores;
        while (!top_k_frequency_heap.empty()) {
            const auto& [score, _] = top_k_frequency_heap.top();
            top_k_scores.emplace_back(score);
            top_k_frequency_heap.pop();
        }
        return top_k_frequency;
    }
  private:
    std::map<int, int> score_to_frequency;
    std::priority_queue<ScoreAndFrequency,
                        std::vector<ScoreAndFrequency>,
                        std::greater<ScoreAndFrequency>>
                        top_k_frequency_heap//minheap
};
```
1回目:15分
2回目:9分
3回目:8分

#### 2週目への宿題
構造体の比較関数やset,mapに使う比較objectについての理解をより深める

