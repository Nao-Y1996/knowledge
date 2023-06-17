# Redis

- [公式ドキュメント](https://redis.io/docs/)
- [日本語ドキュメンtp](http://redis.shibu.jp/)
- [実践Redis入門](https://www.amazon.co.jp/dp/4297131420)

## Redisとは

## データ型と操作

### String

Redisでは、プログラミング言語における文字列、バイナリ、整数、浮動小数点数は全てStringで表現される。
シンプルにkeyとvalueの組み合わせが保存される。当然keyは一意になる。

- 保存
  - `set keyName value`
  - `mset key1 value1 key2 value2`複数の同時保存
- 取得
  - `get keyName`
  - `mget key1 key2`複数の同時取得
- valueの末尾に追記
  - `append key addValue`
- valueの長さを取得
  - `strlen keyName`
- valueの一部分を操作
  - `getrange keyName 0 4`1文字目から4文字目を取得
  - `setrange keyName offset value`  
    指定したoffsetの位置を起点としてvalueをセットする
- valueの数値を操作する
  - `incr key`valueが数字の時、valueを1増やす
  - `decr key`1減らす
  - `incrby key count`valueが数字の時、valueをcount分増やす
  - `decrby key decrement`valueをdecrement分減らす
  -

### List

- リストに要素を追加する
  - `lpush keyName elem1 emlm2`
    - リストの左から要素を追加する
  - 右から追加するときは`rpush`
- リストの要素を削除する
  - `lpop keyName [count]`
    - リストの左から要素を削除する。引数のcountで削除する要素数を指定する
  - 右から削除するときは`rpop`を使う
- リストの値を取得
  - `lindex keyName count`
    - 引数のcountで指定したインデックスの要素を取得する
  - `lrange keyName start stop`
    - 引数startからstopの要素を取得する
    - 正の数は先頭から負の数は末尾からを意味する
    - `lrange keyName 0 -1`で全て取得できる

### Hash

keyが複数のfieldをもち、fieldが1つのvalueを持つようなデータ構造

- keyとvalueの保存
  - `hset keyName field1 value1 field2 value2`
  - fieldがすでにある場合は上書きされる
- valueの取得
  - `hget keyName fieldName`
- fieldの削除
  - `hdel keyName fieldName`
- fieldとvalueの一覧を取得（keyを指定）
  - `hgetall keyName`

### Set

Stringの集合。順序なしで重複なく保持できる。保持されるStringのデータをmemberと呼ぶ

- データの登録
  - `sadd keyName member1 member2`
- memberの数を取得
  - `scard keyName`
- memberが集合に含まれるか判定する（key指定）
  - `sismember keyName member`
  - 1がtrueで0がfalse
- 集合に含まれる全てのmemberを取得
  - `smembers keyName`
  - 計算量はO(n)

### SortedSet

順序のあるSet型,順序はscoreと呼ばれる

- データの登録
  - `zadd keyName score1 member1 score2 member2`
- memberの数を取得
  - `zcard keyName`
- memberのスコアの順位を順番で取得（key指定）
  - 低い順：`zrank keyName member`
  - 高い順：`zrevrank keyNmae member`
- 指定したmemberのscoreを取得
  - `zscore keyName member`
- 集合に含まれる全てのmemberを取得
  - `smembers keyName`
  - 計算量はO(N)
- 集合に含まれるmemberを指定して削除
  - `zrem keyName member`
  - 計算量はO(M * N) Mはmemberの数

## Redisのトランザクション

個々のコマンドはアトミックになっているが、1連の処理をした時その処理全体ではアトミックは保証されない。
redisのトランザクション機能として`MULTI`,`EXEC`がある（RDBMSのBEGINとENDに相当）ただし、ロールバック機能は提供されていない。コマンドをキューに溜め込み、`EXEC`時に溜め込んだコマンドが一斉に実行される。
文法ミスなどでエラーが発生するとトランザクション全体が終了し、全体としては何もしない。
キーが見つからないなどの実行時エラーの時は、エラーが出るだけで後続の処理も実行される。

## Publish/Subscribe機能

ROSのpub/subと同じもの
ROSにおけるTopicはRedisではチャンネルと呼ぶ

publisherがメッセージをチャンネルに送信。
subscriberがチャンネルを購読。そのチャンネルに送られてきたメッセージを受け取る。
subscribeは複数のチャンネルを購読できる。

### pub/sub機能の仕様状況の調査コマンド

- `pubsub channels [正規表現]`  
    subscriberが１人以上いるチャンネル一覧を出力する。引数でチャンネル名の正規表現を指定した場合は該当するもののみが対象になる。引数を省略すると全てが対象。
- `pubsub numsub チャンネル名`  
    引数で指定したチャンネルのsubscriberの数を出力。

## Stream

## タスク

- [x] 各種データ型の、保存、取得方法をまとめる.
  - [x] String
  - [x] List
  - [x] Hash
  - [x] Set
  - [x] SortedSet
- [x] Pub/Subを試してまとめる
- [ ] ストリームの理解とまとめ
- [ ] PythonでRedisを使う部分を読む
- [ ] 5章
  - [ ] 永続化
  - [ ] キャッシュサーバーとしての利用
  - [ ] ベストプラクティス
  - [ ] その他(20ページほど)
