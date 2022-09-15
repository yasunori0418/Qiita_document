# Neovimの起動時間を早くする沼に沈みました

さて突然ですが、皆さんのVim/Neovimの起動時間はどのくらいでしょうか？

*~~えっ？　VSCode…。　知らない子ですね…。~~*

Vim/Neovimには起動時のデフォルトプラグインの読み込みや起動までの時間を計測する`--startuptime {log_file}`があります。
このオプションを使用すれば、起動までにどのようなプラグインが何msかけて読み込まれているのか丸裸になります。

<blockquote class="twitter-tweet" data-partner="tweetdeck">
    <p lang="ja" dir="ltr">
        011.502  000.002: --- NVIM STARTED ---<br><br>もぅ…、おしまいにしないと……
    </p>&mdash; yasunori-kirin0418@Vim沼 (@YKirin0418) 
    <a href="https://twitter.com/YKirin0418/status/1566454256642985986?ref_src=twsrc%5Etfw">September 4, 2022</a>
</blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

最終的に`011.502  000.002: --- NVIM STARTED ---`で起動できるようになりました。
この起動時間は、ファイルを引数に指定しない場合です。
ここではそれを前提に、起動速度の計測をしながら、起動速度を早くしていきたいと思います。


## 起動速度の調整をしていないときのstartuptime

私は[以前にも書いた][link-4]とおり、dein.vimを使用していますので、可能な限りプラグインの起動には遅延を使用していました。
それでも起動時間は150ms～170msでして、遅延起動を使用しているとは言っても対した速さで起動していないということが分ります。
ちなみにプラグイン読み込みを無効化して起動時間を計測してみると、12ms～40msで起動していました。
この数値に近付くように、いろいろと起動時間の調整をしていきます。


## tomlファイルの読み込み方をシンプルにする

まずは自身のプラグインの読み込み方法の改善しなくてはいけないところがあります。
それは、tomlファイルの読み込み方法が複雑化していました。具体的には次のような感じです。

```vimscript:init.vim
" plugins toml-file directory.
let s:toml_dir = g:base_dir ..'toml/'
let s:toml_files = systemlist('ls ' .. s:toml_dir .. '*.toml')

" Begin settings
if dein#min#load_state(s:dein_dir)

  " dein inline_vimrcs setting.(今回は省略)

  call dein#begin(s:dein_dir)

  " ================問題のコード==================
  for toml_file in s:toml_files
    if toml_file == s:toml_dir .. 'dein.toml'
      call dein#load_toml(toml_file, {'lazy': 0})
    else
      call dein#load_toml(toml_file, {'lazy': 1})
    endif
  endfor
  "==============================================

  " end settings
  call dein#end()
  call dein#save_state()
endif
```

このコード`tomlディレクトリ`にtomlファイルを置けば、特別定義しなくても読み込んでくれるので次のような利点があります。
- プラグインの種類をtomlファイルの名前で分けて、管理しやすくする。
- lsp関連・denops依存・ddc/ddu関連のプラグインetc…。
- tomlファイルが増えても勝手に読み込んでくれる。

実際、この部分ができたときは良いコード書けたと思ったのですが、起動速度という面で見れば無駄なコードになってしまいます。
ちなみにこのコードがあったときの`init.vim`の読み込みに100ms～115msもかかっていました。
ですので、`dein#load_toml(...)`をベタ書きして、複数のファイルで分けて管理していたプラグインは`lazy.toml`にまとめました。

この変更で`init.vim`の読み込みは18ms～25msまで抑えることができました。


## デフォルトプラグインの無効化

まずは起動時間を早くする方法を調べると、使用するプラグインをLua製の物にして解決していますが、根本的な解決方法ではありません。
そこで参考にしたのは次の記事です。

<!-- textlint-disable -->
- Neovimの設定を見直して起動を30倍速にした

https://zenn.dev/kawarimidoll/articles/8172a4c29a6653

- あんまり見かけない気がする Vim の Tips 11 + 1 選

https://lambdalisue.hatenablog.com/entry/2015/12/25/000046
<!-- textlint-enable -->

これらの記事ではVim/Neovimインストール時にインストールされるデフォルトプラグインの読み込みを無効にするという方法を行っています。
この方法はVim/Neovimプラグイン読み込みの際に`plugin/`ディレクトリのファイルから最小限の設定が読み込まれます。
そのディレクトリに置かれるファイルには、お作法的に次のようなコードが置かれています。

