# 703. Kth Largest Element in a Stream

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 常にtopKだけが格納されていることが約束されたqriority_queue(最小値取り出し)を用意
- 要素サイズがkを超えたときは、popすることで、常に
- 追加操作後に、queueの最小値を出力する

計算量
- 時間計算量O(NlogK),追加の空間計算量O(k)
    - 各要素を逐一挿入するため時間計算量はlogKが掛かる

```cpp
//topKが入るpriority_queueを作る
//k個を超えたらpopすることで、自動的にtopKを常に保つことが可能
class KthLargest {
private:
    const int k;
    std::priority_queue<int, std::vector<int>, std::greater<int>> top_k_scores;
public:
    KthLargest(int k, vector<int>& nums) : k(k) {
        //常にheap_queueのtopがkthになるようにする
        for (auto num : nums) {
            top_k_scores.emplace(num);
            if (top_k_scores.size() > k) {
                top_k_scores.pop();
            }
        }
    }
    int add(int val) {
        top_k_scores.emplace(val);
        if (top_k_scores.size() > k) {
            top_k_scores.pop();
        }
        return top_k_scores.top();
    }
};

```
### STEP2の前に
自分の中で、STEP2の「マニュアル化」・「課題レポート化」みたいなものを感じる。
この練習で一番大事なのはこのSTEP2で間違いないとは思う。
今のところ「人のコードを読む」「新しいことを理解する」「人のコードにコメントをつける」「感想を書いたり、自分の感情を持つ」は「やったほうがいいこと」とされていて、それはそうだと思う。

でも、多分上記の「やったほうがいいこと」を「leetcode演習における課題」と見做して、「これだけやりました！良いことをしました！」という「提出物」としてこのメモをレポートにしてＰＲで提出するのは、意識がズレているのかもしれないな。と思った。
このSTEP2を充実させなければ、みたいな謎のプレッシャーみたいなものをたまに感じるときがあるため。

なので、今後このSTEP2の記述を充実させるということを自体は目的としないようにする。
具体的には、STEP2にかける時間を制限する。（一旦、リサーチに使う時間の上限を2hとする）
その代わり、「後日戻ってきてからもう一度やれることを考える」ことをする。
Arai60を2週するイメージ。
そのために、1週目では、やり残したこと＝2週目の宿題を何にするかはできる限り記録する。

なぜなら、自分がこのコーディング練習会を引き継ぐことを考えたら、まずはざっくり60問を見渡したいと思うから。最初の1週はとにかく進めたい。あとは引継ぎ元の今の「先生」がどんな指導をしてるか知りたいと思う。
なので、STEP2に新たに "teachers' eye"という項目を設けて、講師陣がどんな点を気にしてコメントしてたかはメモするようにする。真意がわからなくても、とりあえず気になったものはメモしておく。

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/mura0086/arai60/pull/13/files
- https://github.com/lilnoahhh/leetcode/pull/11/files
- https://github.com/ichika0615/arai60/pull/8/files
- https://github.com/konnysh/arai60/pull/8/files
- https://github.com/Fuminiton/LeetCode/pull/8/files
- 

- コメント集の本題の部分

- ドキュメント系
- std::priority_queue
    - https://en.cppreference.com/w/cpp/container/priority_queue
    - コントラクタにコピー元のコンテナのイテレータを指定して構築することもできる
    - 比較関数Compare
        - デフォルトは"<" a < b がtrueなら aはbの"前"にくる）
        - このように並べ替えた後にqueueの"一番後ろ"を.top()などで出力するので、デフォルトは最大値。
        - std::greaterを指定すると 比較関数が ">" になるため、最少が出力されるようになる
- std::greater
    - https://en.cppreference.com/w/cpp/utility/functional/greater
        - 細かいことが理解できなかったのだが、とりあえず x>yを 返す比較関数のようなもの理解した

- heapについて
    - https://medium.com/@yasufumy/data-structure-heap-ecfd0989e5be
    - https://ufcpp.net/study/algorithm/col_heap.html
    - heap化
        - ある親ノードについて、親ノードと子ノード、子ノードと孫ノードの順序関係を見る
            - 親といずれかの子で順序関係を満たさないが
            - それぞれの子については、いずれの孫とも順序関係を満たしている場合
            - 子ノードのうち、順序が手前に来る方を親と入れ替える
    これを葉以外の深いノードから順に操作を行うことで、O(N)で並び替えができる
    - pop
        - 一番"右側"の葉を根に持っていき、順序関係を満たすまで根→葉の方向に入れ替えていく
            - 計算量は階層分なのでlog(N)
    - push
        - 1番"右側"に新しい葉を作って値を格納、以降順序関係を満たすまで葉→根の方向に入れ替えていく
        - 計算量は階層分なのでlog(N)

#### 感想
- kが負の場合になることを想定すらしなかったのはまずい（LeetCode上では無いとはいえ）
- this-> をつけると"クラス変数"という意味を明示できる(k=kのような記述は分かりにくい)
- style guide での定数の命名(先頭k+大文字小文字交じり)を忘れていた。this->じゃなくて命名で区別しよう
- コントラクタとadd関数でqueueに数字を入れる部分の処理は共通してるので、共通化してもよいかも
- 配列であるnumsをまずheapifyする方法もある
    - 最初の配列の数をN,追加する数をMとする場合、時間計算量N+min(0,N-K)logK+Mlog(K)
    - 最初の方針の場合は NlogK +MlogK
    - 差は最初のN個までのheapqを構築するところ、Nが大きくてKが小さいなら作ったheapqを削る分逐一挿入が早い　NとKが近いなら削る式が早い 
