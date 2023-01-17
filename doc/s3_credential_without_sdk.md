# AWSのS3にSDKなしで認証リクエストを作りたかった話

お仕事でSDK使えないよぉ…。

だけど、頑張ってS3から画像は取得しないと…。

## 前提

外部の別プロジェクトでAWS S3にアップロードされた画像を、どうにかして取得する。

* Ruby on Rails(いにしえのバージョン)
* アクセスキーIDとシークレットキーはもらえている
* AWS SDKは使えない(重要)
    * いにしえのバージョンでも使えるのか？
* AWS CLIは使えるけど、今回は対象外
* AWS STSは使っていない

大分縛りがキツいけど、がんばるぞい。

## 調査からリクエストしてみるまで

https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/RESTAuthentication.html

まずは、この方法で取得できる参考資料として、上記のリンクが共有される。

調査してみて、STS使えれば認証情報がもらえるし、たたければ楽できそう？
と思ったが、別プロジェクトではSTSは使っておらず、認証ヘッダを使用していると聞く。

チームメンバーと相談して、結果的に認証ヘッダを使って画像を取得してみようということになる。

とりあえずの調査で、以下のようなサンプルコードを用意してみた。

```ruby
require 'net/http'
require 'uri'
require 'openssl'
require 'base64'

# Sample URL
# https://example-bucket.s3.us-west-2.amazonaws.com/photos/image.jpg

def file
  access_key_id = 'access_key_id'
  secret_key = 'secret_key'
  file_uri = URI.parse('https://example-bucket.s3.us-west-1.amazonaws.com/photos/image.jpg')
  bucket_name = file_uri.host.split('.')[0]     # example-bucket
  file_path = file_uri.path                     # /photos/image.jpg
  request_date_time = Time.now.strftime('%a, %d %b %Y %H:%M:%S %z')
  string_to_sign = "Get\n\n\n#{request_date_time}\n/#{bucket_name + file_path}"
  signature = Base64.encode64(OpenSSL::HMAC.digest('sha1', secret_key, string_to_sign))

  aws_auth = "AWS #{access_key_id}:#{signature}"

  http = Net::HTTP.new(file_uri.host, file_uri.port)
  http.use_ssl = file_uri.scheme === 'https'
  headers = {
    'Authorization' => aws_auth,
    'Date' => request_date_time
  }

  begin
    request = Net::HTTP::Get.new(uri.path, headers)
    response = http.request(request)
  rescue => ex
    @logger.error(ex.message)
  end
  p response
end
```

実際には認証情報を作る部分までのコードだったけど、記事にしようと思ったので、Getリクエストまで書いてみた。
上記のコードは実際に作ったコードと近く、大まかな処理は変わらない。
強いて言うなら、どこから値をもらってくるのかくらいだろうか。

動かせるまでにタイムゾーンを合せたり、時間のフォーマットを合わせたりいろいろあったが、上記のコードが第一段階のゴールだ。

これでファイルを取得できるだろうと思ったのだが、まさかの落し穴が…。


## 認証ヘッダにも種類があるという事実

調査の段階ではこれで行けるだろうと思っていたが、実際に動かしてみると、まさかの落とし穴があった。

> The authorization mechanism you have provided is not supported. Please use AWS4-HMAC-SHA256.
>> お客様が入力された認証メカニズムはサポートされていません。AWS4-HMAC-SHA256をご利用ください。

<!-- textlint-disable -->
*`AWS4-HMAC-SHA256`*

**`AWS4-HMAC-SHA256`**

***`AWS4-HMAC-SHA256`***


「えっ、何キミ？」
<!-- textlint-enable -->

この認証方式にも種類があったようで、`AWS4-HMAC-SHA256`という方式を使用しなくてはいけなかったらしい。
じゃあ、最初にこれで行けるって言ったURLは何だったんだ…。

パッと見て、ハッシュ方式が`SHA1`から`SHA256`にすれば良いんだろうと安易に思ったのだが、変更してもエラーメッセージは変わらなかった。
`AWS4-HMAC-SHA256`でググってみるとまさかの違うAWSの記事がヒットする。

https://docs.aws.amazon.com/ja_jp/general/latest/gr/sigv4_signing.html

ここに来て、ヘッダ情報に追加ではなく、クエリパラメータとして一時認証する方法が出てきた訳だ。

考えるのを辞めたくなったが、アジャイルな現場ゆえに、何とか今スプリント中には終わらせたいところ。
がんばるぞい…。

## とりあえず、順番にリクエストに必要なデータを作っていく

https://docs.aws.amazon.com/ja_jp/general/latest/gr/sigv4-create-canonical-request.html

まずは正規リクエストというものを作っていく。
目的として、署名文字列に含めてあげる必要があるので、正規リクエストを作らなくては何も始まらない。

> 例 正規リクエストの擬似コード

