# 929. Unique Email Addresses

https://leetcode.com/problems/unique-email-addresses/description/

## STEP1
- 何も見ずに解いてみる

#### 考えたこと
- 個々のアドレスを整形して標準系にし、setに格納していく
    - ①ドメイン部分かローカル部分か　② "+"以降の無視している部分か　の2つのフラグを管理しながら各アドレスを整形
- 最終的にsetの要素数が答え

計算量
email.length をN, emails[i].length をMとして
- 時間計算量(M*NlogN)
    - N = M = 100の時 66000ステップ 66マイクロ秒と見積り


```cpp
#include <set>
#include <vector>

class Solution {
public:
    int numUniqueEmails(std::vector<std::string>& emails) {
        std::set<std::string> mailing_list;
        for (const auto& adress : emails) {
            mailing_list.insert(formatAddress(adress));
        }
        return mailing_list.size();
    }
private:
    static std::string formatAddress(const std::string& adress) {
        std::string formatted_address;
        bool is_domain = false;
        bool is_ignore = false;
        for (const auto& character : adress) {
            if (is_domain) {
                formatted_address += character;
                continue;
            }
            if (character == '@') {
                is_domain = true;
                is_ignore = false;
                formatted_address += character;
                continue;
            }
            if (is_ignore) {
                continue;
            }
            if (character == '+') {
                is_ignore = true;
                continue;
            }
            if (character != '.') {
                formatted_address += character;
            }
        }
        return formatted_address;
    }
};
```

## STEP2
### プルリクやドキュメントを参照
#### 問題が解けるより他人のコードを読んだりコメントするほうがよっぽど大事
#### 参照したもの

- https://github.com/mura0086/arai60/pull/18/files
- https://github.com/colorbox/leetcode/pull/28/files
- https://github.com/Ryotaro25/leetcode_first60/pull/15/files
- https://github.com/Fuminiton/LeetCode/pull/14/files
- https://github.com/quinn-sasha/leetcode/pull/14

- ドキュメント系
    - 電子メールの構文規則　https://datatracker.ietf.org/doc/html/rfc5322#section-3.4.1
    - 文字列操作関連
        - rfind https://cpprefjp.github.io/reference/string/basic_string/rfind.html
            - 最後に現れる指定文字列を検索
        - find_first_of https://cpprefjp.github.io/reference/string/basic_string/find_first_of.html
            - 指定された文字列中のいずれかの文字が出現する最初の場所を検索
        - substr https://cpprefjp.github.io/reference/string/basic_string/substr.html
            - 部分文字列を取得
        - replace https://cpprefjp.github.io/reference/string/basic_string/replace.html
            - 文字列の一部を置換
        - std::erase https://cpprefjp.github.io/reference/deque/deque/erase_free.html (c++20から)
          
    範囲や要素の指定は様々なので、使っていったりリファレンスを眺めて覚えていこう

teachers' eye
- メンバ関数にしたほうが、依存関係が明確になる　https://github.com/mura0086/arai60/pull/18/files#r2030041280
- 何にconstをつけるのか　https://github.com/colorbox/leetcode/pull/28/files#r1845467005
- 名前の付くものには名前をつけてしまえ　https://github.com/colorbox/leetcode/pull/28/files#r1844244553
- RVO(NRVO)が行われない場合、コピーが発生する可能性　https://github.com/colorbox/leetcode/pull/28/files#r1845466871
- @が無い場合は？ https://github.com/Ryotaro25/leetcode_first60/pull/15/files#r1649824260
- 知らない言語を触るときに確認するポイント　https://github.com/Ryotaro25/leetcode_first60/pull/15/files#r1641792391

#### 感想
- @の前後でlocalとdomainに分けて考えるのが分かりやすい
- 「整形」というニュアンスでformatを使っていたが、比較のための変形というニュアンスがあるcanonicalizeが良いようだ
- いろいろなinvalidな入力をはじくことも考える必要がある
    - 例外処理はエラーメッセージ&停止しないのが良さそうだ
    - シチュエーション的に宛先が重複しない有効な送り先をリストアップしたいと感じたので


#### STEP1以外の手法と感想
- substrやrfindを使う方法
    - 直感的な操作が可能で良いと思った
- 正規表現を使う方法
    - 認知負荷は高いが習得はするべきだと思った

```cpp
#include <set>
#include <vector>

class Solution {
public:
    int numUniqueEmails(std::vector<std::string>& emails) {
        std::set<std::string> unique_emails;
        for (const auto& adress : emails) {
            std::string canonicalized_email;
            canonicalizeEmail(adress, canonicalized_email);
            unique_emails.insert(canonicalized_email);
        }
        return unique_emails.size();
    }
private:
    // stringを返り値にした時のコピーを防ぐため、出力用の空文字列を参照して更新する
    static void canonicalizeEmail(const std::string& email, std::string& blank_for_canonicalized_email) {
        auto at_position = email.rfind('@');
        auto local = email.substr(0, at_position);
        auto plus_position = local.find_first_of('+');
        local = local.substr(0, plus_position);
        std::erase(local, '.');
        auto domain = email.substr(at_position);
        blank_for_canonicalized_email += local + domain;
    }
};

```

## STEP3
### 3回ミスなく書く

```cpp
#include <set>
#include <vector>

class Solution {
public:
    int numUniqueEmails(std::vector<std::string>& emails) {
        std::set<std::string> unique_emails;
        for (const auto& email : emails) {
            std::string canonicalized_email;
            canonicalizeEmail(email, canonicalized_email);
            unique_emails.insert(canonicalized_email);
        }
        return unique_emails.size();
    }
private:
    // stringを返り値にした時のコピーを防ぐため、出力用の空文字列を参照して更新する
    static void canonicalizeEmail(const std::string& email, std::string& blank_for_canonicalized_email) {
        auto at_position = email.rfind('@');
        auto local = email.substr(0, at_position);
        auto plus_position = local.find_first_of('+');
        local = local.substr(0, plus_position);
        std::erase(local, '.');
        auto domain = email.substr(at_position);
        blank_for_canonicalized_email += local + domain;
    }
};
```

6分、5分、6分で3回

#### 2週目の宿題

- いろいろなinvalidな入力を防ぐ関数も実装する
- 文字列操作のメソッドをよりリファレンス等で調べる
- 正規表現について調べる
- 文字操作の実装について調べてみる