```vimscript:plugin.vim
if exists('g:loaded_plugin_name')
    finish
endif
let g:loaded_plugin_name = 1
```

内容的には`plugin`を常に読み込みに行くので、一度だけ読み込まれるようにしています。
`g:loaded_plugin_name`が起動時にあれば読み込まれることなくプラグインが終了するというしくみです。
これはデフォルトプラグインにもありまして、変数名はそれぞれですが、変数を宣言しておけば読み込みに時間がかからなくなります。

最終的に次の設定を加えました。
```vimscript:options.vim
" Disable default plugins {{{
" Fast Startup Settings!!
" Disable TOhtml.
let g:loaded_2html_plugin       = v:true

" Disable archive file open and browse.
let g:loaded_gzip               = v:true
let g:loaded_tar                = v:true
let g:loaded_tarPlugin          = v:true
let g:loaded_zip                = v:true
let g:loaded_zipPlugin          = v:true

" Disable vimball.
let g:loaded_vimball            = v:true
let g:loaded_vimballPlugin      = v:true

" Disable netrw plugins.
let g:loaded_netrw              = v:true
let g:loaded_netrwPlugin        = v:true
let g:loaded_netrwSettings      = v:true
let g:loaded_netrwFileHandlers  = v:true

" Disable `GetLatestVimScript`.
let g:loaded_getscript          = v:true
let g:loaded_getscriptPlugin    = v:true

" Disable other plugins
let g:loaded_man                = v:true
let g:loaded_matchit            = v:true
let g:loaded_matchparen         = v:true
let g:loaded_shada_plugin       = v:true
let g:loaded_spellfile_plugin   = v:true
let g:loaded_tutor_mode_plugin  = v:true
let g:did_install_default_menus = v:true
let g:did_install_syntax_menu   = v:true
let g:skip_loading_mswin        = v:true
let g:did_indent_on             = v:true
let g:did_load_ftplugin         = v:true
let g:loaded_rrhelper           = v:true

" }}}
```
書き方は人によって違いますが、変数に入れる値に関して私は`v:true`を使用しています。
`truthy`な値であればなんでも大丈夫でしょう。このあたりは、書く人にお任せするところですね。

### それでもstartuptimeからはファイル名は消えない

ファイル読み込みを抑える設定を紹介した後から、急に力技のような方法になりますが、そもそも読み込むファイルを動かして読み込まなくしてしまう方法があります。
先の設定は、あくまでもファイル読み込みが発生しても速攻で読み込みを終了させる方法になります。
この状態で、`--startuptime {log_file}`を実行しても無効化したファイルの読み込みは発生しています。
このわずかな読み込みさえ無効化したかったので、参考になる方法はないかTwitterで呟いてみたら、Shougoさんが設定で極限まで削っているという話を聞きました。

そこで教えてもらったのが、[こちら][link-3]のファイルです。
内容的にはArchLinuxでインストールした時、同時にインストールされるランタイムを消しているようです。
しかし、私はファイルを消してしまうのは怖かったので、別のディレクトリを作成して、そこに使用して欲くないデフォルトプラグインを退避させて読み込まないようにしてみました。
以下は、その際に使用できるスクリプトです。

```bash:vim_files_evacation.sh
#!/bin/bash
if [ "$EUID" -ne 0 ]; then
  echo "Please run as root."
  exit 1
fi

evacation_dir=/usr/local/share/vimfiles_evacation

etc_nvim_dir=/etc/xdg/nvim/
etc_nvim_evacation_dir=$evacation_dir/etc/xdg/

vim_dir=/usr/share/vim/vimfiles/plugin/
vim_evacation_dir=$evacation_dir/vim/vimfiles/

nvim_dir=/usr/share/nvim
nvim_evacation_dir=$evacation_dir/nvim

nvim_evacation_files=( \
  runtime/plugin/gzip.vim \
  runtime/plugin/health.vim \
  runtime/plugin/man.vim \
  runtime/plugin/matchit.vim \
  runtime/plugin/matchparen.vim \
  runtime/plugin/netrwPlugin.vim \
  runtime/plugin/shada.vim \
  runtime/plugin/spellfile.vim \
  runtime/plugin/tarPlugin.vim \
  runtime/plugin/tohtml.vim \
  runtime/plugin/tutor.vim \
  runtime/plugin/zipPlugin.vim \
  runtime/menu.vim \
  runtime/mswin.vim \
  runtime/ftoff.vim \
  runtime/filetype.vim \
  runtime/filetype.lua \
  archlinux.vim)

# Check exists evacation directory.
# When not exists make evacation directory.
if [ ! -d $evacation_dir ]; then
  mkdir -p $nvim_evacation_dir/runtime/plugin
  mkdir -p $vim_evacation_dir
  mkdir -p $etc_nvim_evacation_dir
fi

# XDG/nvim evacation directory
if [ -d $etc_nvim_dir ]; then
  mv $etc_nvim_dir $etc_nvim_evacation_dir
fi

# Vim evacation directory
if [ -d $vim_dir ]; then
  mv $vim_dir $vim_evacation_dir
fi

# Neovim evacation file
for vim_file in "${nvim_evacation_files[@]}"; do
  if [ -e ${nvim_dir%/}/${vim_file} ]; then
    mv $nvim_dir/$vim_file $nvim_evacation_dir/$vim_file
  fi
done
```