```
CanonicalRequest =
    HTTPRequestMethod + '\n' +
    CanonicalURI + '\n' +
    CanonicalQueryString + '\n' +
    CanonicalHeaders + '\n' +
    SignedHeaders + '\n' +
    HexEncode(Hash(RequestPayload))
```

<!-- textlint-disable -->
うっ…情報量が最初よりも多い……
が、がんばる…ぞい……
<!-- textlint-enable -->

この`CanonicalRequest`は、最初の調査の段階でも見た内容なので作れる！
…そう、思ってた。


### 分かる範囲のデータを並べてみる

この記事を書いている段階では、もう使わない方法になったので、とりあえず情報を羅列してみる。
*ここからは調査だけで、ほとんど動かすことができてないので、断片的になります。* 

#### HTTPRequestMethod

これはHTTPのリクエストメソッドで、今回の場合は`Get`を使用する。

#### CanonicalURI

URIのパスで良いみたいだ。
サンプルURLから`/photos/image.jpg`だけで良い。
また、S3だとエンコードをしなくても良いらしい。

<!-- textlint-disable -->
#### CanonicalQueryString
<!-- textlint-enable -->

これに関しては、今回は不要。


#### CanonicalHeaders

これはちょっと面倒で、とりあえず今回の場合は以下のようになる。

```ruby
request_date_time = Time.now.strftime('%a, %d %b %Y %H:%M:%S %z')
canonical_headers = <<'EOS'
content-type:image/jpeg; charset=utf-8\n
host:s3.amazonaws.com\n
x-amz-date:#{request_date_time}\n"
EOS
```

#### SignedHeaders

ここでは、先程の`CanonicalHeaders`に羅列したヘッダ情報のヘッダ名をセミコロンでつなげてあげる必要がある。

```ruby
signed_headers = 'content-type;host;x-amz-date'
```

#### RequestPayload - もう…諦めた…

さぁ、あともう少しで正規リクエストが完成だ。

> SHA256 のようなハッシュ (ダイジェスト) 関数を使用して、HTTP または HTTPS リクエストの本文のペイロードからハッシュ値を作成します。署名バージョン 4 では、ペイロードにテキストをエンコードするための特定の文字エンコードを使用する必要はありません。ただし、一部の AWS サービスでは、特定のエンコードが必要な場合があります。詳細については、そのサービスのドキュメントを参照してください。

> 例 ペイロードの構造

```
HashedPayload = Lowercase(HexEncode(Hash(requestPayload)))
```

？？？？？？？？？

`requestPayload`はサンプル的にはどうしたら、ええんや…。

> 署名文字列を作成するときには、ペイロードをハッシュするのに使用した署名アルゴリズムを指定します。たとえば、SHA256 を使用した場合、署名アルゴリズムとして AWS4-HMAC-SHA256 を指定します。ハッシュペイロードは、小文字の 16 進文字列として表す必要があります。
> ペイロードが空の場合、ハッシュ関数への入力として空の文字列を使用します。IAM の例では、ペイロードは空です。
> 例 ハッシュされたペイロード (空の文字列)

```
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

？？？？？？？？？？？？？？？？？？

<!-- textlint-disable -->
…

……

………

もう……ムリぃ！
<!-- textlint-enable -->



## 最終的に、無理という結論に至る

知識不足ということもあるが、ここまでの調査と実装で、半日以上時間を使用し、ゴールは見えず…。
しかも、残り時間も少なく、これ以上時間を掛けてられないと判断。
当の別プロジェクトではSDKを使っていたようで、認証ヘッダを自作するというようなことはしていなかった。

そもそも、SDKを使わずにAWSとやりとりするのって難しくない？
これ、ある意味で「オレオレSDK」になっちゃうよぉ…。

## 署名にはバージョンがある(☆ 超☆ 重☆ 要☆)

さすがに、ただ「できませんでした」と言う訳には行かないので、いろいろ調べてみた。

[このページ](https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html) の上から2つ目のノートに詳細が書いてある。

初期調査の段階の方法は署名バージョン2という方法で、2014年1月30日より以前にあったリージョンで使われていた署名だった。
そして、2014年1月30日以降に増えたリージョンでは、署名バージョン4しか対応していないということだった。

署名バージョン2をサポートしているリージョンは、[ここ](https://docs.aws.amazon.com/ja_jp/general/latest/gr/signature-version-2.html)で確認できる。

## 最終的に…

ここまでの情報をまとめて相談してみたら、別の方法を検討してくれることになった。

「終わりそうにないなら早めの相談して」というありがたいお言葉もいただき、AWSのドキュメントとのバトルが終わったのであった…。

時間が無限にあったら、実装できたのか。これに関しては分からない。できるかもしれないとしか言えない。
ただ、ドキュメントを調べまくって、動かせるようになるのは好きだから、良い経験ができたと思っている。

まぁ、終わりそうにない作業ではあったので、区切りがついたのでとりあえず一安心です…。
