# Qt5.15.6で動かない…だと…

Arch Linuxをいつものように使っていて、あるときいくつかのアプリケーションが動かないことに気がつきました。
今回は動かなかったfcitx5を代表例として、ターミナルから起動させてみました。

```zsh:terminal
% fcitx5-configtool
Cannot mix incompatible Qt library (5.15.5) with this library (5.15.6)
[1]    41884 IOT instruction (core dumped)  fcitx5-configtool
```

> 互換性のない Qt ライブラリ (5.15.5) をこのライブラリ (5.15.6) と混在させることはできません。

どうやら、Qt5のライブラリに`5.15.(5|6)`が混在するせいで、うまく動かないようです。
ですので、Qt5のライブラリで`5.15.6`をグレードダウンさせて、`5.15.5`に統一することで解決します。

## TL;DR

解決手段として、AURに`downgrade`というツールがありますので、そちらを使用します。
また、ダウングレードするパッケージ量が多いので、パッケージの指定方法をワンラインで解決してみました。

```zsh:terminal
paru -S downgrade     # downgradeをAURヘルパーでインストールします。
paru -Qs qt5 | rg '5.15.6' | sed -r "s/local\///g" | awk -F'[ ]' '{print $1}' | tr '\n' ' ' > pkglist.txt && sudo downgrade `cat pkglist.txt` && rm pkglist.txt
```

今回のコマンドで、注意してほしいこと。
- AURヘルパーには`paru`を使用していますが、ほかのAURヘルパーでも問題なく動くかと思います。
- `downgrade`でのダウングレードバージョンの指定方法は、`fzf`でインクリメントサーチすることになるので、パッケージの数だけ都度指定する必要があります。
- `downgrade`ではダウングレードしたパッケージを`pacman.conf`の`IgnorPkg`に追加する機能もありますが、コマンド終了後に`pacman.conf`を確認した方が良いです。
    - `IgnorePkg`への指定されているパッケージリストが酷いことになります。

あとは、ワンラインで何をやっているのかの解説になります。
よろしければ最後までお付き合いください。