上記のスクリプトは、`/usr/local/share/vimfiles_evacation`下に、起動時に読み込まれるファイル郡が置かれていたディレクトリ構造を再現して退避させています。
仮に設定ファイルがないことで読み込みや起動に支障が出ても退避先のディレクトリを見れば、どこに戻せば良いか分かるということです。
移動させた後、業務も含めて当環境で作業をしていますが、いまの所支障らしい支障には当たっていません。
注意として、このスクリプトはArchLinux環境、かつpacmanでインストールしていた場合のディレクトリを想定しています。
自分でビルドしていた場合やArchLinuxではない場合は想定していませんので注意してください。


## 普段使用しているプラグインをそのままに、起動時間を早くする

さあ、ここまでの設定したら、私の環境ではトータルの起動時間は20msを切るようになってきます。
これ以上起動時間を設定で削る方法は、普段から使用しているプラグインを外すという方法になってきます。
しかし、それではオシャレなカラースキームなどが使えなくなってしまいます。

ここで思い出してもらいたいのは、`--startuptime`は起動するまでファイルの読み込み時間を計測しているのです。
この方法はズルと言われてしまうかもしれませんが、様は起動してからプラグインが読み込まれるようにすればよいのです。
幸い、dein.vimはそういった調整に長けたプラグインマネージャーですので、遅延起動の設定を可能な限り使用して、プラグインの起動タイミングを制御しましょう。

### Neovim限定 ～ impatient.nvim & filetype.nvim

この方法はNeovimでしか使えませんが、プラグイン読み込みを早くするためにプラグインを追加します。
「起動早くするためなのに、プラグイン追加するとか何言ってんだ」なんて、思わないでください。
ここで紹介するプラグインを適切に追加することで、そのほかのプラグインの読み込みが早くなるのです。

<!-- textlint-disable -->
- Neovimのluaモジュールの読み込みを早くして起動時間を改善する`impatient.nvim`

https://github.com/lewis6991/impatient.nvim

- filetype.vimの読み込み時間を改善する`filetype.nvim`

https://github.com/nathom/filetype.nvim
<!-- textlint-enable -->

詳しい説明は、GitHubで確認してもらうとして、導入することで起動が早くなるというのは伝わったかと思います。
また、これらのプラグインは[こちら][link-1]でも紹介されていますね。

#### dein.vimでこれらのプラグインを導入するなら、どう書くべきか

この答えは[以前書いた記事][link-4]にあります。そう、no_lazyなプラグインのディレクトリをマージしてくれる機能です。
`{'lazy': 0}`に設定したtomlファイルの中でこれらのプラグインを管理すれば、問題なく機能してくれます。

```toml:dein.toml
[[plugins]]
repo = 'nathom/filetype.nvim'

[[plugins]]
repo = 'lewis6991/impatient.nvim'
hook_add = '''
lua require('impatient')
'''
```

filetype.nvimに関してはGitHubページを見るといろいろ設定できるところがありますが、私は特別設定せずとも効果を体感しています。


### ゴリ押し設定 ～ `on_event`と`depends`で起動後に時間経過で読み込む

dein.vimにはプラグインの起動タイミングにさまざまな方法を取ることができます。
ポピュラーなところでは、ファイルタイプ毎・コマンド実行あたりだと思います。
詳しく紹介すれば、もう一本記事が書けてしまうほど多機能なプラグインマネージャーですので、ここでは省きます。
ですので今回は、`on_event`と`depends`だけに絞って、プラグイン起動の制御を解説していきます。

