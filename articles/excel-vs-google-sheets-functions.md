---
title: "ExcelとGoogleスプレッドシートで書き方が違う関数20件（公式一覧の突き合わせ）"
emoji: "🔀"
type: "tech"
topics: ["excel", "googlesheets", "spreadsheet", "業務効率化"]
published: true
---

> この記事は AI（Claude）を使って作成し、内容は筆者が確認しました。
> 調べた日: 2026-10-10 ／ 両方の公式の関数一覧と、各関数の公式ヘルプ

Excel で作った表を Google スプレッドシートに移したり、その逆をしたりすると、同じ目的の関数でも名前や引数が違って式が動かないことがあります。
両方の公式の関数一覧を突き合わせて、実務で出会いやすい違いを20件にまとめました。

## 調べ方

次の2つの一覧を 2026-10-10 に取得し、関数名で突き合わせました。

- Excel: [Excel 関数 (アルファベット順)](https://support.microsoft.com/ja-jp/excel/excel-functions-alphabetical)
- Google スプレッドシート: [Google スプレッドシートの関数リスト](https://support.google.com/docs/table/25273?hl=ja)

片方の一覧にしかない関数のうち、日常の集計・文字処理・データ取り込みで使うものを選びました。
引数の違いは、各関数の公式ヘルプの構文で確かめています。
「なし」は、その日の公式一覧に載っていないという意味です。関数は両方とも追加が続いているので、使う前に一覧を見直してください。
統計・財務・工学の専門的な関数は除いています。

## 一覧

| # | やりたいこと | Excel | Google スプレッドシート | 違い |
|---|---|---|---|---|
| 1 | 文字列を区切り文字で分ける | [TEXTSPLIT](https://support.microsoft.com/ja-jp/excel/functions/textsplit-function) | [SPLIT](https://support.google.com/docs/answer/3094136?hl=ja) | 名前が違う。TEXTSPLIT は列と行の区切りを別々に指定できる |
| 2 | 正規表現に一致するか調べる | [REGEXTEST](https://support.microsoft.com/ja-jp/excel/functions/regextest-function) | [REGEXMATCH](https://support.google.com/docs/answer/3098292?hl=ja) | 名前が違う。REGEXTEST は3つ目の引数で大文字・小文字の区別を指定できる |
| 3 | 別のファイルの範囲を読み込む | なし | [IMPORTRANGE](https://support.google.com/docs/answer/3093340?hl=ja) | IMPORTRANGE はスプレッドシートだけ |
| 4 | 文字列をつなぐ（CONCAT） | [CONCAT](https://support.microsoft.com/ja-jp/excel/functions/concat-function) | [CONCAT](https://support.google.com/docs/answer/3093592?hl=ja) | Excel は3つ以上つなげる。スプレッドシートの CONCAT は2つの値だけ |
| 5 | 並べ替えの向きを指定する（SORT） | [SORT](https://support.microsoft.com/ja-jp/excel/functions/sort-function) | [SORT](https://support.google.com/docs/answer/3093150?hl=ja) | Excel は 1（昇順）／-1（降順）、スプレッドシートは TRUE／FALSE |
| 6 | 2つ以上の条件で行を抜き出す（FILTER） | [FILTER](https://support.microsoft.com/ja-jp/excel/functions/filter-function) | [FILTER](https://support.google.com/docs/answer/3093197?hl=ja) | Excel は条件を `*` でつないで1つの引数にする。スプレッドシートは条件を引数に並べる |
| 7 | FILTER で1件もなかったとき | [FILTER](https://support.microsoft.com/ja-jp/excel/functions/filter-function) | [FILTER](https://support.google.com/docs/answer/3093197?hl=ja) | Excel は3つ目の引数（if_empty）で表示を決められ、省くと `#CALC!`。スプレッドシートは `#N/A` |
| 8 | 別の列の値で並べ替える | [SORTBY](https://support.microsoft.com/ja-jp/excel/functions/sortby-function) | [SORT](https://support.google.com/docs/answer/3093150?hl=ja) | スプレッドシートに SORTBY はない。SORT の「並べ替える列」に範囲の外の列を指定する |
| 9 | 並べ替えて上位 n 件を取る | [TAKE](https://support.microsoft.com/ja-jp/excel/functions/take-function)（SORT と組み合わせる） | [SORTN](https://support.google.com/docs/answer/7354624?hl=ja) | スプレッドシートは SORTN の1つで済む。Excel に SORTN はない |
| 10 | 配列の先頭の行だけ残す | [TAKE](https://support.microsoft.com/ja-jp/excel/functions/take-function) | [ARRAY_CONSTRAIN](https://support.google.com/docs/answer/3267036?hl=ja) | スプレッドシートに TAKE はない。ARRAY_CONSTRAIN で行数と列数を指定する |
| 11 | 重複を除いた件数を数える | なし（`COUNTA(UNIQUE(範囲))` で代用） | [COUNTUNIQUE](https://support.google.com/docs/answer/3093405?hl=ja) | COUNTUNIQUE はスプレッドシートだけ |
| 12 | 値の位置を探す | [XMATCH](https://support.microsoft.com/ja-jp/excel/functions/xmatch-function)・[MATCH](https://support.microsoft.com/ja-jp/excel/functions/match-function) | [MATCH](https://support.google.com/docs/answer/3093378?hl=ja) | XMATCH は Excel だけ。スプレッドシートは MATCH を使う |
| 13 | 区切り文字より前の部分を取り出す | [TEXTBEFORE](https://support.microsoft.com/ja-jp/excel/functions/textbefore-function) | なし（[REGEXEXTRACT](https://support.google.com/docs/answer/3098244?hl=ja) で代用） | 例: `-` より前なら `=REGEXEXTRACT(A2, "^([^-]*)-")` |
| 14 | 範囲の末尾の空白行・列を除く | [TRIMRANGE](https://support.microsoft.com/ja-jp/excel/functions/trimrange-function) | なし | TRIMRANGE は Excel だけ |
| 15 | グループごとに集計する | [GROUPBY](https://support.microsoft.com/ja-jp/excel/functions/groupby-function) | [QUERY](https://support.google.com/docs/answer/3093343?hl=ja) | スプレッドシートに GROUPBY はない。QUERY のクエリ言語でグループ集計を書く |
| 16 | Web から値を取り込む | [WEBSERVICE](https://support.microsoft.com/ja-jp/excel/functions/webservice-function)・[FILTERXML](https://support.microsoft.com/ja-jp/excel/functions/filterxml-function) | [IMPORTXML](https://support.google.com/docs/answer/3093342?hl=ja) | Excel は取得と抽出が別の関数。スプレッドシートは IMPORTXML で URL と XPath を一度に書く。WEBSERVICE は Windows の Excel だけで動く（Mac では結果を返さない） |
| 17 | 株価の履歴を取る | [STOCKHISTORY](https://support.microsoft.com/ja-jp/excel/functions/stockhistory-function) | [GOOGLEFINANCE](https://support.google.com/docs/answer/3093281?hl=ja) | 関数も引数の並びも違う |
| 18 | 文章を翻訳する | なし | [GOOGLETRANSLATE](https://support.google.com/docs/answer/3093331?hl=ja) | GOOGLETRANSLATE はスプレッドシートだけ |
| 19 | メールアドレスの形か調べる | なし（[REGEXTEST](https://support.microsoft.com/ja-jp/excel/functions/regextest-function) で代用） | [ISEMAIL](https://support.google.com/docs/answer/3256503?hl=ja) | ISEMAIL はスプレッドシートだけ |
| 20 | 値が2つの値の間にあるか調べる | なし（`AND(A2>=下限, A2<=上限)` で代用） | [ISBETWEEN](https://support.google.com/docs/answer/10538337?hl=ja) | ISBETWEEN は両端を含むかどうかも引数で選べる |

## 移すときに気をつけること

4〜7は、両方に同じ名前の関数があり、中身が違います。
関数名からは気づけないので、移したら次の点を見直してください。

- 4（CONCAT）: Excel で `=CONCAT(A2, B2, C2)` と書いた式は、スプレッドシートの CONCAT の構文（2つの値）に合わない。`&` か TEXTJOIN に書き換える
- 5（SORT）: 並べ替えの向きの指定が 1／-1 と TRUE／FALSE で違う
- 6・7（FILTER）: 条件を2つ以上にするときの書き方と、1件もないときの扱いが違う

残りは片方にしかない関数なので、移した先ではその関数を使えません。
「なし」の行に書いた代わりの式に置き換えてください。

## 参考

- [Excel 関数 (アルファベット順)](https://support.microsoft.com/ja-jp/excel/excel-functions-alphabetical)
- [Google スプレッドシートの関数リスト](https://support.google.com/docs/table/25273?hl=ja)