- 同じ平衡二分木という意味では、mapでもできるのか。
- 最少ヒープをであることを明示するために変数名をtop_k_scoresにしたが、top_k_scores等のほうが良いか
    - 最小値が取り出されることはコメントで書けばよい
- 自分は「常にheapqをtopKが確保されることを担保したい」という気持ちだったが、topKが確保されるタイミングがaddが行われたタイミングなので、「いつ」「何が約束されている必要があるか」で考えたほうがいいと思った。
- 配列を毎回ソートする方法もあると思った（一応全てデータを格納できるため）が、それをしたいなら入れた順番の情報が落ちるのは嫌だし、priority_queueとvectorを別でもってvectorには入れた順番通りemplace_backしたほうが取り回しやすいかな、思った。

- 自分としてはstep1の方法の方が好み


#### teachers' eye
- 計算量の見積り方 https://github.com/lilnoahhh/leetcode/pull/11/files#r1980827776
    - とりあえず,c++なら1秒間に10^8ステップ処理できると仮定しよう。N = 10*5 で,N^2ステップかかるとしたら、100秒かかるといった感じ
- エラーの表示させ方　https://github.com/lilnoahhh/leetcode/pull/11#discussion_r1981173052
- 手作業でやるとしたら？https://github.com/mura0086/arai60/pull/13/files#r2002989542
- private:　も明示 https://github.com/mura0086/arai60/pull/13/files#r2007609945
- 入力を変化すると使用者から意図しない挙動に見える可能性がある https://github.com/frinfo702/software-engineering-association/pull/11/files#r1885458834
- 複雑な実装は紙に絵をかいて考えてみよう https://github.com/ichika0615/arai60/pull/8#discussion_r1898181669
- 他の人の手法を見てみよう https://github.com/frinfo702/software-engineering-association/pull/11/files#r1885074119
- init(コントラクタ)でaddを呼ぶ https://github.com/Fuminiton/LeetCode/pull/8/files#r1957384623 

step1のbrash-up
```cpp
class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) : kHeapSize(k) {
        for (auto score : nums) {
            add(score);
        }
    }
    int add(int val) {
        top_k_scores.emplace(val);
        if (top_k_scores.size() > kHeapSize) {
            top_k_scores.pop();
        }
        return top_k_scores.top();
    }
private:
    const int kHeapSize;
    //min_heap
    std::priority_queue<int, std::vector<int>, std::greater<int>> top_k_scores;
};

```

一旦N個の要素をheapifyする方法

```cpp
class KthLargest {
 public:
  KthLargest(int k, vector<int>& nums)
      : kHeapSize(k), top_k_scores(nums.begin(), nums.end()){};

  int add(int val) {
    top_k_scores.emplace(val);
    while (top_k_scores.size() > kHeapSize) {
      top_k_scores.pop();
    }
    return top_k_scores.top();
  }

 private:
  const int kHeapSize;
  // min_heap
  std::priority_queue<int, std::vector<int>, std::greater<int>> top_k_scores;
};
```
```cpp
top_k_scores(nums.begin(), nums.end(), std::vector<int>, std::greater<int>{})
```
で最後に{}が必要な理由は、コントラクタ引数としてgreaterのオブジェクト（実態）を渡すから
```cpp
std::priority_queue<int, std::vector<int>, std::greater<int>> top_k_scores;
```
はテンプレート引数＝"型"であるため、{}は不要。なお、
```cpp
top_k_scores(nums.begin(), nums.end())
```
のように省略することも可能

抑えたつもりでもSTEP2に3時間弱かかったが、これぐらいが目安かも..
自分としては、もう少しスリムでよい

## STEP3
### 3回ミスなく書く

addを共通化し、コントラクタの時から常に要素をk個に保つ

```cpp
class KthLargest {
 public:
  KthLargest(int k, vector<int>& nums) : kHeapSize(k) {
    for (auto score : nums) {
      add(score);
    }
  }

  int add(int val) {
    top_k_scores.emplace(val);
    if (top_k_scores.size() > kHeapSize) {
      top_k_scores.pop();
    }
    return top_k_scores.top();
  }

 private:
  const int kHeapSize;
  std::priority_queue<int, std::vector<int>, std::greater<int>> top_k_scores;
};
```
```cpp
class KthLargest {
 public:
  KthLargest(int k, vector<int>& nums)
      : kHeapSize(k), top_k_scores(nums.begin(), nums.end()){};

  int add(int val) {
    top_k_scores.emplace(val);
    while (top_k_scores.size() > kHeapSize) {
      top_k_scores.pop();
    }
    return top_k_scores.top();
  }

 private:
  const int kHeapSize;
  std::priority_queue<int, std::vector<int>, std::greater<int>> top_k_scores;
};
```


30分で3回づつAccept(下2回、上1回)

### 2週目の宿題
- priority_queue、heapの実装
- std::greaterのより正確な理解
