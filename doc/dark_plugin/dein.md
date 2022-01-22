# 始めに

この記事は、Shougo氏が作製するプラグインを解説する、闇のプラグインシリーズ第1弾になります。
第1弾では、Vim/NeoVimのプラグインマネージャーである、dein.vimを紹介していこうと思います。

https://github.com/Shougo/dein.vim

ただ、紹介するだけなら、ググればいろいろ出てきます。
ですので、dein.vimがプラグインをどのように管理しているのか、読み解いてみようと思います。


<!-- textlint-disable -->

> _自らがVim、もしくはテキストエディタと一体化することを望まないなら、気を付けた方がいい。<br>
深淵(Vim)をのぞく時、深淵(Vim)もまたこちらをのぞいているのだ。_<br>
> <div style="text-align: right">フリードリヒ・ニーチェの名言よりオマージュ</div>

<!-- textlint-enable -->


## `runtimepath`から見るプラグインマネージャー

Vim/NeoVimのプラグインと呼ばれる物は、基本的には`runtimepath`に指定したパスを起点に、特定のディレクトリ内のスクリプトをプラグインと呼んでいます。
これは、GitHubで配布されているプラグインにかかわらず、最初からいくつかの基本プラグインが読み込まれています。

ただ`runtimepath`は、指定したパス内の特定のディレクトリを読み込みに行くという単純な動作のため、
プラグインマネージャーとは、スクリプトを使って`runtimepath`管理をしやすくしている物と思ってください。

また、プラグインマネージャーの機能として、自動でプラグインのインストールと更新をしたり、特定のファイルタイプでプラグインが起動するようなオプションを提供しています。


### では、dein.vimはどうなっているのか。

例外なくdein.vimも`runtimepath`を利用していますが、既存の`runtimepath`を上書きをして高速化を実現しています。
つまり、dein.vimを導入するということは、全体の`runtimepath`の管理をdein.vimで行うことになるのです。
dein.vimはVim/NeoVimが起動するまでのプロセスの妨げにならず、ユーザーがすべてを操作できるように機能が提供されています。

しかし、それらの機能を理解しないで使うということは、宝の持ち腐れになります。
自分がヘルプやソースコードを読んで、こう思いました。
「プラグインをインストールして遅延起動させたいだけなら、ほかのプラグインマネージャーで良い。
Vim/NeoVimを極限まで調整したい人なら、導入していじり倒すべきプラグインマネージャーなのだ。」


## 他のプラグインマネージャーとの比較

それでは、dein.vimがVim/NeoVimを極限まで調整する人向けのプラグインマネージャーということが分ってもらえたと思います。
ですので、少々脱線してほかのプラグインマネージャーも見てみようと思います。


### 組込みのプラグインマネージャー

Vim/NeoVimには、`packages`という組込みのプラグインマネージャーが存在します。
詳しいことは`:h packages`をご覧いただければ分ると思いますが、結果的に`runtimepath`を使用してプラグインを読み込むことに変わりありません。
プラグインマネージャーを使わず、組込みの機能だけを使うという方は、こちらをお使いください。


### NeoVim限定のプラグインマネージャー

最近では、`packages`を使うプラグインマネージャーが出てきています。
その中でも[packer.nvim][2]は有名で、`packages`を使ってプラグイン管理をしています。

NeoVimでは、Luaで設定ができるようになっていて、NeoVimユーザーならLuaで設定を書くと速くなると言われています。
packer.nvimのコーディングにはLuaが使われているので、NeoVim永住宣言をしている方にピッタリだと思います。


### 大手のプラグインマネージャー

Vim/NeoVimのプラグインマネージャー界隈でも、大手の[vim-plug][3]があります。
こちらは、ワンスクリプトで完結していて、`runtimepath`をセットしたパス内の`autoload/`に配置して、プラグインがダウンロードされる場所を指定するだけで使えるミニマルなプラグインマネージャーです。
このプラグインマネージャーは機能がシンプルで、Vimを初めて触るという方が、プラグインを入れたいと言ったら、お勧めするプラグインマネージャーです。

「えっ？  dein.vimの紹介なのに、ほかのプラグインマネージャーをお勧めしちゃうの？」と、思われるかもしれません。
先にも説明した通り、dein.vimはVim/NeoVimを極限まで調整したいユーザーが導入するべきプラグインマネージャーであり、初心者にお勧めするプラグインマネージャーではないのです。
しかし、このvim-plugは設定もシンプルで、難しいところが少ないので、Vimでプラグインを入れてみたいだけならこれで十分なのです。

