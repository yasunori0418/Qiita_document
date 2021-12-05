## はじめに
デスクトップ環境のLinuxのシステムモニターと言えばConkyですが、GitHubやDeviantart等など、調べるとイケてるウィジェットがいっぱい出てきます。
でも、Linux使ってるんだから、人が作ったものより、自分が作ったやつでデスクトップを飾りたいと思いませんか？
今回紹介する`conky-draw`は、そんなあなたにおすすめです。


## conky_draw

https://github.com/fisadev/conky-draw

簡単に説明するなら、いっぱいLuaを勉強しなくても、Conkyに円グラフを描写することが出来ます。
GUIも開発されているようですが、私は上手く動かせなかったので割愛します


### インストール
まずは、GitHubからクローンしましょう。

```terminal:terminal
git clone https://github.com/fisadev/conky-draw
```

クローンしてきた物の中に`conky_draw.lua`と`conky_draw_config.lua`が有ります。
それらを`~/.conky`にコピーしましょう。

```terminal:terminal
cd conky-draw
cp *.lua ~/.conky
```

使用環境により異なりますが、大体はそこに`.conkyrc`があるはずです。
`.conkyrc`の`conky.config`内に以下を記述します。

```lua:.conkyrc
lua_load = 'conky_draw.lua', 
lua_draw_hook_pre = 'main',
```

これでインストールは完了です。

