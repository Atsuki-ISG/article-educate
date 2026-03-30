![見出し画像](https://assets.st-note.com/production/uploads/images/246744949/rectangle_large_type_2_d1bf57f0979ed049c8a4a4d0a31e252b.png?width=1280)

## note記事をフォーム回答から自動で改善する仕組みを作ってみた
記事の最下部にあるフォームに回答してもらうと、自動で内容を改善する仕組みを作ってみました。

![画像](https://assets.st-note.com/img/1769420221-6FGThfOKNcqs1d3M7LeyJ0tV.png?width=1200)

フォームに回答がされた時に

・GASでGitHubのIssueを追加  
・Issueが追加されたらGitHub ActionsでClaudeが動いて修正案のプルリクエストを作成  
・人間がレビュー

といったフローで動きます。

ただ、「 **マークダウンファイルをそのままnoteに修正アップロード** 」みたいなことはできないので、最後実際にnoteに反映する部分は人間が手で頑張ります。（2重管理で微妙なので [Zenn](https://zenn.dev/zenn/articles/connect-to-github) みたいに自動同期できたら嬉しいんですが）

これを作った背景としましては、記事メディアを続ける中での課題感の一つである「 **あまりフィードバック、要するにコメントがもらえない** 」があります。

動画メディアであるYouTubeは、良くも悪くもコメントのハードルが低く、定性的なフィードバックがもらいやすいです。

一方、記事メディアはコメントする文化があまりないので、noteの場合は「スキ」や「高評価」、それかSNSでたまに見かけるコメントなどを参考にするくらいしかできません。正直私自身も記事にコメントした経験、人生で片手で数えられるくらいの回数しか記憶にないですね。

そんな訳で、今回「匿名で公ではないところで送れる」&「記事が終わったところにすぐにある」ことによってコメントのハードルを下げてみよう！という試みで導入してみるのがこの仕組みです。

記事の説明が足りないことや、「もっとこういうこと書いて欲しかった」というフィードバックもそうですし、ポジティブな感想もあれば、ぜひご入力いただけると嬉しいです。

が、こういうことを言うと微妙な圧力を与えてしまいそうなので、本当に気が向いたらで大丈夫です。先ほど申し上げた通り、私自身も記事メディアでコメントそんなにしないタイプなので、「ちょっと言いたいことがある」のハードルが少し下げられたら良いなくらいの感覚です。

それでは、この先ではどんな風にこの仕組みを作ったかをご紹介していきます。

この仕組みの作り方は、似た **「自分がメンテナンスしているものに」「第3者からフィードバックがある」** という構造のお仕事があれば、応用できると思います。  
例：お問い合わせがあったときにFAQのページやマニュアルを更新

> ⚠️前提知識  
> ・GitHubが扱えること - まだの方は [こちらの動画](https://youtu.be/Uml3jEPWIKo?si=9PQOvhB9dc2d7sBe)[  
> ](https://youtu.be/Uml3jEPWIKo?si=9PQOvhB9dc2d7sBe%EF%BC%89%EF%BF%BC%E3%83%BBGitHub)・GitHub Actionsが扱えること - まだの方は [こちらの記事](https://note.com/ai_saborou/n/nbace3bba9cc5)[  
> ](https://note.com/ai_saborou/n/nbace3bba9cc5%EF%BC%89%EF%BF%BC%E3%83%BB%E4%BD%95%E3%82%89%E3%81%8B%E3%81%AE%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%A8%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%B3%E3%83%88%E3%81%8C%E6%89%B1%E3%81%88%E3%82%8B%E3%81%93%E3%81%A8%EF%BF%BC)・何らかのコーディングエージェントが扱えること[  
> ](https://note.com/ai_saborou/n/nbace3bba9cc5%EF%BC%89%EF%BF%BC%E3%83%BB%E4%BD%95%E3%82%89%E3%81%8B%E3%81%AE%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%A8%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%B3%E3%83%88%E3%81%8C%E6%89%B1%E3%81%88%E3%82%8B%E3%81%93%E3%81%A8%EF%BF%BC)　　まだの方は [Antigravity](https://youtu.be/OBK2HlsOlRU?si=nkrohVu15riZvhFq) か [Claude Code](https://youtu.be/0jKHBND-auw?si=opsowN3Qjy4-3m4j) の動画をご参照ください

## 作り方

このラインより上のエリアが無料で表示されます。

自動化する箇所は次の2つのみです。

1. フォームが入力されたらGitHubのイシューに追加
2. イシューをもとに記事を改善

まず必要なのはGoogle Formです。次の3つの項目を用意しています。

> 1\. 記事タイトル  
> 2\. 分かりづらかったポイント・もっと書いて欲しかったポイントがあれば教えてください  
> 3\. 良かったところを教えてください

記事タイトルは後々AIが編集するときに該当のファイルを見つけるときに必要な項目です。これは事前入力にしてユーザに手間はかけさせないようにします。  
あと2つはフィードバックの項目ですね。これらを基にAIが改善に取り組みます。

また、前提として、note記事の元原稿たちはマークダウンファイルでGitHubレポジトリで管理しています。

構造はものすごいシンプルで、wip(書き途中)とreleased(公開しました)という二つのフォルダーがあって、その中にマークダウンファイルたちが存在するだけです。

![画像](https://assets.st-note.com/img/1769417917-HZtzDNaJk3i1soqETwu7GQLO.png?width=1200)

## フォームが入力されたらGitHubのイシューに追加

それではフォームが準備できたら、GitHubのIssueに追加する自動化を作っていきます。

この自動化にはGASというGoogleのアプリを使った自動化簡単に作れ〜るものを使用します。

[**Apps Script | Google for Developers** *高品質なクラウドベースのソリューションを簡単に開発できます。* *developers.google.com*](https://developers.google.com/apps-script?hl=ja)

実装は以下のプロンプトをClaude Codeに送って書いてもらいます。正直20秒くらいで書いた雑プロンプトではあるのですが、これくらいでも全然実装してくれます。

```javascript
GASでFormの回答からGitHubにイシューを追加するコードを書いて
- フォームの内容は次の3項目だけ。JSONとして記述イシューのテキストを書きこむ
    - 記事 ID
    - 分かりづらかったポイント・もっと書いて欲しかったポイントがあれば教えてください
    - 良かったところもあれば教えてください
```

すると下記のコードが書かれます。  
全部貼ると長過ぎるので詳細は [こちらにアップロード](https://gist.github.com/kazuyaseki/29bd87a74c69d2842a604f8d107a3a23) しておきました。

内容を簡単に解説しますと、フォームの内容を読み取って処理を実行するonFormSubmitという関数がまずあり、その中でcreateGitHubIssueというGitHubでイシューを作成する操作を実行しています。

```javascript
/**
 * Google FormからGitHubにissueを作成するスクリプト
 *
 * 事前設定:
 * 1. スクリプトプロパティに以下を設定
 *    - GITHUB_TOKEN: GitHubのPersonal Access Token
 *    - GITHUB_OWNER: リポジトリのオーナー名
 *    - GITHUB_REPO: リポジトリ名
 *
 * 2. フォームのトリガー設定
 *    - onFormSubmit関数をフォーム送信時のトリガーとして設定
 */

/**
 * フォーム送信時に実行される関数（フォーム直接バインドの場合）
 * @param {Object} e - フォーム送信イベントオブジェクト
 */
function onFormSubmit(e) {
  try {
    // イベントオブジェクトの構造をログ出力（デバッグ用）
    Logger.log('イベントオブジェクト: ' + JSON.stringify(e));

    let articleTitle = '';
    let difficultPoints = '';
    let goodPoints = '';

    // フォーム直接バインドの場合
    if (e && e.response) {
      const itemResponses = e.response.getItemResponses();

      itemResponses.forEach(function(itemResponse) {
        const title = itemResponse.getItem().getTitle();
        const answer = itemResponse.getResponse();

        // デバッグ用ログ（項目名を確認したい場合はコメントアウトを外す）
        Logger.log('項目名: ' + title + ', 回答: ' + answer);

        if (title.includes('記事') && title.includes('タイトル')) {
          articleTitle = answer;
        } else if (title.includes('分かりづらかった') || title.includes('書いて欲しかった')) {
          difficultPoints = answer;
        } else if (title.includes('良かった')) {
          goodPoints = answer;
        }
      });
    }

    // GitHubにissueを作成
    createGitHubIssue(articleTitle, difficultPoints, goodPoints);
  } catch (error) {
    Logger.log('エラーが発生しました: ' + error.toString());
    Logger.log('スタックトレース: ' + error.stack);
    // エラー通知を送りたい場合はここに追加
    throw error; // エラーを再スローして、Apps Scriptの実行履歴に記録
  }
}
```

次にGASのセットアップをしていきます。  
まずGoogle Formの右上からApps Scriptというのをポチッと押します。

![画像](https://assets.st-note.com/img/1769401797-0MXTDQL812EnhO4jNuHfv7It.png?width=1200)

するとGASの画面になるので、AIが書いてくれた先ほどのコードをコピペします。

![画像](https://assets.st-note.com/img/1769401838-gfVl0dFzKyPEQe7BxcapWmsq.png?width=1200)

あと2つ準備するものがありまして、まず一つに **トリガー** というものをセットします。これはフォームの回答が入力された時に、コードを実行するために必要です。

左から時計のマークをクリックしまして、トリガーを追加を右下からクリックします。

![画像](https://assets.st-note.com/img/1769402085-b4NDTtManGz7gc6jKvSB5xHi.png?width=1200)

そして以下のように設定してトリガーを追加します  
基本はデフォルトの選択肢ですが、「イベントの種類を選択」は「フォーム送信時」に変更します。

![画像](https://assets.st-note.com/img/1769402098-CO0RdZwInDSWtHBm3aKYb2uh.png?width=1200)

これでフォームが入力された時に実行されるようになります。試しにフォームからテストメッセージを送ってみましょう。

回答した後は、左のナビの実行ログからちゃんとフォーム回答時に実行されたかを確認することができます。

今回実行はされていたのですが、 **GitHubの情報が不足している** というエラーが出てしまったので、次にこの認証情報をセットしていきます。

![画像](https://assets.st-note.com/img/1769402111-9D4vd6C5oaxRBPFLUSrmAwNG.png?width=1200)

左下の歯車アイコンから設定画面を開き、最下部にある「スクリプト プロパティ」で3つの値をセットします。

・GITHUB\_TOKEN: GitHubのPersonal Access Token  
・GITHUB\_OWNER: GitHubのユーザー名  
・GITHUB\_REPO: リポジトリ名

![画像](https://assets.st-note.com/img/1769402117-H1RYZhSjCsi5NLFd207Gtn9V.png?width=1200)

**Personal Access Token** とは、ざっくり言うと **GitHubにログインするための「合鍵」** のようなものです。

外部のアプリやツールからGitHubに接続するとき、セキュリティ上の理由から通常のパスワードは使えません。代わりに使うのがこのトークンです。

合鍵なので、「読み取りだけ許可」のように権限を制限したり、有効期限を設定したりできます。万が一漏れても、そのトークンだけ無効にすれば本体のアカウントは無事です。

GitHubのPersonal Access TokenはGitHubの Settings > Developer Setting > Personal access tokens から取得できます。以下のURLからも直接行けます。

[https://github.com/settings/personal-access-tokens](https://github.com/settings/personal-access-tokens)

ページが開けたら右上からGenerate new tokenを選択します。

![画像](https://assets.st-note.com/img/1769402129-jK0GQaeVkCFMDgiwXzryR4mE.png?width=1200)

まず名前をつけましょう。有効期限はデフォルトのままでも良いですが、その場合は毎月再生成とセットしないといけないので、有効期限なしにもできます。ちゃんと外部に漏らさないように注意を払えるのであれば、有効期限なしにしても良いでしょう。

![画像](https://assets.st-note.com/img/1769402136-h1fRAwVQKFJuxPlGsLk3paoU.png?width=1200)

そして、noteの記事を管理するレポジトリを選択し、イシューを作れたり消したりする(Read and Write)の権限を付与しましょう。  
ここまでできたら Generate Token を選択します。

![画像](https://assets.st-note.com/img/1769402147-a5iXr4qmRPdQ0cMvsgLtIFzp.png?width=1200)

すると、こんな感じでトークンが表示されるので、コピーして先ほどのGASのスクリプトプロパティに貼り付けましょう。

![画像](https://assets.st-note.com/img/1769402154-fCwuGYUx0I6yTSNsP7OcnVRa.png?width=1200)

それではフォームの回答を送ってテストしてみます。

![画像](https://assets.st-note.com/img/1769402160-1s9KTEQvtoFBOW3P0fL2meGg.png?width=1200)

そして実行すると！無事イシューが作られています。

![画像](https://assets.st-note.com/img/1769402166-CuHB4bMLl7f5vRF3QPhUiS0m.png?width=1200)

では次に、これを元にClaudeに修正をしてもらいます。

## イシューをもとに記事を改善

それでは、イシューが追加されたときに自動で記事を編集してもらうようにします。

実はこれは以前不快メモで作ったものとほぼ同じ仕組みでして、ここで作ったワークフローのプロンプトを変えるだけです。  
詳しいセットアップは「## 発展編: AIに解決策を提案、なんなら実装してもらう」を参考にしてください。ここでは差分だけお見せします。

変えるのはワークフロー内の **prompt: |** の部分のみです。

ポイントとしては

1. 修正が必要かを判断する
2. タイトルを元にファイルを探してもらうこと

です。あとはシンプルにフィードバックの内容をプロンプトに埋め込んで編集してもらいます。  
(こちらも長過ぎるので全体のワークフローファイルは [こちら](https://gist.github.com/kazuyaseki/2bbb8e84102caa160e90f20b648558c5) にアップロードしてあります)

```cs
prompt: |

あなたは優秀なテクニカルライターです。読者からのフィードバックを元に、note記事を改善してください。

## 記事タイトル
${{ steps.extract_title.outputs.article_title }}

## フィードバック内容
${{ github.event_name == 'workflow_dispatch' && 'フィードバックは .github/issue_body.txt を読んでください' || github.event.issue.body }}

## Issue番号
${{ github.event_name == 'workflow_dispatch' && inputs.issue_number || github.event.issue.number }}

---

## タスクの流れ

### ステップ1: フィードバックの解析と編集必要性の判定
1. **フィードバック内容の読み取り**
- Issue本文からJSON形式のフィードバックデータを抽出
- \`articleTitle\`、\`difficultPoints\`、\`goodPoints\`を取得

2. **編集必要性の判定**
以下の条件で判定してください：

**編集が必要なケース（PRを作成）：**
- \`difficultPoints\` に具体的な内容が記載されている
- 分かりづらかったポイントや改善要望がある
- 追加で書いて欲しいという要望がある

**編集が不要なケース（Issueにコメントのみ）：**
- \`difficultPoints\` が空、または「(記載なし)」
- \`goodPoints\` のみに内容がある（ポジティブフィードバックのみ）
- 具体的な改善要望がない  

3. **判定結果の出力**

編集が不要と判断した場合：
- Issueに以下の内容でコメントを投稿

## 📋 フィードバック処理結果
**判定**: 編集不要

**理由**:

- \`difficultPoints\` が空、または具体的な改善要望なし
- ポジティブフィードバックのみ

**対応**: 記事の編集は行わず、このissueをクローズします。
- **ここで処理を終了（PRは作成しない）**  

### ステップ2: 記事ファイルの特定（編集が必要な場合のみ）

1. **ファイルの検索**
- \`released/\` ディレクトリ内で記事タイトルに対応する \`.md\` ファイルを検索
- ファイル名は記事タイトルと完全一致、または類似している可能性がある
- 複数候補がある場合は、最も適切なものを選択

2. **ファイルが見つからない場合**
- Issueにコメントで「該当する記事ファイルが見つかりませんでした」と報告
- 処理を終了

### ステップ3: 記事の編集（編集が必要な場合のみ）

1. **記事の読み込み**
- 特定した記事ファイルを読み込む
- 既存の内容を理解する

2. **フィードバックに基づく編集**
以下の方針で編集してください：

**分かりづらかったポイントの改善：**
- 該当箇所を特定し、より分かりやすい表現に修正
- 必要に応じて説明を追加
- 具体例やコード例を追加
...
```

それでは、試しにフィードバックをフォームで送って改善してもらいます。出来上がったプルリクエストがこんな感じです。

![画像](https://assets.st-note.com/img/1769402219-xVIFgMayK6To4W8QUlsRAd3t.png?width=1200)

![画像](https://assets.st-note.com/img/1769402229-HIRwA69fojQtGpkWxbhFuycC.png?width=1200)

これを元に、良さそうならそのまま取り込みますが、修正が必要だったらClaudeにこう直して欲しいと伝えます。pullして自分で書き換えちゃうのもアリです。

また、AIによる修正の品質を上げていく時に大事なのが自分の文体ルールなどを育てることです。それを読み込んだ上で修正してもらうことによって、Claudeの修正がそのまま取り込める確率が上がります。

こうしたルールの育て方に関しては [こちらの動画](https://youtu.be/SymrYLHxw0s?si=4ws6ci9RDyKJVJAB) で取り扱っているのでぜひ参考にしてみてください。

## 小話: フォームのタイトルの自動埋め込み

最後にAI要素は全くないのですが、フォームを使うときのワンポイントとして、フォームの内容の事前入力の仕方をご紹介します。

まず今回のフォームでは記事のタイトルを入力する必要があります。そうしないと、何の記事を修正したら良いかAIが迷ってしまいます。

ただ、ユーザーにタイトルを入力してもらうのはちょっとありえないというか、余計な一手間をかけさせてしまいますし、ミスをする可能性だってあります。

そこで！Google Formの「フォームの事前入力」を使います。

![画像](https://assets.st-note.com/img/1769402276-SasjW6T7YXrtzldfwcK0Bn9I.png?width=1200)

  
これで事前入力をすると、入力した値が埋め込まれたフォームのURLが取得できます。

![画像](https://assets.st-note.com/img/1769402395-waYemlsOrMkCD5tQEJgxFpAq.png?width=1200)

そのリンクをこのnoteにペタッと貼ったのが以下です。無事タイトルが事前入力されています。

この事前入力の仕組みがどうなっているかというと、URLの後ろにクエリパラメータというものを付けていて、そこで「このフィールドはこの値」という指定がされています。

> 💡 クエリパラメータとは、URLの末尾に「?」を付けて追加する「おまけ情報」のことです。「〇〇=△△」という形式で、Webページに「この情報を使ってね」と指示を渡せます。ネット通販で「サイズ=M、色=青」と注文するようなイメージです。

以下がフォームの実際のURLですが、 \`entry.265761805\` が「記事タイトル」のフィールドを指していて、それが「 **noteを自動で改善する仕組みを作ってみました** 」という値だよと指定している訳ですね。

なので、この「=」部分より後ろを書き換えれば、逐一Google Formにリンクを取得しに行かずとも事前埋め込みのタイトルを変えられます。

![画像](https://assets.st-note.com/img/1769402537-hVOSPFwzaM0trcgDf3WvdxTy.png?width=1200)

以上、自動でnoteの改善をする自動化の作り方のご紹介でした。  
たまたま今回私が作ったのが「記事の改善」という文脈ではありましたが、こういった仕組みはお問い合わせ対応などでも活かせる部分があると思います。

ぜひ似たようなお仕事があれば試してみてください。  
それではまた次回の記事で！