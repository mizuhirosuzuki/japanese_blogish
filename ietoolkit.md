# ietoolkit ってなに？

インターンをしてて気になったのでまとめてみました。
もとのGitHubページは[ここ](https://github.com/worldbank/ietoolkit)です。

要約してしまうと、ietoolkitは世銀のDIME Analyticsという部署が政策効果の測定のためにつくったStataのコマンド集みたいなものです。
基本的には因果推論で使うことがあるコマンド、あとは「ちゃんとフォルダを管理しましょうね」「Git使ってますか？」的なお助けコマンドもあるよ。
ちなみにietoolkitの双子の兄弟は[iefieldkit](https://github.com/worldbank/iefieldkit)で、これはフィールドワークの各過程で便利なコマンドを提供しているそう。
さらにちなみに、ietoolkitはStataに関するGitHubのレポのなかで二番目に星の数が多いそう（2020年6月26日時点）で、一番多いのは[Mostly Harmless Econometricsのreplicationをしているページ](https://github.com/vikjam/mostly-harmless-replication)だそうです。
というわけで、この記事を読んで「役に立ったなー」という方は(ietoolkitのGitHubページ)[https://github.com/worldbank/ietoolkit]にいって星を押しましょう。

# どんなコマンドがあるの？

とりあえず公式のGitHubページにあるのは以下のコマンド:

- *ietoolkit* returns meta info on the version of ietoolkit installed. Can be used to ensure that the team uses the same version.
- *iebaltab* is a tool for multiple treatment arm balance tables
- *ieddtab* is a tool for difference-in-difference regression tables
- *ieboilstart* standardizes the boilerplate code at the top of all do-files
- *iefolder* sets up project folders and master do-files according to DIME's recommended folder structure
- *iegitaddmd* adds placeholder README.md files to all empty subfolders allowing them to be synced on GitHub
- *iematch* is an algorithm for matching observations in one group to the "most similar" observations in another group
- *iegraph* produces graphs of estimation results in common impact evaluation regression models
- *iedropone* drops observations and controls that the correct number was dropped
- *ieboilsave* performs checks before saving a data set

ざっくり訳してしまうと、

- *ietoolkit*: ietoolkitのメタ情報を返すコマンド、チーム内で同じバージョンを使ってるかチェックするのに使う
- *iebaltab*: 処置群（複数でもOK）と統制群の間でちゃんとランダム化が行われているかチェック
- *ieddtab*: 差の差分析の結果を表にしてくれる
- *ieboilstart*: 全部のdoファイルの一番初めに「書いておくべきこと」を書いてくれる
- *iefolder*: DIMEが考える「こういうフォルダ構成にすべき！」というのに従ってプロジェクトフォルダを作って、master do fileも用意してくれる
- *iegitaddmd*: 空のフォルダにREADME.mdファイルを置いてくれる（これをしないと空のフォルダがGitHubで同期されない）
- *iematch*: グループ間の似た者同士をマッチングする
- *iegraph*: 推計結果をよくある感じのグラフにしてくれる
- *iedropone*: drops observations and controls that the correct number was dropped
- *ieboilsave*: データを保存する前にチェックしてくれる

ってな感じです。
以下でコマンドの使用例を紹介するときには、[Impact Evaluation in Practice](https://www.worldbank.org/en/programs/sief-trust-fund/publication/impact-evaluation-in-practice)で使われている Health Inusrance Subsidy Program (HISP) というデータを使います。
ただの例として使うだけなので、出力結果やもろもろの分析方法については気にしないで、「ああ、これを回すとこういう感じの表とか図が出てくるのね」くらいの感じでお楽しみください。

## ietoolkit

このコマンドはietoolkitのパッケージのバージョンを教えてくれます。
なので、
```
ietoolkit
```
と打つと、ぼくのPCではこんな感じででてきます：

<img src="./Figures/ietoolkit/ietoolkit.png?" width="500"/>

例えば master do file のはじめに書いておいて、「このコードを回す人のPCではにちゃんとietoolkitインストールされてますか？」とか「ちょっとバージョン古すぎますよ、アップデートしたほうがいいんじゃないですか？」みたいな確認ができます。
(ちなみに master do file については[ここ](https://dimewiki.worldbank.org/wiki/Master_Do-files) に詳しく書いてあるし、[これ](https://www.poverty-action.org/publication/ipas-best-practices-data-and-code-management)にもちらっと書かれています。)
こういう使い方をするためのコードはietoolkitのヘルプファイルに書いてあって、こんな感じです：

```
cap ietoolkit
if "`r(version)'" == "" {
  *ietoolkit not installed, install it
  ssc install ietoolkit
}
else if `r(version)' < 5.0 {
  ietoolkit version too old, install the latest version
  ssc install ietoolkit , replace
}
```

## iebaltab

このコマンドは、「ちゃんとランダム化されてますか？」というのをチェックするのが基本的な使い方のようです。
なので、「指定した変数の平均がグループ間（例えば介入群と統制群）でどれくらい違うか」の表を出してくれます。
例えばHISPのデータを使って

```
iebaltab age_hh age_sp educ_hh educ_sp, grpvar(treatment_locality) savetex("iebaltab.tex") onerow texnotewidth(0.6) replace
```

と打つと.texで結果を保存できて、こんな感じの表が作られます(texnotewidthというオプションは表のフットノートの幅を設定するもので、これをうまく設定しないとちょっと見た目がダサくなります）。

<img src="./Figures/ietoolkit/iebaltab.png?" width="500"/>

ややフォーマットが気になりますが（T-testではなく$t$-testのほうがいいのでは…？とか）、少なくとも内部で結果を手早く共有したりするのには便利そう。
ちなみに、個人的に気になったフォーマットについては、GitHubのレポをフォークして、自分で編集して、プルリクエストをする予定です。
このレポのIssuesを見てみるとわかるのですが、DIME Analyticsの人たちはユーザーからのフィードバックにすごく丁寧に対応してくれて、こっちの提案も結構受け入れてくれます。
いい人たちや…

このコマンドはオプションがたくさんあって、例えば「固定効果をコントロールした上でどれくらいグループ間の差があるか」「クラスター標準誤差をつかったら差は統計的に有意か」などなど、いろんなことができます。
ちなみに、以下のようにアウトカムの変数（ここではhealth expenditure)を使うことで、「グループごとのアウトカムの平均はどれくらいか」をみて、介入の効果がどれくらいあったかを見せることもできますね。

```
iebaltab health_expenditures, grpvar(treatment_locality) savetex("iebalteb_health.tex") onerow texnotewidth(0.7) replace
```

結果はこんな感じ：

<img src="./Figures/ietoolkit/iebaltab_health.png?" width="500"/>

これならStandard errorよりStandard deviationのほうが見たい気がしますが、そういうときには stdev というオプションがあります。
[この世銀のブログ](https://blogs.worldbank.org/impactevaluations/ie-analytics-introducing-ietoolkit)でもiebaltabは紹介されていますので、こちらもどうぞ。

## ieddtab

これは差の差分析 (difference-in-differences) をして結果を出してくれます。
例えば

```
ieddtab health_expenditures , t(round) treatment(treatment_locality) onerow
```

を走らせるとこんな表がStata上で出てきます：

<img src="./Figures/ietoolkit/ieddtab.png?" width="500"/>

もちろん.texで結果を保存することもできます。
こんな感じで出てきます：

<img src="./Figures/ietoolkit/ieddtab_tex.png?" width="500"/>

このコマンドもオプションが豊富で、コントロール変数を加えたり標準誤差の計算方法を変えたりできます。
いまのところは2×2のデザインにしか使えないみたいです。





