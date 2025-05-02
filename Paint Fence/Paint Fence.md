# Paint Fence
https://leetcode.com/problems/paint-fence/description/

```
問題文
You are painting a fence of n posts with k different colors. You must paint the posts following these rules:

Every post must be painted exactly one color.
There cannot be three or more consecutive posts with the same color.
Given the two integers n and k, return the number of ways you can paint the fence.

Example 1:
Input: n = 3, k = 2
Output: 6
Explanation: All the possibilities are shown.
Note that painting all the posts red or all the posts green is invalid because there cannot be three posts in a row with the same color.

Example 2:
Input: n = 1, k = 1
Output: 1

Example 3:
Input: n = 7, k = 2
Output: 42

Constraints:

1 <= n <= 50
1 <= k <= 10^5
The testcases are generated such that the answer is in the range [0, 2^31 - 1] for the given n and k.
```

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 動的計画法を用いたい paint_ways[i][j] : i番目までの壁について、i番目の壁で2連続同じ色をつかったどうかがj(true/false)であるときの塗り方の場合の数
- 遷移は、今の壁がもう既に2連続で同じ壁を塗っている場合
    - 次の壁は今の色以外（k-1通り）で塗る他ない
- 今の壁が2連続で同じ色ではない場合
    - 次の壁を、今の壁と色と同じ色（1通り）で塗る
    - 次の壁を、今の壁と違う色（k-1通り）でぬる
- の2パターン

計算量
- 時間計算量 O(N) ステップ数2N
- 空間計算量 O(N)

```cpp
class Solution {
  public:
    int countWays(int n, int k) {
        if (n == 1) {
            return k;
        }
        std::vector<vector<int>> paint_ways(n, vector<int>(2));
        // paint_ways[i][j] : i番目までの壁について、i番目の壁で2連続同じ色をつかったどうかがj(true/false)であるときの塗り方の場合の数
        paint_ways[0][0] = k;
        paint_ways[0][1] = 0;
        for (int i = 0; i < n - 1; i++) {
            paint_ways[i + 1][0] = (paint_ways[i][1] + paint_ways[i][0]) * (k - 1);
            paint_ways[i + 1][1] = paint_ways[i][0];
        }
        return paint_ways[n - 1][0] + paint_ways[n - 1][1];
    }
};
```



## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/tarinaihitori/leetcode/pull/30/files
- https://github.com/Fuminiton/LeetCode/pull/30/files
- https://github.com/Ryotaro25/leetcode_first60/pull/33/files


- ドキュメント系

teachers' eye
- vectorはヒープ領域に動的にメモリを確保するので、使用しなくて良いなら、そちらの方がパフォーマンスが高い　https://github.com/Ryotaro25/leetcode_first60/pull/33/files#r1744858499
- 意味のある変数名 https://github.com/Fuminiton/LeetCode/pull/30/files#r2037026276
    - fence_indexは良い変数名だと思った
-　式の意味 https://github.com/Fuminiton/LeetCode/pull/30/files#r2038226406


#### 感想
- 1次元DPでの手法、自分にはかなり難しくみえた。自分の中で分けて考えているもの（その壁を2連続で塗っているか否か）をまとめて処理できるんだ、不思議だな。というイメージ
- 1つ前の場合の数のうち、その壁を2連続で塗っている時の場合の数が、2つ前の壁の場合の数そのもの（なぜなら遷移が1通りしかないため）が理由とは理解している
- DPの配列の命名方法はみなさん悩んでいるようだ。特に2次元だと[0],[1]のような表現も出てきてしまう。今回はコメントで対応することにした。


#### STEP1以外の手法と感想
- DPはメモ化再帰と類似する（既に分かっている情報を記録しておいてそれを利用することで計算量を削減できる）という話を聞いたことがあるが、これを往々するイメージがなかった。ただ、1次元DP的な発想がないとこの手法はできない。
- メモ化再帰としてはmapを作ってメモを更新していく方法があるが、pythonのrlu_cacheのようなデータ構造を作成して使う方法もあるようだ
    - rlu_cacheは実装が大変そうなので2周目の宿題にする
    - メモ化しないと2^nぐらいの計算量になるので n = 50が上限だと、メモ化は必須
    - 再帰上限はn = 50では問題なし