_※今回は`conky-draw`の紹介なので、`.conkyrc`の記述は省きます。
詳しくは[Conky Wiki](https://github.com/brndnmtthws/conky/wiki)をご覧下さい。_

---


## プロパティ

基本的に`conky_draw_config.lua`の`elements`内に書いていきます。
全体的な書き方はGitHubの[Examples](https://github.com/fisadev/conky-draw#examples)を参考にしてください。
ここでは、各プロパティの種類と説明をします。

---


### `kind`とrequired
最初に`kind`に描写したい物の種類を設定します。

kind|説明
---|---
ring|円を描写します。
ring_graph|円グラフを描写します。<br>conky_valueで指定した変数に応じたグラフになります。
ellipse|楕円を描写します。
ellipse_graph|楕円グラフを描写します。<br>conky_valueで指定した変数に応じたグラフになります。
line|線を描写します。
bar_graph|バーグラフを描写します。<br>conky_valueで指定した変数に応じたグラフになります。
static_text|文字を描写します。
variable_text|conky内で使える変数を描写します。<br>Conky変数のcpugraphの様な物には対応してません。

描写出来るものは以上になります。描写時に必須のプロパティは以下の表になります。

required|kind|説明 
---|---|---
from|line<br>bar_graph<br>static_text<br>variable_text|描写時の始点<br>`{x=?,y=?}`の形式で指定
to|line<br>bar_graph|描写時の終点<br>`{x=?,y=?}`の形式で指定
radius|ring<br>ring_graph<br>ellipse<br>ellipse_graph|円の半径
center|ring<br>ring_graph<br>ellipse<br>ellipse_graph|描写時の中心座標
width<br>height|ellipse<br>ellipse_graph|楕円描写時に、高さと幅を指定して、潰しているイメージ
text|static_text|文字列
conky_value|bar_graph<br>ring_graph<br>ellipse_graph<br>variable_text|conkyで使用できる変数<br>基本グラフには、パーセンテージを入れる

---


### `max_value`
`conky_value`にパーセンテージ以外の変動する数値を入れた場合、`max_value`に正しく最大値を入れれば、パーセンテージ系の変数を入れた時と同じ振る舞いをします。
が、最大値を取得できるconky変数を`max_value`にセットしても、そこから最大値は取得してくれません。何かしら、一工夫が必要です。
素直に最大値を調べて、`max_value`に入れてしまうのが無難ですね。

---


### `color`
色の設定は、16進数表記のRGBを使用します。このスクリプトでは、`0x000000`という形で宣言します。

一覧は以下の表になりま。

プロパティ|kind|説明
---|---|---
color|line<br>ring<br>ellipse<br>static_text<br>variable_text|文字列の色
background<br>_color|bar_graph<br>ring_graph<br>ellipse_graph|グラフの背景部分の色
bar_color|bar_graph<br>ring_graph<br>ellrpse_graph|前景部分の色

---


### `thickness`
線の厚さの設定になります。

一覧は以下の表になります。

プロパティ|kind|説明
---|---|---
thickness|line<br>ring<br>ellipse|線の厚さ
background<br>_thickness|bar_graph<br>ring_graph<br>ellipse_graph|背景部分の線の厚さ
bar_thickness|bar_graph<br>ring_graph<br>ellipse_grape|前景部分の線の厚さ

---


### `alpha`
不透明度の設定になります。`alpha=0`で透明。`alpha=1`で不透明。`alpha=0.5`の様に設定してください。

一覧は以下の表になります。

プロパティ|kind|説明
---|---|---
alpha|line<br>ring<br>ellipse<br>static_text<br>variable_text|線や文字の不透明度
background<br>_alpha|bar_graph<br>ring_graph<br>ellipse_graph|背景部分の線の不透明度
bar_alpha|bar_graph<br>ring_graph<br>ellipse_grape|前景部分の線の不透明度

---


### text関連
文字を描写する時に関連するプロパティ一覧は、以下の表になります。

プロパティ|説明
---|---
font|描写テキストのフォント<br>デフォルトは"Liberation Sans"
font_size|描写テキストのサイズ<br>.`conkyrc`よりも小さい。詳しくは[こちら](#文字サイズ)
bold|描写テキストを太文字にする。<br>設定値はBoolean
italic|描写テキストをイタリック体(斜体)にする。<br>設定値はBoolean

`bold`や`italic`は、ちゃんと対応したフォントのインストールが、必須だと思われます。
また、フォントによっては、極太とか極細みたいな、フォントファミリーがあるかもしれませんが、それらには未対応の様です。

---


### `critical_threshold`とchange系プロパティ
グラフで一定の値になった時に、グラフの見た目を変えることができます。
`critical_threshold`にしきい値を設定し、しきい値以上になったら、change系プロパティに設定した通りの見た目に変わります。
この時の変化は前景背景関係なく全部変わります。
この変化を無効にしたい場合は、`max_value`と同じ値にしてください。

change系プロパティの一覧は、以下の表になります。

プロパティ|説明
---|---
change_color<br>_on_critical|しきい値以上になったら、設定した色に変化します。
change_thickness<br>_on_critical|しきい値以上になったら、設定した厚さに変化します。
change_alpha<br>_on_critical|しきい値以上になったら、設定した不透明度に変化します。

---


### angle
![GitHubより-欠けた円](https://raw.githubusercontent.com/fisadev/conky-draw/master/samples/sample5.png "GitHubより-欠けた円")
↑こういう感じの欠けた円を作ります。円関係に使えます。
使用する際は、`start_angle`と`end_angle`にそれぞれ角度を、整数で設定してください。

円を東西南北で見た時、角度は以下の表になります。

方角|角度
---|---
東|0°
南|90°
西|180°
北|270°

デフォルトでは、
```
start_angle=0, 
end_angle=360,
```
となっています。

---


### `graduation`
![GitHubより](https://raw.githubusercontent.com/fisadev/conky-draw/master/samples/graduated_ring.png)

↑設定すると、線や円の見た目をこんな感じにします。(以下:graduation)

このスクリプトを使って円グラフを作るなら、必ずは使ってみたいプロパティですね。

プロパティ|説明
---|---
graduated|線や円をgraduationにする場合は、必ずtrueにしてください。
number_graduation|目盛り(graduation)の数
space_between<br>_graduation|目盛り(graduation)の間隔

_こういうグラフの名前って正確には何て言うんですかね……_
_一応、目盛りとか言ってますけど、graduationで調べても卒業って単語しか出ないんですよね。_
_ご存知の方がいらっしゃったら、コメントください。_


## 応用編
自分でウィジェット作る時に、覚えておくと便利な物を紹介します。

### 変数
座標や色など、何度も入力するのは面倒になるはずなので、`elements`の上の方に変数を設定しておきます。

```Lua:conky_draw_config.lua
local thickness_size = 15

local center_x = 190
local center_y = 190

local minimum_radius = 50
local ring_increment_size = 18

local bg_color = 0x606060
local bg_alpha = 0.5

local fg_color = 0x006eff
local fg_alpha = 0.8
elements{
    {
        kind = 'ring_graph',
        conky_value = 'cpu cpu8',
        center = {x = center_x, y = center_y},
        thickness = thickness_size,
        bar_color = fg_color,
        bar_alpha = fg_alpha,
        bar_thickness = thickness_size,
        background_color = bg_color,
        background_alpha = bg_alpha,
        background_thickness = thickness_size,
        radius = minimum_radius + (ring_increment_size * 0),
        max_value = 100,
        start_angle = 270,
        end_angle = 0,
        graduated = true,
        number_graduation = 75,
        angle_between_graduation = 0.5,
    },{
        kind = 'ring_graph',
        conky_value = 'cpu cpu7',
        center = {x = center_x, y = center_y},
        thickness = thickness_size,
        bar_color = fg_color,
        bar_alpha = fg_alpha,
        bar_thickness = thickness_size,
        background_color = bg_color,
        background_alpha = bg_alpha,
        background_thickness = thickness_size,
        radius = minimum_radius + (ring_increment_size * 1),
        max_value = 100,
        start_angle = 270,
        end_angle = 0,
        graduated = true,
        number_graduation = 75,
        angle_between_graduation = 0.5,
    },{
        kind = 'ring_graph',
        conky_value = 'cpu cpu6',
        center = {x = center_x, y = center_y},
        thickness = thickness_size,
        bar_color = fg_color,
        bar_alpha = fg_alpha,
        bar_thickness = thickness_size,
        background_color = bg_color,
        background_alpha = bg_alpha,
        background_thickness = thickness_size,
        radius = minimum_radius + (ring_increment_size * 2),
        max_value = 100,
        start_angle = 270,
        end_angle = 0,
        graduated = true,
        number_graduation = 75,
        angle_between_graduation = 0.5,
    },{……

```

自分の作ってる奴の一部を抜粋してきました。こんな風に変数使うんだな、くらいに思って貰えれば大丈夫です。


### 文字サイズ
`.conkyrc`内の`conky.text`で文字を描写する時と、`conky-draw`で文字を描写する時とでは、フォントサイズが`conk-draw`の方が小さいです。
座標で指定できる`conky-draw`の方が文字を描写する時も便利ですが、多少なりとも、`.conkyrc`も併用したい時が出てきます。

自分は`.conkyrc`のフォントサイズに合わせる様に、以下の関数を`elements`の上の方に作って使ってます。

```Lua:conky_draw_config.lua
function font_gap(font_size)
    return font_size * 1.333
end

elements{
    {
        kind = 'static_text',
        from = {x = 100, y = 100},
        text = 'test',
        font_size = font_gap(16),
    },{……}
}
```

自分の環境で何度か試した結果、`conky-draw`で描写するテキストを`1.333倍`すれば、`.conkyrc`で描写するテキストと同じサイズになりました。[^1]


## さいごに
全プロパティの紹介には、骨が折れました。（笑）
GitHubのページを翻訳すれば、ある程度は分かる事なのですが、こうやって日本語にしておくだけでも、自分だけのウィジェット開発の参考になるので、こういう形でアウトプットするのも有りだと思いました。

このスクリプト、なかなかの神スクリプトだと思うのですが、開発が2年前で止まっているのが少々残念な所。issuesにもリクエストが幾つか来ているし、伸び代はいっぱいあるんですけどね……


### 注釈
[^1]: 自分の環境以外では違うのか、検証できてません。違うよって人が居たら教えてください。
す
