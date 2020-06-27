# ietoolkit ってなに？

インターンをしてて気になったのでまとめてみました。
つかれたー。
もとのGitHubページは[ここ](https://github.com/worldbank/ietoolkit)です。

要約してしまうと、ietoolkitは世銀のDIME Analyticsという部署が政策効果の測定のためにつくったStataのコマンド集みたいなものです。
基本的には因果推論で使うことがあるコマンド、あとは「ちゃんとフォルダを管理しましょうね」「Git使ってますか？」的なお助けコマンドもあるよ。
ちなみにietoolkitの双子の兄弟は[iefieldkit](https://github.com/worldbank/iefieldkit)で、これはフィールドワークの各過程で便利なコマンドを提供しているそう。
さらにちなみに、ietoolkitはStataに関するGitHubのレポのなかで二番目に星の数が多いそう（2020年6月26日時点）で、一番多いのは[Mostly Harmless Econometricsのreplicationをしているページ](https://github.com/vikjam/mostly-harmless-replication)だそうです。

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

訳してしまうと、

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

ってな感じですかね。


## ietoolkit



