# Server用に使用する最小設定の.vimrcを作った

前回の投稿から、大分期間が空いてしまいました。
実は、転職などの身辺の変化があり記事を書いていませんでした。

今回は転職後に会社のサーバ研修でViを触ることがあったのですが、
普段のVim/Neovimとは違う操作感で思ったような操作ができず、途中からVimをインストール[^1]して最小設定の`.vimrc`を使用しました。

新規のVimmerが増えたら良いなという思いも込めて、必要と思う設定を書き出してみました。
それでは、私が感じたViとVimの違いとともに最小設定の`.vimrc`を構築していきましょう。


## ViとVimの違い

* 入力モード中、Backspaceで文字を消せない。
* Backspaceでは前方向にカーソル移動して、そこから入力を続けると、文字列の挿入ではなく置き換えになってしまう。
* uで戻せるのは1つ前の操作のみ。
* テキストオブジェクトを使用して、Vimでは可能な`ciw`や`ci'`などの操作ができない。

上記の操作が、私が実際にサーバ研修で遭遇したVimとは違う部分になります。
これら以外にも違いはありますが、それらに関してVimの中で`:h compatible`を実行すればヘルプを確認できます。

もともとVimとはViの拡張版であり、基本的な操作は変わりありません。
しかし、30年のバージョンアップを重ねてViだけでは実現できない機能もあります。
また、初期状態のVimでもViより高機能な設定が施されているので、`.vimrc`を作成しなくてもその恩恵を受けること "は" 可能です。

せっかくVimを使うんですから、`.vimrc`を作成してちょっとした操作を快適にしていきましょう！


## Vimのインストール

もしVimがインストールされていなければ、各環境の方法でVimをインストールしてください。

Linuxディストリビューションでは、まれに`vi`でVimが起動するようにエイリアスが設定されている場合があります。
`vi`か`vim`の両方を試してみてVimが起動しなかったらインストールしたら良いと思います。

Macでは初期にMacVimと呼ばれる物がインストールされているようですが、通常のVimとは違いmacOSに合わせた設定がされているらしいので、
通常のVimをインストールすることをお勧めします。


## vimrcを作成して、どこでも同じ操作感でvimする

まず始めに、`.vimrc`はホームディレクトリに置く必要があります。
ホームディレクトリに置くことで自動的にVimが読み込んでくれます。
「我こそは」、という猛者なら素のVimで`.vimrc`を作成すれば良いでしょうが、そういう人はすでに自分の`.vimrc`を持っているものです。

ここでは`.vimrc`を普段使用しているエディタで作成し、GitHubなどにプッシュしてdockerなどのサーバ環境にクローンして使うことを想定します。
一般的に`.vimrc`などのドットで始まる設定ファイルは、dotfilesというリポジトリで管理されることが多いです。


### 必要な設定項目と書き方

`.vimrc`の中ではvimscriptと呼ばる言語で設定を書きますが、基本的な設定ならそこまで難しくはありません。

```vimscript:.vimrc
" ダブルクォーテーションでコメントアウト
set {option_name}            " 設定のオン
set no{option_name}          " 設定のオフ

" ノーマルモードのキーリマップ
nnoremap {実際に実行するキーバインド} {もともとvimで使用されているキーバインドやコマンド}

" インサートモードのキーリマップ
inoremap {実際に実行するキーバインド} {もともとvimで使用されているキーバインドやコマンド}

" ビジュアルモードのキーリマップ
xnoremap {実際に実行するキーバインド} {もともとvimで使用されているキーバインドやコマンド}
```

キーマッピングの変更はこれら以外にもいろいろありますが、今回使用するものだけにします。

---

それでは以下に`set`を使用する設定項目をリストアップしました。

<!-- textlint-disable -->
|# |オプション名                                                      |説明                                                                        |
|--|------------------------------------------------------------------|----------------------------------------------------------------------------|
|1 |encoding=utf-8                                                    |Vim内で使用される文字コード。                                               |
|2 |fileencoding=utf-8                                                |ファイル作成時に使用される文字コード。                                      |
|3 |fileencodings=utf-8,sjis,iso-2022-jp,euc-jp                       |ファイル読み込み時に使用する文字コード。                                    |
|4 |fileformats=unix,dos                                              |改行方式                                                                    |
|5 |autoread                                                          |Vimの外部で変更があった場合に自動で読み直す。                               |
|6 |number                                                            |行番号の表示。                                                              |
|7 |relativenumber                                                    |カーソル行から相対行番号を表示する。                                        |
|8 |cursorline                                                        |現在のカーソル行をハイライトする。                                          |
|9 |hlsearch                                                          |検索文字列をハイライトする。                                                |
|10|incsearch                                                         |検索文字列の入力中にマッチする文字列へジャンプする。                        |
|11|ignorecase                                                        |検索文字列の大小文字を区別しない。                                          |
|12|smartcase                                                         |検索文字列に大文字が含まれたらignorecaseを上書きして、大小文字を区別します。|
|13|wrapscan                                                          |検索が末尾まで進んだらファイル先頭から検索を再開する。                      |
|14|smartindent                                                       |改行時に自動でインデントする。                                              |
|15|expandtab                                                         |インデントにスペースを使用する(お好みで)。                                  |
|16|shiftwidth=2                                                      |改行時やキーバインドで2個分のスペースをインデントとして挿入する。           |
|17|tabstop=2                                                         |スペース2個分をインデントとして扱う。                                       |
|18|softtabstop=2                                                     |<Tab>を押した時にスペース2個分を挿入する。                                  |
|19|laststatus=2                                                      |ウィンドウの一番下にステータス行を表示する。2は常時表示。                   |
|20|wildmenu                                                          |コマンド入力時に<Tab>でコマンドの候補が表示されるようになる。               |
|21|list                                                              |不可視文字を表示する。                                                      |
|22|listchars=tab:»-,space:･,trail:･,nbsp:%,eol:↲,extends:»,precedes:«|各不可視文字の記号を設定する。                                              |
<!-- textlint-enable -->

