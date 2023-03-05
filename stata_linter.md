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
replace v_`i'_`j'=v_`i'_`j'/v_`i'
}
```

## Stata_linterってなに？

世銀のDIME Analyticsという部署がつくったStataのパッケージで、「コードのこの部分、書き方よくないよ」と指摘してくれる機能や、コードを読みやすく直してくれる機能があります。
このパッケージ内のBest practiceの基準は、あくまでDIME Analyticsが採用している基準で、他の人・組織が採用しているBest practiceの基準とは異なる部分も多いと思います。
もし「自分の書き方のほうが読みやすい」というのであればこのパッケージで使われている基準に沿う必要は全くないと思いますが、おそらく「現状、多くのStataのコードは読みにくく、このパッケージで使われている基準に従うことで読みやすさを改善できる」という想定でパッケージが開発されていると思います。

## どうやって使うの？

## なにができるの？

## その他

