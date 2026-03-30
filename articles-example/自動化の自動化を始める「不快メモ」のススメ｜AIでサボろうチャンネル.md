![見出し画像](https://assets.st-note.com/production/uploads/images/241055632/rectangle_large_type_2_c3a9b486af9c065f5e83063489c5b9f1.png?width=1280)

## 自動化の自動化を始める「不快メモ」のススメ
自動化を習慣化するのは難しいことです。

弊チャンネルでは自動化の民主化を掲げて日々AI活用の情報を発信しているわけですが、そんな私でも、「 **これもっと早く自動化しとけばよかったなぁ、なんでずっと手動でやってたんだろう** 」みたいな気持ちになることは多々あります。

自動化から気持ちが遠ざかってしまう理由はいくつかあります。

- それをどう自動化すればいいのか、Howを知らない
- なんとなく実現できそうなイメージは持っているが時間がかかりそうな予感がする
- シンプルに忙しい

ですが、私が上記のような気持ちになった課題たちは、結果論としては自分はその実現方法を知っていたり、やってみると「意外と簡単だったな…」という気持ちになったりしました。

また、ずっと自動化への時間を確保できないほどに追い込まれていたわけでもありません。むしろさっさと自動化をしなかったことにより忙しくなってしまっていました。

つまりは、純粋に自動化手法を学ぶことや、時間を確保すること以外にも、 **着実に自動化に向き合うための仕組み** が必要そうです。

言い換えると、「自動化に取り組むこと」自体を自動化する **自動化の自動化** が必要なのではないかと。

そこで、最近「 **不快メモ** 」という取り組みを始めました。めんどくさいと感じたことをメモするだけで、AIが勝手に解決策を考えてくれる。この仕組みを作ったことで、自動化のループがちゃんと回るようになってきました。

ざっくり言うと

1. とりあえず不快に感じたこと、面倒に思ったことをぼやく
2. GitHub ActionsでClaude Codeが解決策を自動で教えてくれる or 簡単に実装できそうだったら作ってもらう

と言うシンプルな仕組みです。

![画像](https://assets.st-note.com/img/1767427267-eqVkGjzlOQMbRw2ouaYFx60I.png?width=1200)

本記事では、そんな取り組みを紹介していきます。

## 不快メモとは

それでは、私が実践した解決策である不快メモについて解説します。  
不快メモというのは、もう名前の通り過ぎますが、 **不快に感じたことをメモ** していくことです。

下記のようにシンプルで良いのでとりあえずメモしていきましょう。  
めんどいめんどい言ってて、私がめんどくさそうなやつに思えるかもしれないですが、素直な気持ちを書いていきましょう。

```
音声入力後の校正がめんどい
領収書(物理)の管理がめんどい
claude codeの起動になぜか時間がかかる
AIを待っている間についXやYouTube Studioを見てしまう
自分の動画のURLを概要欄に持ってくるのめんどい
スライドでサムネ貼る時に逐一アーカイブから探すのめんどい
ツール入門動画の撮影と編集がめんどい
```

メモを書く先はどこでも良いですが、後半で解説する「メモから自動化」まで見据えるのであれば、

- GitHubのイシュー(これが何かについて後ほど解説します)
- スプレッドシート
- Notionのテーブル
- Slackなどのチャットアプリ

などで、後々自動化のトリガーにしやすい場所に置いてあるのが良いでしょう。ただ一旦は自分が普段使っているメモアプリに箇条書きしていくとかで良いと思います。

ポイントとしては、 **めんどくさいと"少しでも"思ったことをすぐに入力する** ことです。  
実際に自動化できるかとか、どうやって自動化していくかとかは、この時は一切考えません。

そのためにメモの **入力のハードルを可能な限り下げる** ことを意識します。

私はRaycastというアプリでコマンドを作って、1行入力するだけでGitHubのイシューとして追加されるようにしています。

```ruby
#!/bin/bash

# Required parameters:
# @raycast.schemaVersion 1
# @raycast.title fukai-memo
# @raycast.mode silent
# @raycast.icon 📝
# @raycast.argument1 {"type":"text","placeholder":"追加するメモ（1行）"}
# Optional parameters:
# @raycast.argument2 {"type":"text","placeholder":"補足（あれば）"}
# Documentation:
# @raycast.description 不快メモに1行追加

set -euo pipefail

REPO="kazuyaseki/nandemo-automation"
TITLE="${1:-}"
BODY="${2:-}"

[[ -z "$TITLE" ]] && { echo "タイトルが空です"; exit 1; }

gh issue create --repo "$REPO" --title "$TITLE" --body "${BODY:-(本文なし)}"
```

![画像](https://assets.st-note.com/img/1767424267-oyEgut9D3BKMW5648GehOUfq.png?width=1200)

すぐにメモしてGitHub issueにアップロード

また、ここではハードルを下げて **とにかく何でもいいから入力しておくこと** が大事なんですが、とはいえ、後ほどAIにリサーチしてもらったり解決策を実装してもらうときに、ある程度具体的な内容がないと、的外れな解決策を提案してきて無駄に時間がかかります。

AIに情報が足りな過ぎたら聞き返してもらうようにはできるのですが、最初の段階である程度入力してある方がスムーズです。

ここで推したいのは音声入力です。音声入力ではディテールをちょっと足すハードルが下がります。  
私は去年からAqua Voiceをずっと使っているのですが、うまい具合にAIが補完してくれてかなり精度が良いように感じているので、まだの方はぜひ試してみてください。

以下は私のAqua Voiceの招待リンクで1ヶ月無料でお試しできます。(私も1ヶ月分のクレジットがもらえてハッピーになります。)

上記のような手段たちを用意して、とにかくちょっとでも「面倒だなぁ…」と感じた自動化のタネを逃がさないようにすること、それがここでの目的です。

## 発展編: AIに解決策を提案、なんなら実装してもらう

こういったメモを取るだけでも十分に意味があると思いますが、更なる発展として **メモったらAIが勝手に解決策を提案** してくれる状態を作りましょう。

この「AIにとりあえず解決策を教えてもらう」というのは結構大事だと思っていまして、自分の知っている範囲だけで考えてしまうと、 **無意識のうちに諦めてしまっている** 状態が発生します。

これは長年ソフトエンジニアとして働いてきた私でもそうで、難しそうと思い込んでいたものがAIに相談してみたら「え、そんな簡単に実現できるの？」と気づくことは多々あります。

そこで、AIに考えてもらうためにどうしたらいいかという話ですが、一つはシンプルにChatGPT、Gemini、Claudeなどお好きなAIアプリケーションにコピペして調べてもらうということです。

これでも望む結果は得られるのですが、やはり人間、この **「コピペして結果を待つ」という少しのアクション** があるだけで、 **習慣化から遠ざかって** しまいます。

という訳で **メモったらAIが勝手に考えてくれる状態** を自動化して作っていきましょう。

ここにも段階がありまして、1ステップ目は「 **AIが解決策を自動で答えてくれる** 」で2ステップ目は「 **AIが実装可能な自動化は勝手に実装してくれる** 」を行います。

今回は難易度が上がりますが、2ステップ目まで行えるGitHub Actionsを使った方法をお伝えします。1ステップ目までだけだったら n8n などのワークフローツールでも「スプレッドシートで行が追加されたら、AIが考えて、Slackに通知」みたいな形の仕組みが作れると思います。

Git + GitHubを使ったことがない方はこちらの動画からぜひ始めてみてください。

![](https://www.youtube.com/watch?v=Uml3jEPWIKo)

GitHub Actionsとはなんぞやについて学びたい方はこちらの記事をご参照ください。

このラインより上のエリアが無料で表示されます。

まず前提として何でも自動化のスクリプトやプロンプトを保存するだけのレポジトリを用意します。

私は \`nandemo-automation\` というふざけた名前で作りました。

![画像](https://assets.st-note.com/img/1767432664-hiJ3lWAZ68ePwLfbNvEmSUIa.png?width=1200)

また、今回の仕組みではGitHubの"イシュー"という機能を活用します。  
この"イシュー" とはなんぞやという話ですが、GitHub内で課題を登録できる機能です。

![画像](https://assets.st-note.com/img/1767433830-65V7fQlHZLkpFOGMmCTIAKWd.png?width=1200)

イシュー一覧は左上のタブから開ける

課題のタイトルと説明が入力できるだけのものなので、そんなに難しく考えなくて大丈夫です。

![画像](https://assets.st-note.com/img/1767433876-pS9Yg4OvshH3ZVNumntW85yi.png?width=1200)

イシューの新規作成

![画像](https://assets.st-note.com/img/1767433960-Crtfqa6XJVQDGYZlxwOB2pER.png?width=1200)

実際のイシューの画面

次にClaude CodeをGitHub Actionsで動くようにします。Claude CodeはAnthropic社のコーディングツールで、動かすには月額$20のProプランが必要です。  
(記事の最後に無料で同じことができるJulesについても軽くご紹介します。)

使ったことがない方はターミナルから以下のコマンドでインストールできます。

```javascript
curl -fsSL https://claude.ai/install.sh | bash
```

ターミナル・CLIが初という方は以下の動画にて解説してあるので、よければご覧ください。

![](https://www.youtube.com/watch?v=_FU3Yfizfn0)

Claude Codeを、上記のレポジトリのフォルダーを開いた状態で立ち上げて \`install-github-app\` というコマンドを実行すると簡単にセットアップできます。

![画像](https://assets.st-note.com/img/1767425274-XmDbeIpUF3lziOWnH7CMLk0v.png?width=1200)

なお、Claude Codeの利用コストが気になる方もいるかもしれませんが、  
Claudeの定額プラン（Pro/Max）に加入していれば、追加料金なしで  
GitHub ActionsでのClaude Codeも利用できます。  
また記事の最後に触れるJulesでは無料で始めることができます。

そして、次にイシューが追加されたときに、Claude Codeが動いて、「こんな解決策がありますよ」とコメントしてくれたり、それがそのまま実装できそうだったら、コードを書いてもらって、実装できるような仕組みを作ります。

以下のようなワークフローファイルを作って、レポジトリの.github/workflows 配下に配置します。

やっていることはシンプルで、イシューが作られた時に

- そのタイトルと中身の文章を読む
- 解決策をコメントする or 足りないものがあったらより深掘って質問する or 実装できそうだったら、そのままプルリクエストを作ってもらう

をしています。

```cs
name: Issue Solver

on:
  issues:
    types: [opened]
  workflow_dispatch:
    inputs:
      issue_number:
        description: '対象のIssue番号'
        required: true
        type: string

jobs:
  solve:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
      id-token: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get Issue Info (for workflow_dispatch)
        if: github.event_name == 'workflow_dispatch'
        id: issue_info
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          ISSUE_JSON=$(gh issue view ${{ inputs.issue_number }} --json title,body,number)
          echo "title=$(echo $ISSUE_JSON | jq -r '.title')" >> $GITHUB_OUTPUT
          echo "number=$(echo $ISSUE_JSON | jq -r '.number')" >> $GITHUB_OUTPUT
          # bodyは改行を含むのでワークスペース内にファイルとして保存
          echo $ISSUE_JSON | jq -r '.body' > .github/issue_body.txt

      - name: Run Claude Code for Issue Solving
        id: claude
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          
          prompt: |
            あなたは優秀なソフトウェアエンジニアです。以下のIssueを読み、解決策を提案・実装してください。

            ## Issueタイトル
            ${{ github.event_name == 'workflow_dispatch' && steps.issue_info.outputs.title || github.event.issue.title }}

            ## Issue本文
            ${{ github.event_name == 'workflow_dispatch' && 'Issue本文は .github/issue_body.txt を読んでください' || github.event.issue.body }}
            
            ## Issue番号
            ${{ github.event_name == 'workflow_dispatch' && inputs.issue_number || github.event.issue.number }}

            ## 🚨 最重要ルール：実装タイプの分類（最初に必ず実行）

            Issueを読んだ後、**最初に**以下の分類を行い、どのカテゴリに該当するか判断してください：

            ### カテゴリA: AIのAPI呼び出しが必要なタスク（実装禁止）
            以下に該当する場合は **絶対にコードを実装してはいけません**：
            - テキスト生成、要約、翻訳、校正などの自然言語処理タスク
            - 画像生成、画像認識、音声処理
            - チャットボット、対話システム
            - 感情分析、分類、推論が必要なタスク
            - OpenAI API、Anthropic API、Google AI APIなどの呼び出しが必要なもの
            
            → **このカテゴリの場合**: \`.claude/skills/\` ディレクトリにプロンプトファイル（.md）のみを作成してPRを出す。スクリプトやコードは一切書かない。

            ### カテゴリB: 自己完結型のタスク（実装OK）
            以下に該当する場合は実装してPRを作成してOK：
            - ファイル操作、データ変換、フォーマット変換
            - 静的なWebサイト（HTML/CSS/JS）
            - シェルスクリプト、自動化スクリプト
            - GitHub Actions ワークフロー
            - 計算、集計、データ処理（AIなしで可能なもの）

            ### カテゴリC: 情報不足（質問が必要）
            → Issueにコメントで質問を投稿し、処理を終了

            ---

            ## 実行手順

            1. **Issueの内容を分析**
               - Issueのタイトルと本文を注意深く読む
               - 何を解決したいのか、何を実現したいのかを理解する
               - **上記の分類ルールに従ってカテゴリA/B/Cを判定する**

            2. **カテゴリAの場合（AIのAPI必須）**
               ⚠️ **以下の手順のみを実行。それ以外は禁止。**
               - \`.claude/skills/\` ディレクトリを作成（なければ）
               - \`.claude/skills/{適切な名前}.md\` ファイルを作成
               - ファイル内容はプロンプトのみ（使い方の説明とプロンプト本文）
               - PRを作成してIssueにコメント
               - **スクリプトやコードは絶対に書かないこと**

            3. **カテゴリBの場合（自己完結型）**
               - 適切なディレクトリ構成でファイルを作成
               - 必要に応じてREADMEや使い方の説明を追加
               - コードはシンプルで読みやすく保つ
               - PRを作成してIssueにコメント

            4. **カテゴリCの場合（情報不足）**
               - Issueにコメントで追加の質問を投稿する
               - **質問には必ず選択肢をチェックボックス形式で提示する**
               - 以下のフォーマットで質問を作成：
                 \`\`\`
                 ご要望をより詳しく理解するために、いくつか質問させてください。
                 該当する項目にチェックを入れてください：

                 **[質問1の内容]**
                 - [ ] 選択肢A
                 - [ ] 選択肢B
                 - [ ] 選択肢C

                 **[質問2の内容]**
                 - [ ] 選択肢A
                 - [ ] 選択肢B

                 その他ご要望があれば、コメントで追記してください。
                 \`\`\`
               - ユーザーがチェックボックスをクリックするだけで回答できるようにする
               - **ここで処理を終了（PRは作成しない）**

            5. **PRの作成**（カテゴリA・Bの場合）
               - ブランチ名: \`solve/issue-{Issue番号}\`
               - PRタイトル: 「[Solve #{Issue番号}] {解決内容の要約}」
               - PR本文に以下を含める:
                 - **先頭に \`Closes #{Issue番号}\` を必ず記載**
                 - カテゴリAの場合: 「プロンプトのみを提供」と明記
                 - 解決策の説明
                 - 使い方・セットアップ方法（必要な場合）

            6. **Issueへのコメント**
               - PRを作成した場合：PRへのリンクを含むコメントを投稿
               - Issue番号は上記「Issue番号」セクションを参照

            ## 注意事項
            - **カテゴリAに該当する場合、Node.js/Python等のスクリプトを書くことは厳禁**
            - シンプルな解決策を優先する
            - セキュリティに配慮したコードを書く
            - 日本語でコメント・説明を書く

          # PRを作成するために必要な権限
          additional_permissions: |
            contents: write
            pull-requests: write
            issues: write

          # ファイル編集とgitコマンドを許可
          claude_args: |
            --allowedTools "Read,Write,Edit,Bash(git:*),Bash(gh:*),WebSearch,WebFetch"
```

また、先ほどの \`install-github-app\` を実行したときに作られる claude.yaml も、イシューへの返信時のプロンプトをこちらのものと合わせます。  
こんな感じで分岐を作ります。

```cs
# Issueコメント用のジョブ（解決策検討プロンプトを使用）
  claude-issue:
    if: |
      github.event_name == 'issue_comment' && 
      contains(github.event.comment.body, '@claude') &&
      !github.event.issue.pull_request
```

こうやって準備すると、あとはイシューを追加したら、Claude Codeが勝手に動いてくれてコメントしてくれます。  
下記が実際のイシューのやりとりのスクショです。

![画像](https://assets.st-note.com/img/1767425382-9iRJeDg2pUY6GB17tqQ8dAIy.png?width=1200)

Skillの作成をしてくれてます。

![画像](https://assets.st-note.com/img/1767425417-1U3xJdiLe7sEjRTgknDbIFyl.png?width=1200)

正直、このSkillを作るくらいだったらそこまで大したことない感はありますが、

> 自分の過去の動画のURLを、その動画を紹介した動画の概要欄にコピペするのめんどい  
> 今は逐一YouTube Studioを開く -> 該当の動画を探してURLとタイトルをコピペしてる

とメモをするだけで簡単なWebアプリを作ってくれたりもしました。

![画像](https://assets.st-note.com/img/1767425490-uwAITFD4iNrpXEjzyt235SOl.png?width=1200)

また、要件が曖昧過ぎる場合には質問をしてくれます。  
そこでチェックボックスでポチポチしたり、自由入力で回答しつつ、

![画像](https://assets.st-note.com/img/1767514672-37zUFm1XDnoYkRNcw4HTW58q.png?width=1200)

@claude とメンションをつけて回答すれば、より詳細な意図を伝えた上でリサーチや実装を始めてくれます。

![画像](https://assets.st-note.com/img/1767515228-mr9wBu4AhC2c7ZfYdpGaEvTF.png?width=1200)

もちろん、簡単に実現できるものばかりではないのですが、こういった仕組みを作っておくと、 **自動化を始めるときのハードルがかなり下がる** のでおすすめです。

仮にAIがドンピシャな解決策を提示できなくとも、とりあえず課題の管理はできている状態になるので、定期的に見返して、インパクトがありそうだったら「 **腰を据えて解決策を自分で作る** 」という扱い方をすると良いでしょう。

最後に、GitHub Actionsはややハードル高めに感じるかもしれないので、同様のことが実現できる方法を二つほどご紹介します。

まず一つ目はClaude CodeのSlackアプリです。こっちの方がインターフェースとしては楽かもしれません。

私は不快なことリストをGitHubのイシューとして管理したかったのと、また、このSlackアプリ方式ではCLAUDE.md以外の方法で具体的なプロンプトを書く手段がなさそうに見えたので、GitHub Actionsを採用しました。

また、JulesというGoogle製のクラウドで動くコーディングアプリがあるのですが、こちらはイシューを元に勝手に動き始めるという機能を持っています。

この機能を使うのは簡単でGitHubのイシューにjulesというラベルを付けるだけです。Raycastで追加する例を冒頭にお見せしましたが、その時にこういったラベルを付与すれば勝手に動いてくれます。

![画像](https://assets.st-note.com/img/1767426630-UVQrwBuEk7XTOmjpAgbq58IS.png?width=1200)

つまり、上に貼ったワークフローファイルを作る必要がなく、セットアップの手間がかなり軽いのでありがたい機能です。

私がClaude Codeを使うのはClaude Codeが現在最強のコーディングエージェントだからなのですが、この不快メモで解決したいことは簡単な課題が多くなるため、最強の性能は必ずしも必要ではないかもしれません。

なので Jules で簡単に始めてみるのも良い選択肢でしょう。

以上、不快メモのご紹介でした。  
ぜひ自動化のハードルを下げる仕組み、試してみてください！