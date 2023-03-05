# Stata linter について


（詳しくは[こちらのブログ](https://blogs.worldbank.org/impactevaluations/stata-linter-produces-stata-code-sparks-joy)と[GitHubレポ](https://github.com/worldbank/stata-linter)をご覧くださいませ）
この記事内の意見についてはあくまで私個人の意見であって、他の著者たちや作成したチームの意見ではないことにご留意ください。

## どうして読みやすいコードが大事なの？

世銀のブログに”Good code is both correct (produces the intended output) and easily comprehensible to someone who has never seen it before”とある通り、読みやすさは「良いコード」の必要条件だと言えると思います。
「ミスを減らす」「他の人とコラボレーションをするときのコミュニケーションコストを減らす」といったことに加えて、最近注目が高まっている「研究の再現性」の観点で言えば、「結果を再現しようとしている人の負担を減らす」という利点もありそうです。
私が他の人の研究のreproductionをしているときも、`for loop`が何重にもネストされていて、かつインデントがまったくされていなかったりすると（↓こんな感じ）、何が起こっているのか理解するのにとても苦労します。

```
* インデントして...
* iとかjとか無意味な記号を使わないで...
* =とかの前後にスペースをつけて...
foreach i of var x1 x2 x3 {
gen v_`i'=`i'^2
foreach j of var y1 y2 y3 {
gen v_`i'_j`=`v_`i'+`j'
}
replace v_`i'=`i'*2
}
```

## Stata_linterってなに？

世銀のDIME Analyticsという部署がつくったStataのパッケージで、「コードのこの部分、書き方よくないよ」と指摘してくれる機能や、コードを読みやすく直してくれる機能があります。
このパッケージ内のBest practiceの基準は、あくまでDIME Analyticsが採用している基準で、他の人・組織が採用しているBest practiceの基準とは異なる部分も多いと思います。
もし「自分の書き方のほうが読みやすい」というのであればこのパッケージで使われている基準に沿う必要は全くないと思いますが、おそらく「現状、多くのStataのコードは読みにくく、このパッケージで使われている基準に従うことで読みやすさを改善できる」という想定でパッケージが開発されていると思います。

## どうやって使うの？

基本的には`ssc install stata_linter`でパッケージをインストールできます。
ただし、裏ではPythonでStataのコードを（テキストファイルとして）処理しているので、Pythonのインストールと使われるパッケージのインストールも必要になります。
詳しくは[こちら](https://github.com/worldbank/stata-linter#requirements)まで。

## なにができるの？

主に”Detection”と”Correction”という２つの機能があります。

### Detection

Detectionは、コード内のbad practiceを見つけてお知らせしてくれます。
GitHubページにある例を使うと（本当は手元で実際にコードを試してみたかったのですが、私のStataライセンスが失効していて出来ませんでした…）、例えばこのレポ内の[test/bad.do](https://github.com/worldbank/stata-linter/blob/master/test/bad.do)に対して

```
lint "test/bad.do"
```

とすると、

```
-------------------------------------------------------------------------------------
Bad practice                                                          Occurrences                   
-------------------------------------------------------------------------------------
Hard tabs used instead of soft tabs:                                  Yes       
One-letter local name in for-loop:                                    3
Non-standard indentation in { } code block:                           7
No indentation on line following ///:                                 1
Missing whitespaces around operators:                                 0
Implicit logic in if-condition:                                       1
Delimiter changed:                                                    1
Working directory changed:                                            0
Lines too long:                                                       5
Global macro reference without { }:                                   0
Use of . where missing() is appropriate:                              6
Backslash detected in potential file path:                            0
Tilde (~) used instead of bang (!) in expression:                     5
-------------------------------------------------------------------------------------
```

という感じで結果が出てきます。
例えば１つ目は、「インデントにはTabじゃなくてSpaceを使いましょう（Tabだと設定によってスペース２つ分になったり４つ分になったりするので、コードが読みにくくなる可能性がある）」という基準に基づいて、コード内でTabが使われているかを教えてくれます。
２つ目は、「`for loop`とかのlocal nameが`i'とか`x'だと何に対してループしてるのかわからないから、countryとかnameとか具体的な名前を使うべき」という基準に基づき、コード内の何箇所でアルファベット一文字の（無意味な）local nameがループに使われているか教えてくれます。
（これらの項目に反対の人もいるでしょうが、上で言ったように、多くのスクリプトはこれらに従うことで読みやすくなると思います。）

コードのどの部分に問題があるかを知りたい場合は、
```
lint "test/bad.do", verbose
```
とすればOKです。

## その他