つまり、対象としているユーザー層が違うのです。


## dein.vim\~高速化の鍵\~

ここからは、dein.vimがどのようにして高速化を実現しているか、紐解いていきたいと思います。

今回は、以下の環境で見ていきます。

<dl>
    <dt>OS</dt>
    <dd>Linux</dd>
    <dt>Vim or NeoVim</dt>
    <dd>NeoVim</dd>
    <dt>インストール先</dt>
    <dd><code>`~/.cache/dein`</code></dd>
    <dt>プラグイン管理方式</dt>
    <dd>tomlファイル</dd>
</dl>

また、dein.vimのインストールで使用するスクリプトは以下のようにしています。

<!-- dein.vim install script e.g {{{ -->
<div><details><summary>dein.vimインストールスクリプト</summary>

```vim:~/.config/nvim/init.vim
source ~/.vim/init.vim
```

```vim:~/.vim/init.vim
if &compatible
  set nocompatible
endif

" dein base directory.
let s:dein_dir = expand('~/.cache/dein')

" dein repository directory.
let s:dein_repo = s:dein_dir . '/repos/github.com/Shougo/dein.vim'

" vim setting directory.
let g:base_dir = fnamemodify(expand('<sfile>'), ':h') .. '/'

" plugins toml-file directory.
let s:toml_dir = g:base_dir ..'toml/'


if &runtimepath !~# '/dein.vim'
    if !isdirectory(s:dein_repo)
        execute '!git clone https://github.com/Shougo/dein.vim' s:dein_repo
    endif
    execute 'set runtimepath^=' . s:dein_repo
endif

if dein#min#load_state(s:dein_dir)
    call dein#begin(s:dein_dir)

    let s:dein_toml = s:toml_dir .. 'dein.toml'
    let s:dein_lazy_toml = s:toml_dir .. 'lazy.toml'

    " read toml and cache
    call dein#load_toml(s:dein_toml, {'lazy': 0})
    call dein#load_toml(s:dein_lazy_toml, {'lazy': 1})

    " end settings
    call dein#end()
    call dein#save_state()
endif


if dein#check_install()
    call dein#install()
endif

filetype plugin indent on
syntax enable
```
</details></div>

<!-- }}} -->

### no_lazyのプラグインも高速化するマージ機能

tomlファイルでプラグインを管理する場合`dein#load_toml()`を使い、読み込むtomlファイルとオプションを指定します。
オプションでは主に遅延の有無で、2つのtomlファイルを用意ことになると思います。
ここからすでに高速化は始まっていて、最初の鍵になるのは、遅延設定をしていないtomlファイルなのです。

どちらかというと`VimEnter`の後に、プラグインを起動させることが高速化につながるのですが、dein.vimは違います。
遅延設定していないプラグインにこそ、高速化のひと手間を施しているのです。
先に解説した通り、プラグインは`runtimepath`に指定されたパス内のディレクトリにあるスクリプトを読み込むことで、プラグインとして成り立ちます。
この`runtimepath`というのは指定された分だけちゃんと順番に読み込むので、パスの数が多くなればなるほど、読み込みに時間がかかるようになるのです。
遅延設定をしていないプラグインがあった場合、通常ですとプラグインマネージャーがダウンロードしてきたプラグインのディレクトリのパスを指定して、Vimが起動する際に`runtimepath`が順次読み込みにいきます。

しかし、dein.vimでは違います。
遅延設定していないプラグインは`~/.cache/dein/.cache/init.vim/.dein/`[^1]というディレクトリにマージされて、読み込む`runtimepath`の数を少なくしてくれます。
`runtimepath`の読み込みはパス内の読み込むディレクトリをすべて読み込んだあと、次のパスの読み込みに移ります。
この数をdein.vimでは、ディレクトリをマージしてしまうことで少なくしているのですね。
もちろんプラグインによってはマージされたら困るものや、スクリプト名が被ってしまう可能性もありますので、ユーザーが明示的にマージを無効化できます。

toml内でマージしてほしくない場合は、以下のように記述します。

```toml:dein_no_lazy.toml
[[plugins]]
repo = 'user_name/repository_name'
merged = 0
```