- 2状態を変数に持って、それを更新していく方法が、空間計算量もO(1)に削減
    - 型に当てはめればinline_DPによるメモリ削減、ということになるけど、そういう話ではないだろう。もっと素直に考えるとこれが自然
    - 自分はこれが一番いいと思う。
    - このように書けるのは、今の状態が、特定の状態に（今回は、1つ前）のみ依存するからであると理解している


トップダウンの方法(メモ化再帰)
```cpp
#include <map>

class Solution {
  public:
    int countWays(int n, int k) {
        if (n == 0 || k == 0) {
            return 0;
        }
        // 1-index
        std::map<int, int> index_to_paint_ways;
        return CountWaysHelper(n, k, index_to_paint_ways);
    }
    int CountWaysHelper(int fence_index, int num_colors, std::map<int, int>& index_to_paint_ways) {
        if (fence_index == 1) {
            return num_colors;
        }
        if (fence_index == 2) {
            return num_colors * num_colors;
        }
        if (index_to_paint_ways.contains(fence_index)) {
            return index_to_paint_ways[fence_index];
        }
        index_to_paint_ways[fence_index] = (CountWaysHelper(fence_index - 1, num_colors, index_to_paint_ways) + 
                                            CountWaysHelper(fence_index - 2, num_colors, index_to_paint_ways)) *
                                            (num_colors - 1); 
        return index_to_paint_ways[fence_index];
    }
};
```

```cpp
        index_to_paint_ways[fence_index] = (CountWaysHelper(fence_index - 1, num_colors, index_to_paint_ways) + 
                                            CountWaysHelper(fence_index - 2, num_colors, index_to_paint_ways)) *
                                            (k - 1); 
```
ここはどのように表記するのが一番わかりやすいだろう。
```cpp
        index_to_paint_ways[fence_index] = (CountWaysHelper(fence_index - 1, num_colors, index_to_paint_ways) 
                                          + CountWaysHelper(fence_index - 2, num_colors, index_to_paint_ways))
                                          * (k - 1); 
```

こっちの方が気持ち見やすいだろうか。


1次元DP
```cpp
#include <vector>

class Solution {
  public:
    int countWays(int n, int k) {
        if (n == 0 || k == 0) {
            return 0;
        }
        if (n == 1) {
            return k;
        }
        std::vector<int> paint_ways(n);
        paint_ways[0] = k;
        paint_ways[1] = k * k;
        for (int fence_index = 2; fence_index < n; fence_index++) {
            paint_ways[fence_index] = (paint_ways[fence_index - 1] + paint_ways[fence_index - 2]) * (k - 1);
        }
        return paint_ways[n - 1];
    }
};
```

"その壁を前の壁と同じ色で塗っている時の場合の数"と、"その壁を前の壁と異なる色で塗っている時の場合の数"　の2つの変数を持つ方法
```cpp
class Solution {
  public:
    int countWays(int n, int k) {
        if (n == 0 || k == 0) {
            return 0;
        }
        int paint_ways_of_same_color = 0;
        int paint_ways_of_different_color = k;
        for (int fence_index = 1; fence_index < n; fence_index++) {
            int temp = paint_ways_of_different_color;
            paint_ways_of_different_color = (paint_ways_of_same_color + 
                                             paint_ways_of_different_color)
                                           * (k - 1);
            paint_ways_of_same_color = temp;
        }
        return paint_ways_of_same_color + paint_ways_of_different_color;
    }
};
```

複数行にわたる四則演算の書き方が、何が一番見やすいかまだ分からない

## STEP3
### 3回ミスなく書く

```cpp
class Solution {
  public:
    int countWays(int n, int k) {
        if (n == 0 || k == 0) {
            return 0;
        }
        int paint_ways_of_same_color = 0;
        int paint_ways_of_different_color = k;
        for (int i = 0; i < n - 1; i++) {
            int temp = paint_ways_of_different_color;
            paint_ways_of_different_color = (paint_ways_of_same_color + 
                                             paint_ways_of_different_color)
                                           * (k - 1);
            paint_ways_of_same_color = temp;
        }
        return paint_ways_of_same_color + paint_ways_of_different_color;
    }
};
```

5分,4分,4分で3回Accept

#### ２周目の宿題
- lru_cahceの実装とこの問題への応用

