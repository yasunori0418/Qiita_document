## はじめに
ManjaroでXMind8のインストールから、起動するまでに躓いた事の記録です。
しかし、最終的に使用することにしたのはXMind2020です。

___結論:XMind2020を使った方が断然いい！___ 

## インストール
AURにXMindのパッケージが配布されていたので、yayを使ってインストールしました。

```zsh:terminal
yay -S xmind
```

この時、XMind8とXMind2020のどちらかにするか聞かれたので、デフォルトになっていたXMind8にしました。

起動はrofiを使いましたが、ターミナルを使うなら以下のコマンドで起動します。

```zsh:terminal
XMind
```

### その1 _起動しない_
しかし、エラーが出て起動しません。この時はインストールが失敗したと思い、再インストールを何度かしましたが解決せず。
調べたら依存しているパッケージとフォントがインストールしているなら、起動するようです。面倒だったので別の手段を見つけました。
[こちらの記事](https://qiita.com/ocean_f/items/21c86c2f2fb97f5d27c0#xmindiniの編集)で`XMind.ini`を編集して、`Java-8-OpenJDK`を使うようにする必要がある事が分かりました。

`XMind.ini`は、yayでインストールした場合、以下のディレクトリに有ります。

```zsh:terminal
/usr/share/xmind/XMind/XMind.ini
```

### その2 _OpenJDKのインストール_
`Java-8-OpenJDK`を確認しに行くと、インストールされていませんでした。これもエラーの原因かもしれませんが、`PKGBUILD`で依存関係は解決してくれたような……
とりあえずインストールして、ディレクトリは以下のようになってます。

```zsh:terminal
yay -S java-8-openjdk #インストール

/usr/lib/jvm/java-8-openjdk/bin #ディレクトリ
```

インストールはしましたが、古いバージョンのものを使うという事に、このとき違和感を覚えました。

### その3 _XMind.iniの編集_
管理者権限での編集が必要なので、vimを使って編集しました。

```zsh:terminal
sudo vim /usr/share/xmind/XMind/XMind.ini
```

[参考記事](https://qiita.com/ocean_f/items/21c86c2f2fb97f5d27c0#xmindiniの編集)の通り、`Java-8-OpenJDK`を使えるように以下の2行を追加します。

```ini:XMind.ini
-vm 
/usr/lib/jvm/java-8-openjdk/bin
```

### その4 _起動するけど……_
ある程度やれる事はやりましたが、コマンドを実行しても起動しません。
そこで試しに`sudo`を使って見ると、何のエラーも出さずに起動しました。
_でも……`sudo`付けなきゃダメ？_


## 起動スクリプト
ターミナルから起動するのは面倒です。
rofi等のランチャーで起動するにしても`sudo`を付けて、ターミナルへパスワードを入力するなんて、スマートじゃないですよね。

なので`Zenity`を使って、パスワード入力用ダイアログを表示し、それで起動できるシェルスクリプトを作ることにしました。

###　`Zenity`
`Zenity`は通知やダイアログを表示できるプログラムで、シェルスクリプトと合わせれば、簡単なGUIプログラムが作れます。
今回は`Zenity`のパスワード入力用ダイアログを表示させます。

[マニュアル](https://help.gnome.org/users/zenity/3.32/password.html.ja)を参考に以下のようにしました。

```bash:xmind.sh
##!/usr/bin/env bash

pass=`zenity --title "User authentication" --password`
case $? in
    0) echo $pass | sudo -S XMind
    ;;
    1) echo "Authentication has been canceled."
    ;;
    -1) echo "Unexpected error." 
    ;;
esac
```

`sudo`のパスワードを受け入れる部分は[こちら](https://stackoverflow.com/questions/11955298/use-sudo-with-password-as-parameter)を参考にしました。
上のスクリプトをパスの通る場所に置いて、実行権限を与えてあげれば��成です。これで、ランチャーから起動した時にダイアログが表示されて、パスワードを入力すれば問題なく起動します。

## 画面のちらつき
これに関しては、使用環境によるものなのか分かりませんが、マインドマップを編集していると、画面がちらついて集中できないのです。
調べてみても、他に同じような症状を訴えている人を見つけることができず、XMind8の使用を断念しました。

## さいごに
結果として、冒頭で述べたようにXMind2020をインストールすることで解決しました。
こちらに関してはyayでインストールすれば、普通に起動できます。しかも、起動が早い！
なんだかんだ、遠回りした気がしましたが、`Zenity`を使った起動スクリプトの辺りに関して、このまま消してしまうのは勿体なかったので記事にしました。

## 参考リンク
[Ubuntu 18.04にXMind 8をインストール＆Dockから起動](https://qiita.com/ocean_f/items/21c86c2f2fb97f5d27c0)
[use-sudo-with-password-as-parameter](https://stackoverflow.com/questions/11955298/use-sudo-with-password-as-parameter)