---

また`set`を使用しない設定項目として以下の設定があります。

```vimscript:.vimrc
" ファイル形式別プラグインとインデントの有効
filetype plugin indent on
" シンタックスハイライトの有効
syntax enable
```

<!-- textlint-disable -->
内容としては…
* ファイルの形式を自動認識
* Vimに付属するftplugin.vimというプラグインが読み込まれる。
    * 各ファイル形式毎にVim側で用意している設定が使用される。
* ファイル形式毎のインデント設定が読み込まれる。
* シンタックスハイライトが有効になりコードに色が付く。
<!-- textlint-enable -->
上記の設定があるだけで最小設定とは思えないほど、見た目も良くなります。

---

設定項目としては以上です。
それでは、あると便利なキーバインドをいくつか紹介します。

```vimscript:.vimrc
" カーソルにファイル名とパスがあればgfで開く。
" path/to/target_file:10 だったら、target_fileの10行目を開く。
nnoremap gf gF

" 検索後のハイライトを<Space>nで消す。
nnoremap <Space>n :<C-u>nohlsearch<CR>

" xで文字を消したときに、レジスターに残さない。
nnoremap x "_x
xnoremap x "_x

" インサートモードでjを２回素早く入力するとノーマルモードに戻る。
inoremap jj <Esc>
" インサートモードでCtrl+lを使用するとDeleteキーと同じになる。
inoremap <C-l> <Del>

" US配列のキーボードを使用している場合のキーバインド
" JIS配列なら不要です。
nnoremap ; :
nnoremap : ;
nnoremap q; q:
xnoremap ; :
xnoremap : ;
```


## 最終的な最低限のvimrc

すべてを設定した場合、最終的に以下のような`.vimrc`になったかと思います。

```vimscript:.vimrc
set encoding=utf-8
set fileencoding=utf-8
set fileencodings=utf-8,sjis,iso-2022-jp,euc-jp
set fileformats=unix,dos
set autoread
set number
set relativenumber
set cursorline
set hlsearch
set incsearch
set ignorecase
set smartcase
set wrapscan
set smartindent
set expandtab
set shiftwidth=2
set tabstop=2
set softtabstop=2
set laststatus=2
set wildmenu
set list
set listchars=tab:»-,space:･,trail:･,nbsp:%,eol:↲,extends:»,precedes:«

nnoremap gf gF

nnoremap <Space>n :<C-u>nohlsearch<CR>

nnoremap x "_x
xnoremap x "_x

inoremap jj <Esc>
inoremap <C-l> <Del>

nnoremap ; :
nnoremap : ;
nnoremap q; q:
xnoremap ; :
xnoremap : ;

filetype plugin indent on
syntax enable
```

あとは、作成したリポジトリを使用したい環境にクローンして、ホームディレクトリへ張り付けるか、シンボリックリンクを設定すれば使用可能です。


## まとめ

これで最小設定の`.vimrc`は完成です。お疲れさまでした。
あとはVimを使用しながら、改善していきたい所を見つけて、`.vimrc`を拡張していきましょう！

さらなる設定やVimについて信頼できる情報は、vim-jpというコミュニティで[日本語翻訳しているヘルプ](https://vim-jp.org/vimdoc-ja/)がありますので、そちらをご覧ください。
日本語翻訳されたヘルプは[プラグイン](https://github.com/vim-jp/vimdoc-ja)として導入できますので、Vimを触るなら導入をお勧めします。

また、手前味噌ですが、[私のdotfiles](https://github.com/yasunori-kirin0418/dotfiles)を公開していますので、そちらもよろしければご覧ください。

私はVimが大好きですので、この記事を切っ掛けにVimmerが増えてくれたらうれしいです。

ここまでの閲覧、ありがとうございました。


### 参考リンク集

* vim-jp：https://vim-jp.org/
* Vim日本語ヘルプ：https://vim-jp.org/vimdoc-ja/
* Vim日本語ヘルプのプラグイン：https://github.com/vim-jp/vimdoc-ja
* 筆者のdotfiles：https://github.com/yasunori-kirin0418/dotfiles

### 注釈
[^1]: サーバ研修と言ってもdockerを使用して、手元で仮想Linux環境を構築しての研修ですので、実機ではありません。
