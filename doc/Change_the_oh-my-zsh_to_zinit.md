##はじめに
数々のQiitaの記事を読んでいると、やってみたいことが色々あります。
そんな中でも、シェル環境は普段の操作にいろいろ影響があるものです。
初投稿ということで、まずはお手軽そうな内容から始めてみます。


##環境
OS:Manjaro-i3wm
職業はITとは無縁ですが、少しでもプログラマーに近づくため、形から入っています。
<div><details><summary>飛ばしても問題ないようなお話</summary>
過去にUbuntu系のディストリをいじっていて、そのときにi3wmやZshに出会いました。
ただ、アップデートを繰り返すと環境が壊れ、そのたびに再インストールしていたのですが、
Arch系LinuxのManjaroはちょっと初心者脱却にはいいかもしれないと思って使っています。
しかし、その初心者脱却と言いつつ、テンプレートのようにOh-my-zshを入れてしまったという……
これはそんな、脱初心者へ向かう私が四苦八苦している記事です。
</details></div>


##Zsh
Zsh環境について思うことをいくつか述べていきます。


###Oh-my-zsh
Zshを導入するとなると、必ずセットで紹介されるのはOh-my-zshやPreztoです。
Google検索でZshの導入について調べると、Oh-my-zshを紹介する記事が多く見られます。
そして、Oh-my-zshについて深く調べていくと、「重い、時代遅れ、組み込みプラグインがメンテされてない」等など、
あまり良い印象ではない。
そもそもOh-my-zshが色々いい感じにしてくれているので、初心者感があるのです。


###そこで出会ったのがZinitだった
Zshには様々なプラグインが作られています。
このプラグインを高速で読み込んでくれるのが、Zinitになります。
GitHubのページはこちら

https://github.com/zdharma/zinit

Qiitaだとこの記事が詳しい。

https://qiita.com/9sako6/items/23b164a95b91c644aade

