# Stata linter について

最近、世銀のDIME Analyticsという部署がStataのlinterを公開しました。
一言でいえば、「Stataのコードを読みやすくしてくれるツール」です。
（詳しくは[こちらのブログ](https://blogs.worldbank.org/impactevaluations/stata-linter-produces-stata-code-sparks-joy)と[GitHubレポ](https://github.com/worldbank/stata-linter)をご覧くださいませ。）
私もすこーし開発に関わっていて、せっかくなのでちょっと記事を書いてみました。
（この記事内の意見についてはあくまで私個人の意見であって、他の著者たちや作成したチームの意見ではないことにご留意ください。）

## どうして読みやすいコードが大事なの？

世銀のブログに”Good code is both correct (produces the intended output) and easily comprehensible to someone who has never seen it before”とある通り、読みやすさは「良いコード」の必要条件だと言えると思います。
「ミスを減らす」「他の人とコラボレーションをするときのコミュニケーションコストを減らす」といったことに加えて、最近注目が高まっている「研究の再現性」の観点で言えば、「結果を再現しようとしている人の負担を減らす」という利点もありそうです。
私が他の人の研究のreproductionをしているときも、`for loop`が何重にもネストされていて、かつインデントがまったくされていなかったりすると（↓こんな感じ）、何が起こっているのか理解するのにとても苦労します。

```
* インデントして...
* iとかjとか無意味な文字をlocal nameに使わないで...
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

世銀のDIME Analyticsという部署がつくったStataのパッケージで、「コードのこの部分、書き方よくないよ」と指摘してくれる機能や、コードを読みやすいように直してくれる機能があります。
このパッケージ内のBest practiceの基準は、あくまでDIME Analyticsが採用している基準で、他の人・組織が採用しているBest practiceの基準とは異なる部分も多いと思います。
もし「自分の書き方のほうが読みやすい」というのであればこのパッケージで使われている基準に沿う必要は全くないと思いますが、おそらく「現状、多くのStataのコードは読みにくく、このパッケージで使われている基準に従うことで読みやすさを改善できる」という想定のもとでパッケージが開発されていると思います。
DIME Analyticsの提供するStataコードの書き方の基準は[こちら](https://worldbank.github.io/dime-data-handbook/coding.html#the-dime-analytics-stata-style-guide)。

## どうやって使うの？

基本的には`ssc install stata_linter`でパッケージをインストールできます。
ただし、裏ではPythonでStataのコードを（テキストファイルとして）処理しているので、Pythonのインストールと使われるパッケージのインストールも必要になります。
詳しくは[こちら](https://github.com/worldbank/stata-linter#requirements)まで。

## なにができるの？

主に”Detection”と”Correction”という２つの機能があります。

### Detection

Detectionは、コード内のbad practiceを見つけてお知らせしてくれます。
GitHubページにある例を使うと（本当は手元で実際にコードを試してみたかったのですが、私のStataライセンスが失効していて出来ませんでした。無念…）、例えばこのレポ内の[test/bad.do](https://github.com/worldbank/stata-linter/blob/master/test/bad.do)に対して

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
２つ目は、「`for loop`とかのlocal nameが`i`とか`x`だと何に対してループしてるのかわからないから、`country`とか`name`とか具体的な名前を使うべき」という基準に基づき、コード内の何箇所でアルファベット一文字の（無意味な）local nameがループに使われているか教えてくれます。
（これらの項目に反対の人もいるでしょうが、上で言ったように、多くのスクリプトはこれらに従うことで読みやすくなると思います。
それぞれの項目についての詳しい説明は[こちら](https://github.com/worldbank/stata-linter#coding-practices-to-be-detected)まで。）

コードのどの部分に問題があるかを知りたい場合は、
```
lint "test/bad.do", verbose
```
とすればOKです。

### Correction

Correctionは指定したStataコードを読みやすくしてくれます。
例えば、
```
lint "test/bad.do" using "test/bad_corrected.do"
```
とすると、`test/bad.do`の問題のある部分を直して、`test/bad_corrected.do`として保存してくれます。
大事な点ですが、**常にオリジナルのスクリプトは保存しておくことと、correction後のファイルで元の結果が再現できることを確認することが強く推奨されています**。
linterも完璧ではないので、correctionがうまくされず、元のスクリプトの結果と異なるものを出力してしまうスクリプトになってしまうかもしれません。
なので、Correctionを過信しすぎず、常にCorrection前後の結果が整合的かをチェックする必要があります。
アグレッシブにコードを直そうとすることでコードの意図を変えてしまうことを防ぐために、CorrectionはDetectionが指摘する問題点のうちのいくつかのみを直すものになっています（そのリストは[ここ](https://github.com/worldbank/stata-linter#coding-practices-to-be-corrected)）。

Correctionをつかうと、例えば

```
if something ~= 1 & something != . {
do something
if another == 1 {
do that
}
}
```
は
```
if something ~= 1 & something != . {
  do something
  if another == 1 {
      do that
  }
}
```
になります。
ちゃんとインデントされていると読みやすいね！

## その他

### おすすめのワークフロー

上に書いたように、Correctionはコードを直してくれるけど、バグを引き起こす可能性もはらんでいます。
一方、Detectionはコードを自動で直してくれはしないけど、コードのどの部分を直せばいいかを教えてくれます。
それを踏まえて、おすすめのワークフローが[ここに書かれています](https://github.com/worldbank/stata-linter#recommended-use)。
まとめると、
1. まずDetectionをつかってどのくらい問題があるかを見てみる
2. もしそんなに問題がないようだったら、手動でコードを直す
3. もしたくさん問題点がみつかったら、Correctionを使ってコードを直すけど、出力結果が変わらないことはチェックする。
4. もう一度Detectionを使って、どれくらいの問題点が残っているかを確認し、その問題点を手動で直す

という感じです。

### バグの報告

自分も作成に関わっておりながらこんなことを言うのはなんですが、多分バグがたくさん発見されるんじゃないかなと予想しています。
これは主に、Stataのコードの書き方が人それぞれすぎるのと、書き方によってエッジケースがいくらでも生まれそうだと思っているからです。
もし問題が見つかったら、[ここ](https://github.com/worldbank/stata-linter/issues)で報告すると、将来バグが直されるかもしれません。
こういうののバグの報告は「パッケージを開発してくれてありがとう！」という感謝の気持ちを伝えることだと個人的には思っているので、バグを見つけても、寛容に、報告をしてもらえると嬉しいです。

### レポにいいね

もし実際に使ってみて便利だなと思ったら、このパッケージのレポにいってStarを押してもらえると、たぶん（私含め）いろんな人が喜びます。

