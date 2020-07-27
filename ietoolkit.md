# ietoolkit ってなに？

インターンをしてて気になったのでまとめてみました。
もとのGitHubページは[ここ](https://github.com/worldbank/ietoolkit)です。

要約してしまうと、ietoolkitは世銀のDIME Analyticsという部署が政策効果の測定のためにつくったStataのコマンド集です。
基本的には因果推論で使うことがあるコマンド、あとは「ちゃんとフォルダを管理しましょうね」「Git使ってますか？」的なお助けコマンドもあります。
ちなみにietoolkitの双子の兄弟は[iefieldkit](https://github.com/worldbank/iefieldkit)で、これはフィールドワークの各過程で使える便利なコマンドを提供しているそう。
さらにちなみに、ietoolkitはStataに関するGitHubのレポのなかで二番目に星の数が多いそう（2020年6月26日時点）で、一番多いのは[Mostly Harmless Econometricsのreplicationをしているページ](https://github.com/vikjam/mostly-harmless-replication)だそうです。
というわけで、この記事を読んで「役に立ったなー」という方は[ietoolkitのGitHubページ](https://github.com/worldbank/ietoolkit)にいって星を押しましょう。

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
- *ieboilstart*: 「はじめに設定しておくべきこと（使える変数の数とかメモリとか）」を*いい感じに*設定しておいてくれる
- *iefolder*: DIMEが考える「こういうフォルダ構成にすべき！」というのに従ってプロジェクトフォルダを作って、master do fileも用意してくれる
- *iegitaddmd*: 空のフォルダにREADME.mdファイルを置いてくれる（これをしないと空のフォルダがGitHubで同期されない）
- *iematch*: グループ間の似た者同士をマッチングする
- *iegraph*: 推計結果をよくある感じのグラフにしてくれる
- *iedropone*: サンプルをデータから落とす、ただし「いくつのサンプルを落とすか」を前もって指定して、それと違う数のサンプルが落とされそうになったら教えてくれる
- *ieboilsave*: データを保存する前にもろもろのチェックをしてくれる（例えば「ちゃんと一人ひとりにユニークなIDはありますか？」とか）

ってな感じです。
以下で分析用のコマンドの使用例を紹介するときには、[Impact Evaluation in Practice](https://www.worldbank.org/en/programs/sief-trust-fund/publication/impact-evaluation-in-practice)で使われている Health Inusrance Subsidy Program (HISP) というデータを使います。
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

<img src="./Figures/ietoolkit/ieddtab.png?" width="700"/>

もちろん.texで結果を保存することもできます。
こんな感じで出てきます：

<img src="./Figures/ietoolkit/ieddtab_tex.png?" width="500"/>

このコマンドもオプションが豊富で、コントロール変数を加えたり標準誤差の計算方法を変えたりできます。
いまのところは2×2のデザインにしか使えないみたいです。

## ieboilstart

do fileのはじめに、読み込める変数の上限を設定するために "set maxvar 15000" って書いたりしますよね。
この ieboilstart っていうコマンドは、 *ベストプラクティス* な設定を勝手にしてくれます。

例えば

```
ieboilstart, versionnumber(15.0)
```

と打つと、こんなメッセージが出てきます：

<img src="./Figures/ietoolkit/ieboilstart.png?" width="700"/>

このコマンドのあとに `r(version)' と書いておけば、バージョンも 15.0 として設定されます。
注意メッセージにある通りうまく機能しない可能性もあるみたいですが、何人かでコードを書いたり回したりするときはこういう設定をしないと「コードが回らない！」ということになりかねないので大事ですね。
あと、DIME Analyticsは「レプリケーションできるようにコードを書く」というのに力をいれていて、その点からも「他の人がちゃんと結果を再現できるようにこういう設定をしましょう」ということでもあるのでしょう。

## iefolder

このコマンドは、DIMEの考える「フィールドワークでデータを集めるときにはプロジェクトのフォルダ構成はこうあるべき！」というのに従ってフォルダを作ってくれるものです。
新しいプロジェクトを作るときは、例えば *ProjectABC* というフォルダがあるとしたら

```
iefolder new project, projectfolder("ProjectABC")
```

として、そうすると↓みたいな構成でフォルダやファイルが作られます：

<img src="./Figures/ietoolkit/iefolder1.png?" width="700"/>

この時点では *EncrypteData* と *MasterData* のフォルダはからっぽ。
*Project_MasterDofile.do* は master do file のいい例になってます。
*global_setup.do* はいろいろなグローバルな設定＝複数のファイルで使う設定をしておく場所で、「ポンドからキロ」「エーカーからヘクタール」などの変換の設定がしてあるところがちょっとおもしろいです。

で、つぎに「調査は家計単位」という場合には

```
iefolder new unitofobs household, projectfolder("ProjectABC")
```

を回すと↓みたいにフォルダが追加されます：

<img src="./Figures/ietoolkit/iefolder2.png?" width="700"/>

*EncryptedData* の中にはサンプルされる家計のリスト、介入を受ける家計のリスト、調査の結果得られたデータを保存しますが、これらは名前、住所、電話番号など超プライベートな情報（Personally Identifiable Information: PII)を含むので、ちゃんと暗号化して保存しましょう。
このへんの情報は[こちら](https://dimewiki.worldbank.org/wiki/Master_Data_Set#Back_to_Parent)でざっくり説明されています。
こういったデータからPIIを除いたもの(de-identified data)を *MasterData* に保存します。
ちなみに、ここで出来た *Master household Encrypted* と *household* へのパスは勝手に *Project_MasterDofile.do* にグローバル変数として追加してくれます。

あとはラウンドごと（ベースライン、ミッドライン、エンドライン）のデータ収集、異なる単位（村、家計、個人）でのデータ収集など、いろいろなケースに合わせてサブフォルダを作ってプロジェクトフォルダをきれいに管理するのをお手伝いしてくれるみたいです。
この辺も機能がたくさんあるので、詳しくはヘルプファイルか[こちら](https://dimewiki.worldbank.org/wiki/Iefolder)までどうぞ。

## iegitaddmd

GitHubはフォルダに何のファイルも保存されてないからっぽの状態だと、そのフォルダを同期してくれない困ったさんなんですね。
なので、ある人が「このコードを回してこのファイルをこのフォルダに保存しておこーっと」といっても、そのフォルダが空っぽだとGitHubはそれを無視してしまうので、別の人がそのコードを回したときに「は？そんなフォルダねーよ」というエラーメッセージに泣かされてしまいます。
そういうことがないように、iegitaddmdというコマンドは空のフォルダに「適当なファイル」を入れておいてくれます。
この「適当なファイル」は README.md なので実はあまり適当ではなくて、どこかのタイミングでここに「このフォルダの目的」みたいなものを書いておくのがいいです。

コマンドとしては

```
iegitaddmd, folder(/Path/to/Projectfolder)
```

という感じです。
ちなみにデフォルトの README.md は[こんな感じ](./Figures/ietoolkit/README.md)。

## iematch

これは指定した変数をもとに「統制群の人に似た人を介入群から連れてくる」コマンドです。
「使う？」と聞かれると「…」となるので、とりあえず「こういうのもあるよ」という紹介だけ…

## iegraph

これは回帰分析の結果をもとに「統制群のアウトカム ＋ 介入効果」を計算して、統制群のアウトカムと合わせて棒グラフにしたものです。
ちょっとこの辺の言葉遣いや解釈はうっかり地雷を踏みかねないので、単に結果だけ…

```
reg health_expenditures treatment_locality
iegraph treatment_locality , yscale(r(0 30)) ylabel(0(5)30)
```

で↓みたいなグラフが出てきます：

<img src="./Figures/ietoolkit/iegraph.png?" width="500"/>

Stataっぽい図ですね…
コマンド1行目の回帰分析でコントロール変数を加えたりクラスター標準誤差を使ったり、いろいろできます。

## iedropone

これはStataの drop コマンドと同じように特定のサンプルをデータから落とすために使います。
ただし違いは「いくつのサンプルを落とすか」を指定する点で、例えば「Household IDが123456の家計を1つ落とします」として、Household IDが123456の家計が2つあったら、「なんか変だよ！」と教えてくれます。
使い方としては、例えばHousehold IDが123456の家計のデータに問題があったとき、Aさんは「よし、これはデータから落とそう」ということで "drop if hhid == 123456" と書いて、Bさんは「これはIDを999999にしよう」ということで "replace hhid = 999999 if hhid == 123456" と書いてしまうと、Aさんのコードを回したときにStataは「ん、hhidが123456の人はいないね、じゃあ次いきまーす」とエラーメッセージを吐かずにずんずん進んでいってしまいます。
でも、 ”iedropone if hhid == 123456, numobs(1)" としておけば、「このコマンドで1つのサンプルが落とされないとエラーを出しますよ」ということになるので、プログラムはちゃんと止まってくれます。
ちょっと使い所はむずかしい気もするけど、フィールドワーク中とかには大事なのかもしれないですね。

## ieboilsave

このコマンドは、データを保存する前に「このデータ、ひとつひとつのサンプルにユニークなIDが与えられていますか？」をチェックしてくれます（他にも機能はあるけど、これが一番大事）。
例えば農家家計調査でプロット単位のデータだったら、「家計−プロット」ごとにユニークなIDを定義しましょう、ということです。
「家計IDとプロットIDでユニークに特定できるんだからいいじゃん」というのもベストプラクティスではないようです（mergeの時とかに困るからかな）。

例えばHISPのデータでは、各家計が2ラウンドで調査されているので、「家計−ラウンド」でユニークなIDを作れますね。
なので、

```
gen str5 hhid_str = string(household_identifier, "%05.0f")
gen str1 round_str = string(round, "%01.0f")
gen unique_id = hhid_str + round_str
```

でユニークなID unique_id を作って、

```
ieboilsave, idvarname(unique_id) missingok
```

でエラーメッセージが出ないかチェックします。
missingok というオプションは、「本来は欠損値を単に "." とするんじゃなくて、欠損の理由に合わせて ".a", ".b", ... とするのがベストプラクティスだけど、まあそれは時間かかるしOKとしましょう」、というものです。

# おわりに

今あるコマンドは以上ですが、DIME Analyticsでは今あるコマンドの改良＋新しいコマンドの開発を進めているようです。
というわけで、ちょっとでも役に立ったなーという方は[ietoolkitのGitHubページ](https://github.com/worldbank/ietoolkit)にいって星を押しましょう。