#### `on_event`

<!-- textlint-disable -->
Vim/Neovimにはさまざまなイベントがあり、それをトリガーとして`autocmd`でコマンドや関数を実行できます。
dein.vimではこのイベントが発生したタイミングでプラグインの読み込みを可能としています。それが、`on_event`ですね。
<!-- textlint-enable -->
このタイミングの一覧は`:h autocmd-events-abc`で確認できます。
様はこの`on_event`に起動した後で発生するイベントを指定し、プラグインが読み込まれるようにすれば、`startuptime`で出力されるログには記録されないのです。

起動後に発生するイベントのお勧めとしては、次のイベントです。

|イベント名  |説明                                                                                                                    |
|------------|------------------------------------------------------------------------------------------------------------------------|
|CursorHold  |`updatetime`の間、キー操作をしなかったら発生するイベント。<br>デフォルトでは4秒間操作しなかった場合です。`:h updatetime`|
|CursorMoved |カーソルを移動した場合に発生するイベント。                                                                              |
|InsertEnter |インサートモードに入ったときに発生するイベント。                                                                        |
|CmdlineEnter|コマンドモードに入ったときに発生するイベント。                                                                          |

これらのイベントは、ステータスライン系のプラグインやカラースキームに指定することで、起動時間をかなり短縮できます。

#### `depends`

`on_event`に紹介したイベントを指定して、起動に時間がかかるプラグインを読み込むようにしても大丈夫ですが、プラグインには設定などで依存しているプラグインがあったりします。
イベント指定にして雑にプラグインを読み込む設定にするのは楽ですが、エラーが発生してしまう可能性を回避できません。
ここで使えるのが`depends`なのです。

dein.vimでは`{'lazy': 1}`を指定したtomlファイル内でプラグインを記述した場合、明示的にプラグインの起動タイミングを指定しないと起動しません。
つまり、`on_*`系統の設定で起動タイミングを指定しなくてはいけないのです。
この仕様を活かして、プラグインの起動タイミングを指定せず、別のプラグインが起動するために依存するプラグインを指定できるのが`depends`になります。

この`depends`を利用できるかどうかで、プラグインの読み込ませ方に幅が広がります。

起動時間短縮に利用できる場面としては、次のようになります。

`lightline.vim`の設定に`vim-gitbranch`と`nightfox.nvim`を使用している。 → `depends`に配列で追加。`lightline.vim`の起動タイミングは`on_event`で解説したイベント郡を指定。
`nightfox.nvim`は`nvim-treesitter`が必要。 → `depends`に`nvim-treesitter`を指定。起動タイミングの`on_*`系統は設定しない。

例として上記の設定は、私の環境で設定している起動順になります。
これが正しく設定できていれば、起動後に次のような流れでプラグインが読み込まれていきます。

`lightline.vim`の起動タイミングとして、CursorHold/CursorMoved/InsertEnter/CmdlineEnterのいずれかのイベントが発生。
<!-- textlint-disable -->
`nvim-treesitter` → `nightfox.nvim` & `vim-gitbranch` → `lightline.vim`
<!-- textlint-enable -->

また、`depends`と似たような設定として、`on_source`という起動タイミングがあります。
こちらは、`on_source`に指定したプラグインの起動にフックして、`on_source`が設定されたプラグインを読み込むというものです。

このあたりの違いは難しいので、いろいろなプラグインの遅延起動の設定をしてみてください。


## 最終的な起動速度



## まとめ



## 参考リンク集

- [Neovimの設定を見直して起動を30倍速にした][link-1]
- [あんまり見かけない気がする Vim の Tips 11 + 1 選][link-2]
- [Shougoさんが使用するデフォルトプラグイン削除スクリプト][link-3]
- [手前味噌な記事][link-4]

<!-- リンク集(Hidden) -->
[link-1]: https://zenn.dev/kawarimidoll/articles/8172a4c29a6653
[link-2]: https://lambdalisue.hatenablog.com/entry/2015/12/25/000046
[link-3]: https://github.com/Shougo/shougo-s-github/blob/master/install-nvim.sh
[link-4]: https://qiita.com/yasunori-kirin0418/items/4ac5fc07041977a8366f
