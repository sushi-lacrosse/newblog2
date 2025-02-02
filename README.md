# ブログ運用ルールメモ


## 目的
エラーによるブログにアクセスできない状態を無くす(ユーザーアクセスの機会損失)  
そのためにブランチを作ってプッシュしてエラーがないか事前に確認してから、masterブランチにマージする  

後から振り返ってわかりやすいようにこまめにpushする  
途中で機能づくり諦めるときにブランチごと削除できるように  

エラーに対してなにをやったか記録する 
## Gitフロー  
```mermaid
flowchart TB
  subgraph 事前準備
  A["新しく作業を始める"] --> B["g cob これからしたい作業内容がわかる短めブランチ名"]
  end
  B --> D["hugo new posts/記事内容がわかるタイトル/記事内容がわかるタイトル.md"]
  subgraph 新しく記事を作る
  D --> D3["記事を書く"]
  D3 --> D1[".mdファイルのFrontMatterの記事タイトルタグとカテゴリを'日本語'で入力する"]
  subgraph OGP画像生成
  D1 --> D2["ホームディレクトリで zsh ogp.sh blog/content/posts/記事内容がわかるタイトル/記事内容がわかるタイトル.md"]
  end
  end
  subgraph 新しい機能を作るor編集する
  B --> C1["blogディレクトリでファイル作るorファイルを編集する"]
  end
  C1 --> E
  D2 --> E["g st #変更したいファイルだけ変更されているか確認"]
  E --> F["g add . #カレントディレクトリ以下の変更があったファイルをステージング"]
  F --> G["g cm -m 'なにを、どうしたか、何のために'"]
  G --> H["g ps"]
  H --> I{"完成した？"}
  I --> |Yes| J["GitHubブラウザでプルリクとマージを行う"]
  I --> |"No"| K["さらに記事内容を書く"]
  K --> E
  J --> L["g co master"]
  L --> M["g pl"]
  M --> N["g brd"]
```
### 新しく記事を作るとき
postsの中に記事タイトルがわかるディレクトリ名(例:HowToWatchPLL)をつけてその中にindex.mdをつくる  
`hugo new posts/HowToWatchPLL/HowToWatchPLL.md`  
記事で使う画像ファイルもすべて今作ったディレクトリの中にいれていく

### 新しく作業を開始する時（リモートリポジトリからローカルリポジトリにダウンロード）
`g pl origin master`
### ブログに新しく機能を追加する時
git ブランチを新しく作る  
`g add ファイル名`  
新しく作ったブランチにプッシュしていく  
pull request送る（netlify上で見れるようになる）  
netlifyで問題ないか確認する  
マージする  

### ブログの記事更新時・新機能作成途中の
## zshエイリアス
### vimで内容確認変更  
ホームディレクトリでターミナルに`vim ~/.zshrc`と入力する  

### エイリアス単語リスト
```
g = 'git'
```

## git エイリアス
### vimでgitエイリアス内容確認・変更
ホームディレクトリでターミナルに  
`vim ~/.gitconfig ##vimというエディタでホームディレクトリにある.gitconfigファイルを開く`  
と入力する

### 現状のエイリアスリスト  
```
add = a
co = checkout                 ## `g co ブランチ名` 指定したブランチに移動   
cob = checkout -b             ## `g cob ブランチ名` 新しいブランチを作成してそのブランチに移動
b = branch                   ## `g br` 現在あるブランチ一覧を表示
brd = branch -d               ## `g brd ブランチ名` 指定したブランチがマージ済であれば削除
brdd = branch -D              ## `g brdd ブランチ名` 指定したブランチを強制的に削除
d = diff                      ## `g d` 現在のワークツリーとインデックスの差分を表示
st = status                   ## `g st` 
cm = commit  
pl = !git pull origin `git rev-parse --abbrev-ref HEAD`  
ps = !git push origin `git rev-parse --abbrev-ref HEAD`  
```

g st の項目  
| ログ名                           | 意味                                    |
| ----------------------------- | ------------------------------------- |
| Changes to be committed       | コミットする予定の変更したファイル（インデックスに反映されているファイル） |
| Changes not staged for commit | まだインデックスに追加されていない、変更したファイル            |
| Untracked files               | Git でまだ管理されていないファイル（新しく追加したファイル）      |  


## 作業したコミットを元に戻したいとき
`git log --oneline`  コミットログを一行ごとに表示
`git reset --hard ハッシュ値` もどしたいハッシュ値を最後に追加して実行

## tcardgen（サムネ画像自動作成ツール）使い方
(https://michimani.net/post/development-generate-ogp-image-by-tcardgen-in-hugo/)  

サムネ画像を作りたい記事のカテゴリとタグだけ入力してから
以下ホームディレクトリでターミナルへ  
```
zsh ogp.sh blog/content/posts/004HowToWatchPLL.md
# ogp.sh というシェルスクリプトの中に、記事ファイルのパスを第一引数として渡して実行している。
```
以下ogp.shの中身
```
if [ $# != 1 ] || [ $1 = "" ]; then
  echo "One parameters are required"
  echo ""
  echo "string: path to markdown file of target post"
  echo ""
  echo "example command"
  echo "\t$ sh ./scripts/gen_ogp.sh ./content/post/test/test.md"
  exit
fi

TARGET_POST_PATH=$1

tcardgen \
--fontDir blog/static/fonts/Kinto_Sans \
--output blog/static/ogp \
--template blog/static/img/tcardgen_template.png \
$TARGET_POST_PATH
```
```
tcardgen 
--fontDir=自動生成する画像に使うフォントのディレクトリのパス 
--output=画像を自動生成した時の出力先ディレクトリ 
--template=自動生成する前の背景テンプレート画像のファイルパス 
タイトルやタグの情報を取得する記事のファイルパス
```

## 便利なショートカット
選択範囲のブロックコメント化:(MacOS) Shift + option + a  

## Pocket (ネット記事ページ保存サービス) 使い方
エラーに対してやった施作の記事はタグ[blog],[did]をつける  