マージして読み込むディレクトリを少くしても、読み込むスクリプトが多ければ時間がかかるので、マージ機能を過信しないでください。
あくまで、読み込みを高速化するために、ひと手間処理をしてくれているものと思ってください。


### 設定は`state_nvim.vim`[^2]に集結する。

プラグイン読み込みのタイミングを指定する方法はいくつかありますが、`on_*`系が一番ポピュラーですね。

dein.vimでは、プラグインの読み込みや更新のタイミングでフックして、ユーザーが書いた設定処理を実行できます。
フックについて分りやすいのは以下の記事になります。

https://qiita.com/delphinus/items/cd221a450fd23506e81a

それでは、プラグインの読み込みタイミングや設定内容が、どのように処理されているのかというと、`~/.cache/dein/state_nvim.vim`にまとめられています。
実際にプラグインをインストールして、いくつか設定をしたうえで`state_nvim.vim`を見てみると、tomlファイルに記述した設定内容がVim Scriptになって反映されています。

ここでも読み込むファイル数を少なくして、高速化を図るdein.vimの設計思想が伺えますね。
そして、`state_nvim.vim`を読んでもらえると分るかと思いますが、ここではVim/NeoVimの起動時の`runtimepath`を定義しています。

これらの処理は、`dein#save_state()`によって実行され、`state_nvim.vim`を生成してくれます。


### プラグインの更新について

dein.vimでは、プラグインの更新に使う関数として、`dein#update()`があります。
通常プラグインの更新というと、各リポジトリの`master/main`ブランチのバージョンを確認してプルなどで更新します。
しかし、dein.vimは更新確認に`git`コマンドを使用しないで、`.git/`内のファイルを参照してバージョンを確認しているようです。
Windowsでは外部プロセスの起動に時間がかかるため、高速化になるようです。

[参考記事:永遠に未完成より][4]


## お勧めのオプション

dein.vimで使わなくても大丈夫だけど、使うともっと便利に高速化できるオプションを紹介したいと思います。


### `g:dein#inline_vimrcs`で分割している設定を、`state_nvim.vim`にまとめる。

ファイルの見通しを良くするために設定ファイルを分割するユーザーもいます。
<!-- textlint-disable -->
たとえば、`:set`などの基本オプションだけまとめたスクリプトや、オレオレキーバインド、GUI専用のオプション……etc.
<!-- textlint-enable -->
そういった人のために、それらの設定を`state_nvim.vim`にまとめてくれる機能がdein.vimにはあります。

設定方法は簡単で、読み込んでほしい設定ファイルのパスを`g:dein#inline_vimrcs`に追加することです。
`g:dein#inline_vimrcs`はリスト型になっているので、`if has() … end`と`add()`を使えば使用条件によって、読み込む設定ファイルの変更が可能です。

<!-- g:dein#inline_vimrcs code e.g {{{ -->
<div><details><summary><code>g:dein#inline_vimrcs = []</code>の記述例</summary>

<!-- ここにg:dein#inline_vimrcs = []の記述例を入れる。 -->

</details></div>
<!-- }}} -->

注意点は、`dein#begin()`よりも前に定義する必要があることですね。

最終的に設定ファイルが読み込まれると、コメントや空行などの無駄なものを省いて、`state_nvim.vim`に含めてくれます。


### `g:dein#lazy_rplugins`でリモートプラグインの読み込みも遅延する。

NeoVimには、リモートプラグインという別の言語で書かれたプラグインを読み込むためのスクリプトがあります。
これも基本プラグインに含まれる訳ですが、依存するプラグインが起動するまで、読み込まないようにできます。
使用するプラグインの依存関係によりますが、そういったプラグインを使用しているなら、設定してみるのはどうでしょうか。


### GitHub GraphQL APIを使用した、高速アップデート


## 注釈

[^1]: .vimrcを読み込んでいる場合、`~/.cache/dein/.cache/.vimrc/.dein/`になります。
[^2]: ファイル名はNeoVim使用時です。Vim使用時は、`state_vim.vim`になります。


<!-- Links -->
[1]: https://github.com/Shougo/dein.vim
[2]: https://github.com/wbthomason/packer.nvim
[3]: https://github.com/junegunn/vim-plug
[4]: https://thinca.hatenablog.com/entry/dein-vim-with-graphql-api