---
title: "Claude CodeのBashでgrepがエラーになる、lsで確認が出る原因"
emoji: "🔁"
type: "tech"
topics: ["claudecode", "claude", "grep", "bash"]
published: true
---

> 検証日: 2026-10-15 ／ Claude Code 2.1.270 ／ macOS 13.7.8

Claude Code の Bash ツールに打った `grep` は、そのまま `/usr/bin/grep` に届くとは限りません。
私の環境では、シェル関数が `grep` を ugrep に差し替え、rtk（出力を圧縮する CLI のプロキシ）のフックが `ls` や `jq` を別のコマンドに書き換えていました。
どちらも、打った側が気づかないうちに、検索の件数や許可の確認を変えることがあります。

以下の数字は、2026-09-01 と 2026-09-14 の作業で得たものです。

## 何が起きたか

1つめは 2026-09-01 です。
トランスクリプトを解析していて、Bash ツールで次の `grep` を実行したところ、失敗しました。

```
$ grep -o '.\{0,80\}foo.\{0,40\}' file
ugrep: error: error at position 84
... exceeds complexity limits
```

打ったのは `grep` なのに、エラーは `ugrep` を名乗っています。
同じパターンを `/usr/bin/grep` で実行すると、正常に通りました。
当時の私は、この原因を rtk のフックに帰属させていました。

2つめは 2026-09-14 で、プランモードで作業していたときです。
`ls` `wc` `jq` のような読むだけのコマンドで、実行してよいかの確認が出ました。
一方で `sed -n` や `cut` は、作業フォルダの中なら確認なしで通ります。
どれもファイルを読むだけなので、扱いが分かれる理由が分かりませんでした。

## どう測ったか

`grep` のほうは、同じコマンドを4つの経路で比べました。
経路ごとに外れる層が違うので、どの経路で直るかを見れば、効いている層が分かります。

```
1. grep …           # フックとシェル関数の両方が効く
2. command grep …   # シェル関数だけ外れる
3. /usr/bin/grep …  # 実体を直接呼ぶ
4. rtk proxy grep … # rtk のフィルタだけ外れる
```

シェル関数だけを外す2で直りました。
`type -a grep` を見ると `grep is a function` と出て、シェル関数が注入されていることを確かめました。
rtk は無関係で、私はこの件の原因を rtk のせいだと2回取り違えていました。

`ls` の確認のほうは、`claude -p --permission-mode plan` を Haiku で動かし、出力の `permission_denials` で判定しました。
あわせて `rtk hook check "<コマンド>"` で、rtk がどのコマンドを書き換えるかを1つずつ調べました。

- 書き換えられる: ls、wc、cat と head と tail（`rtk read` に）、grep、jq、tree、stat、du、ps、git branch、gh pr view、diff、curl
- 書き換えられない: sed、cut、sort、awk、file、env、which

確認が出た ls、wc、jq は書き換えられる側に、確認が出なかった sed と cut は書き換えられない側に入っていました。
作業フォルダ内の `jq -r .a f.json` は `rtk jq` に書き換わり、確認が出ました。

## 原因

`grep` が ugrep になっていたのは、Bash ツールのシェルに `grep` という名前のシェル関数があるからです。
この関数は `claude` 本体（`$CLAUDE_CODE_EXECPATH`）を `exec -a ugrep` で起動し、次のオプションを毎回付けます。

```
-G --ignore-files --hidden -I
--exclude-dir=.git --exclude-dir=.svn --exclude-dir=.hg
--exclude-dir=.bzr --exclude-dir=.jj --exclude-dir=.sl
```

`-G` は BRE のモードで、繰り返し回数の指定が2つ以上並ぶと上限に当たります。
冒頭のエラーはこれでした。

ただ、エラーになる場合はまだ気づけます。
`--ignore-files` と `-I` と `--exclude-dir=.git` が常に効くので、`.git` の下、バイナリ、ignore 対象のファイルは、何も言われずに結果から消えます。
`/usr/bin/grep` と件数が食い違っても、エラーは出ません。

`ls` の確認は、rtk の PreToolUse フック（`rtk hook claude`）の返し方から来ていました。
このフックは `updatedInput` でコマンドを書き換えますが、`permissionDecision` は返しません。
そのため Claude Code は、書き換えた後の `rtk ls` で許可を判定します。
`rtk ls` や `rtk jq` は組み込みの読み取り専用のセットに入っていないので、許可ルールが無ければ確認が出ます。

## どう直したか（直せなかったか）

`grep` は、正確さが要る場面では素の `grep` を使わないことにしました。
`.git` の下やバイナリや ignore 対象を含めたいときと、複雑な BRE を使うときです。

```
command grep …       # 関数だけを外す。いちばん短い
/usr/bin/grep …      # 実体を明示する
bash -lc "grep …"    # 別のシェルなので関数が無い
```

関数の側にも抜け道があり、特定のオプションを付けると `command grep` に落ちます[^escape]。

rtk の確認は、2つの手で解消できました。
1つは、rtk の設定（`~/Library/Application Support/rtk/config.toml`）の `[hooks] exclude_commands` に足す手です。
jq、tree、stat、du、git branch を足すと、どれも確認なしで通りました。
組み込みの判定とパス制限がそのまま効く代わりに、そのコマンドの出力は圧縮されなくなります。

もう1つは、`Bash(rtk ls *)` のような許可ルールを足す手です。
こちらは圧縮を保てますが、パス制限は効きません。
私は ls、read、grep、wc、diff をこの方式にし、jq などは除外の方式にしました。
どちらにしても、効いたかは `rtk hook check "<コマンド>"` で確かめられます。

この2つで消えるのは、書き換えが原因の確認だけです。
作業フォルダの外を読むときは、書き換えと関係なく確認が出ます。
`curl` は読み取り専用のコマンドではないので、除外しても確認が出ます。

## 残っている疑問

確かめたのはプランモードだけです。
default モードでも同じ確認が出ると考えていますが、これは推測です。

`updatedInput` で書き換えて `permissionDecision` を返さないフックなら、rtk でなくても同じことが起きると見ています。
こちらも、私が踏んだのは rtk だけです。

冒頭の `position 84` のエラーは、ugrep が名乗り出てくれたぶん、まだ運がよかったと思っています。
同じ仕組みで、ほかのコマンドも差し替えられているかもしれません。
挙動が想定と違うコマンドに当たったら、まず `type -a <コマンド>` で、それが本当に打ったとおりのものかを見ることにしています。

[^escape]: 引数に `-Z` `-z` `--null` `---*` `-@*` などが含まれると、関数は `command grep` に処理を渡します。
