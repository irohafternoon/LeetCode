# 20. Valid Parentheses

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- stackにカッコを順に入れていくイメージ
- stackが空でなく、今見ているカッコが")","]","}"のうちのどれかで
- その相方がstackのtopであれば、stackをpopする
- それ以外なら、stackに追加する
- 最後にstackが空ならtrue
- 時間・空間計算量O(N)

#### 過程
- if (character == '}' && stack.top()=='{' || .....) のような羅列を書くか悩んだ
- mapを使って羅列を回避したが、別に羅列しても良いなと思った(読み手の認知負荷が小さい)
- mapをつかうと、どんなmapか確認するために視線の移動が発生し、ifの条件文もすんなり入ってこない
- ただ、長文でif（A and (B or C or D)） のような構造になるので、どっちもどっち
- ほかの人のコードが気になる

mapを使用したもの
```cpp
class Solution {
 public:
  map<char, char> open_to_close{
      {']', '['},
      {')', '('},
      {'}', '{'},
  };

  bool isValid(string s) {
    stack<char> parentheses;
    for (char character : s) {
      if (!parentheses.empty() && open_to_close.count(character) &&
          parentheses.top() == open_to_close[character]) {
        parentheses.pop();
        continue;
      }
      parentheses.emplace(character);
    }
    return parentheses.empty();
  }
};
```
mapを使用せず、if文の羅列
```cpp
class Solution {
 public:
  bool isValid(string s) {
    stack<char> parentheses;
    for (char character : s) {
      if (!parentheses.empty() &&
          (character == ')' && parentheses.top() == '(' ||
           character == ']' && parentheses.top() == '[' ||
           character == '}' && parentheses.top() == '{')) {
        parentheses.pop();
        continue;
      }
      parentheses.emplace(character);
    }
    return parentheses.empty();
  }
};

```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/KTakao01/leetcode/pull/5/files
- https://github.com/mura0086/arai60/pull/11/files
- https://github.com/plushn/SWE-Arai60/pull/6/files
- https://github.com/HitoshiKoba/Arai60-public/pull/2/files
- https://github.com/saagchicken/coding_practice/pull/21/files

- コメント集の本題の部分

- ドキュメント系
  - stack (https://cpprefjp.github.io/reference/stack/stack.html)
  - deque (https://cpprefjp.github.io/reference/deque/deque.html)
  - dequeについて (https://kikairoya.hatenablog.com/entry/20100718/1279465696)
  - dequeについて2 (https://qiita.com/recuraki/items/fb84e0133d02e0d53e3f)

- c++ のstackは内部的にdequeを持っているらしい
- pythonはdual-linked-listで管理しているようなので、要素の追加は常にO(1)
- c++ では各ブロックごとのメモリを配列として管理している
- ブロックごとのメモリなので、vectorと異なり1直線に要素がメモリ上に並ぶわけではない
- 要素の追加を続けると、時々メモリを記録する配列をreallocateする必要があり、その時はO(N)かかる
- reallocateされると、配列の大きさは2倍になる
  - 最初は前後に2048個ずつpushできる状態からスタート
  - ということはN回全てpush_frontするとO(NlogN)くらいのオーダーになるということか


#### 感想
- STEP1の書き方はどちらにせよわかりにくい。もっと良い書き方がある
- 1つのif文が長くなるなら、単純に条件分岐を分かりやすくすればいい
- 最後まで文字列を見る必要がない。ダメと分かった瞬間にreturnすればよい
- stackに入れる文字は'(','[','{'の3種類でよい
考え型
- 文字が'(','[','{'のいずれかならstackに入れる
- 文字が'(','[','{'以外の場合
  - stackが空なら、ダメ
  - stackのtopが対応するカッコならstackからpop
  - 対応しないなら、ダメ
  - (カッコですらない別の文字なら、エラーを表示する)
- 最後に、stackが空ならOK

STEP1をbrush-up
```cpp
class Solution {
 public:
  bool isValid(string s) {
    stack<char> open_brackets;
    for (char character : s) {
      if (open_to_close.count(character)) {
        open_brackets.emplace(character);
        continue;
      }
      if (open_brackets.empty()) {
        return false;
      }
      char close_bracket = open_to_close[open_brackets.top()];
      if (close_bracket != character) {
        return false;//これでカッコ以外の入力にもfalseを返す
      }
      open_brackets.pop();
    }
    return open_brackets.empty();
  }

 private:
  map<char, char> open_to_close{
      {'(', ')'},
      {'[', ']'},
      {'{', '}'},
  };
};
```

## STEP3
### 3回ミスなく書く

```cpp
class Solution {
 public:
  bool isValid(string s) {
    stack<char> open_brackets;
    for (char character : s) {
      if (open_to_close.count(character)) {
        open_brackets.emplace(character);
        continue;
      }
      if (open_brackets.empty()) {
        return false;
      }
      char close_bracket = open_to_close[open_brackets.top()];
      if (close_bracket != character) {
        return false;
      }
      open_brackets.pop();
    }
    return open_brackets.empty();
  }

 private:
  map<char, char> open_to_close{
      {'(', ')'},
      {'[', ']'},
      {'{', '}'},
  };
};
```

17分で3回Accept