![Zshプラグインを使用したときの、起動時間の比較](https://raw.githubusercontent.com/zdharma/zinit/images/startup-times.png)
↑そしてこんな比較があったら、導入するしかないじゃない！
脱初心者するなら、Zinitに切り替えて、Zshについて深く知ろうというわけです！


##導入
###Step1　Oh-my-zshのアンインストール
最初にインストールされているOh-my-zshをアンインストールします。

```terminal:terminal
$ uninstall_oh_my_zsh
```

これだけでアンインストールは完了します。
この時、Oh-my-zshをインストールする前の`.zshrc`のバックアップを残していた場合、それに置き換わります。
Oh-my-zshで設定していた内容は新たにバックアップが取られます。
自分はバックアップが取られるか分からず、コマンドで無駄にバックアップを取ってしまいました。


###Step2　.zshrcの設定
Oh-my-zsh導入で、特にいじらずとも、最初から見た目や補完機能など、使いやすいようにしてくれます。
しかし、Oh-my-zshが無くなった今、何の味付けもされていないZshになってしまいました。ここからは`.zshrc`に設定していきます。
主に参考にしたのは以下のリンク

https://www.rasukarusan.com/entry/2019/07/17/201912

https://zenn.dev/k4zu/articles/zsh-tutorial

```shell:.zshrc
#################################  HISTORY  #################################
# history
HISTFILE=$HOME/.zsh_history     # 履歴を保存するファイル
HISTSIZE=100000                 # メモリ上に保存する履歴のサイズ
SAVEHIST=1000000                # 上述のファイルに保存する履歴のサイズ

# share .zshhistory
setopt inc_append_history       # 実行時に履歴をファイルにに追加していく
setopt share_history            # 履歴を他のシェルとリアルタイム共有する

setopt hist_ignore_al_dups     # ヒストリーに重複を表示しない
setopt hist_save_no_dups        # 重複するコマンドが保存されるとき、古い方を削除する。
setopt extended_history         # コマンドのタイムスタンプをHISTFILEに記録する
setopt hist_expire_dups_first   # HISTFILEのサイズがHISTSIZEを超える場合は、最初に重複を削除します


# enable completion
autoload -Uz compinit; compinit

autoload -Uz colors; colors

# Tabで選択できるように
zstyle ':completion:*:default' menu select=2

# 補完で大文字にもマッチ
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Z}'

# ファイル補完候補に色を付ける
zstyle ':completion:*' list-colors ${(s.:.)LS_COLORS}

# ディレクトリ名の補完で末尾の / を自動的に付加し、次の補完に備える
setopt auto_param_slash

# カッコを自動補完
setopt auto_param_keys

# ファイル名の展開でディレクトリにマッチした場合 末尾に / を付加
setopt mark_dirs

# 補完キー連打で順に補完候補を自動で補完
setopt auto_menu

# スペルミス訂正
setopt correct

# コマンドラインでも # 以降をコメントと見なす
setopt interactive_comments

# コマンドラインの引数で --prefix=/usr などの = 以降でも補完できる
setopt magic_equal_subst

# 語の途中でもカーソル位置で補完
setopt complete_in_word

# 日本語ファイル名を表示可能にする
setopt print_eight_bit

# ディレクトリ名だけでcdする
setopt auto_cd

# ビープ音を消す
setopt no_beep

# コマンドを途中まで入力後、historyから絞り込み
autoload -Uz history-search-end
zle -N history-beginning-search-backward-end history-search-end
zle -N history-beginning-search-forward-end history-search-end
bindkey "^P" history-beginning-search-backward-end
bindkey "^N" history-beginning-search-forward-end



# lsコマンドのalias関連
alias ls='ls --color=auto -G'
alias la='ls -lAG'
alias ll='ls -lG'

# clearコマンドのalias関連
alias c='clear'
alias cc='c &&'
```

参考リンクから丸コピしたような内容だけど、何をしているのかを分かっているかどうかと言うのは、必要なことだと思っています。
参考リンク外から取ってきた設定が有りましたらすいません。
あちこち見て回って、「いいなコレ」と思ってソース元の記録取らずに、設定に加えてしまったものです。
aliasの設定に関してはまだ拙い所が多いので、これからも色々いじっていきたい所。


###Step3　Zinitをインストール
ある程度設定が落ち着いたので、Zinitをインストールします。
[ZinitのGitHub](https://github.com/zdharma/zinit#option-1---automatic-installation-recommended)ページに書いてあるワンライナーでインストールしました

```terminal:terminal
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/zdharma/zinit/master/doc/install.sh)"
```

コマンドを実行すると、インストールが開始され、途中でプラグインを入れるかどうか聞かれたので、自分はとりあえず入れておきました。
これが何をするのか、よく分かりませんでした。
とりあえず、説明しているページのリンクは貼っておきます。

https://zdharma.github.io/zinit/wiki/Annexes/

自分でプラグイン作る時には使うのかなぁ……何て思ったり。


###Step4　プラグインの追加
色々なプラグインが開発されていますが、これは必ず必要と思う物から入れます。

|Plugin|説明|
|:--|:--|
|[zsh-users/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)|コマンド入力中に候補出してくれる。|
|[zdharma/fast-syntax-highlighting](https://github.com/zdharma/fast-syntax-highlighting)|コマンドに色を付けて強調してくれる。|

このあたりは定番ですね。Zshを使うとなると、必ず紹介されているものですね。
以下は自分が追加で入れたプラグインです。
こっちは、あってもなくても困らないけど、便利な奴らです。

|Plugin|説明|
|:--|:--|
|[zsh-users/zsh-completions](https://github.com/zsh-users/zsh-completions)|Tabを押して出てくる補完の強化。|
|[zdharma/history-search-multi-word](https://github.com/zdharma/history-search-multi-word)|Ctrl+Rでコマンド履歴を検索。|
|[b4b4r07/enhancd](https://github.com/b4b4r07/enhancd)|ディレクトリ移動の強化。<br>[詳しくはこちら](https://qiita.com/b4b4r07/items/2cf90da00a4c2c7b7e60)ご覧ください。|
|[romkatv/powerlevel10k](https://github.com/romkatv/powerlevel10k)|プロンプトの見た目を良くして、動作を早くしてくれる。|

Pulginのインストールは各GitHubのページに説明がなければ、以下の2行を`.zshrc`に記述するだけで大丈夫です。

```zsh:.zshrc
zinit ice wait'!0'
zinit light GitHubユーザー名/リポジトリ名
```

デフォルトでGitHubからCloneして勝手にインストールしてくれます。
2行目だけでもプラグインのインストールと起動は問題ないですが、1行目のコードを加えることで、起動が爆速になります。


####Turbo modeとIce modifiersについて
`.zshrc`が読み込まれてからシェルが起動しますが、Zinitには`.zshrc`読み込み後にプラグインが起動してくれるようにできます。
これをTurbo modeと呼び、それを行っているのが1行目のコードになります。
`zinit ice wait'!0'`と書いていますが、!マーク無しで記述すると、プラグインを読み込んだことを知らせるメッセージが表示されます。
!マークを付けておくことで、プラグイン読み込み後にプロンプトをリセットして、メッセージを消してくれます。

その他にも、メッセージをそもそも表示しないようにしたり、プラグインを読み込む前後で、別の処理を実行出来るようにしたり色々できます。
それらはIce modifiersという物で、正直自分も把握しきれてないので、割愛しますが、活用することでさらなる高速化や工夫が出来るみたいです。


####Powerlevel10k
ターミナルにNerdFont系を設定しておくことで、プロンプトにアイコンを使えます。自分はURxvtに公式推奨のフォントをセットしています。
また、インストール後、初回起動時に質問形式でプロンプトの見た目を設定できるようになっています。
長くなるので、設定内容は省きますが、自分は下の画像のようにしてます。

![20210425-142757.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/936194/d76ffa4c-a244-28fe-e7e8-e566e2e82a71.jpeg)

この辺りはOh-my-zshを使ってた頃と変わりないので、この画面に戻って安心感が有りますね。


##導入後の感想
見た目は今まで通りだけど、プラグインで凄い便利になりました。
Oh-my-zshと、比べてどう早いのかと言われたら、そこまで検証をしている訳では無いので、具体的な事は言えませんが、サクサク動くようになった気がします。


##さいごに
作業的にはそんなに時間はかかってないですが、投稿するために文章にする作業は大変ですね。
ここまでお読みいただきありがとうございます。
