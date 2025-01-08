---
title: 将棋アプリからデータを抜くことはできますか
published: 2024-09-05
description: 将棋の棋譜データを集めるのがめんどくなので悪の道に手を染めてみます
category: Tech
tags: [iOS]
---

## 概要

### 今回選んだアプリ

- [詰将棋パラダイス](https://apps.apple.com/jp/app/id555635034)
- [詰将棋パラダイス2](https://apps.apple.com/jp/app/id1567265053)
- [詰将棋ライト](https://apps.apple.com/jp/app/id524565911)
- [Shogi](https://apps.apple.com/jp/app/id525195399)

### 検討

データは内部に保存されているか、都度サーバーから取得しているかの二通りが考えられます。

楽なのは内部に保存されているパターンで、この場合はサーバーとの通信を覗くという手間が省けます。

暗号化されていないのが最も楽なパターンですが、暗号化されていたとしても前者であればバイナリに復号のためのキーが入っているので容易に解析できます。

後者の場合は共通鍵であれば前者と同じですが、公開鍵暗号の場合には多少ややこしくなります。その場合は動的解析が必要になるでしょう。

また、内部に保存されているパターンでありがちなのが、ファイルが複数ある場合に独自の謎のフォーマットで圧縮されているパターンです。これはめんどくさいのであんまりやりたくないですね。

## 調査開始

まずはパケット解析をして動的解析の方面から攻めてみようと思います。

内部データが入っていれば一番いいのですが、HTTPSで生データをそのまま送受信してくれているとそっちの方がありがたいです。

アプリのバイナリを復号して抜き出したらSideloadlyでApple SiliconのMacにインストールします。iOSデバイスでもパケットキャプチャはできるのですが、証明書のインストール等がめんどくさいのでiOSのバイナリが動かせるMacがあるならばそちらでやったほうが楽です。

### 詰将棋パラダイス

詰めパラの場合は`https://aws.tumepara.com/tsp/works/getAllWorkList/1`というURLにリクエストを送っていることがわかりました。

何故かPOSTリクエストでフォームのデータを送っているのですが、同様のリクエストをエミュレートしてみるとこれまた何故かURLエンコードされたデータが返ってきます。

それを貼っても仕方がないのでデコードしたものを掲載します。

```
d=0&id1=0&title1=チュートリアル&level1=0&author1=ｱﾗｲﾓﾝ&ansflg1=0&id2=1&title2=よく噛んで食べよう&level2=10&author2=市原誠&ansflg2=0&id3=2&title3=飛角図式の両王手&level3=2&author3=市原誠&ansflg3=0&id4=3&title4=11角との連動&level4=1&author4=市原誠&ansflg4=0&id5=4&title5=玉を飛び道具で挟む&level5=2&author5=ｱﾗｲﾓﾝ&ansflg5=0&id6=5&title6=嫌でも詰みそう&level6=6&author6=ｱﾗｲﾓﾝ&ansflg6=0&id7=6&title7=銀を呼ぶ手段&level7=20&author7=ｱﾗｲﾓﾝ&ansflg7=0&workcnt=21590
```

アカウントを作成した段階ではレベルが1しかなく、自身のレベル以下の問題しか挑戦することが出来ません。

とりあえずレベル1の問題であるNo.3のデータを取得してみます。

`https://aws.tumepara.com/tsp/works/getWorkInfo/3`

一回のリクエストでこれだけしか取ってこないのはちょっともったいない気もするのですが、どうなんでしょうか？

すると、こちらは同じPOSTリクエストなのですがフォームに何も入れなくてもデータを取ってくることができました。同様にURLエンコードされたテキストが降ってきます。

```
d=0&workid=3&csv=0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_3611_1602_1101_1502_1211_0011&hint=取られてもいいです&progresscnt=3&answercsv=15330_36160_33220_15330_36160_11221_&title=11角との連動&authorname=市原誠&workupdate=2008/11/18[2:48:00]&point=2&level=1&key=DR7ZzwicTHEkQ1_0DNGa2FLEMsbZI3&manscsv1=15240&marrcsv1=36161&mdescsv1=▽１六飛で失敗。この竜が取られても大丈夫なように攻めよう。
```

謎フォーマットのテキストです。

KIF形式が返ってきてくれれば楽だったのですがこれは困りました。

とはいえ、なんとなくフォーマットは理解できます。

| キー        | 意味         | 
| :---------: | :----------: | 
| d           | 不明         | 
| workid      | 問題ID       | 
| csv         | 盤面         | 
| hint        | ヒント       | 
| progresscnt | 手数         | 
| answercsv   | 解答         | 
| title       | タイトル     | 
| authorname  | 作者名       | 
| workupdate  | 更新日       | 
| point       | 獲得ポイント | 
| level       | レベル       | 
| key         | 不明         | 
| marrcsv1    | 不明         | 
| manscsv1    | 不明         | 
| mdescsv1    | 不明         | 

おそらくこんな感じだと思います。

最低限必要なのは盤面だけなので`csv`だけが理解できれば大丈夫ですが、`progresscnt`と`authorname`も使えるとよいですね。

とはいえ後ろの二つはテキストデータがそのまま入っているだけなので簡単です。

```
0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_3611_1602_1101_1502_1211_0011

15330_36160_33220_15330_36160_11221_
```

なかなかに気持ち悪いフォーマットですが、おそらくアンダーバーを区切り文字としたCSVのつもりなのでしょう。

```
V2.2
P1 *  *  *  *  *  *  *  * +KA
P2 *  *  *  *  *  *  *  * -OU
P3 *  *  *  *  *  *  *  *  *
P4 *  *  *  *  *  *  *  *  *
P5 *  *  *  *  *  *  *  * +UM
P6 *  *  *  *  *  * -HI * +RY
P7 *  *  *  *  *  *  *  *  *
P8 *  *  *  *  *  *  *  *  *
P9 *  *  *  *  *  *  *  *  *
P-00KI00KI00KI00KI00GI00GI00GI00GI00KE00KE00KE00KE00KY00KY00KY00KY00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU
+
+1533UM
-3616HI
+1122UM
%TORYO
```

盤面と指し手については同様の問題をCSAで出力すると上記のようになります。

そして解答については簡略型CSA形式と考えることができます。

実はこの問題には解答が二つあり、

`+1533UM -3616HI +1122UM`でも`+1533UM -3616HI +3322UM`でも正解になります。

今回の問題では`rogresscnt=3`であることがわかっているので三つずつに分割して比較すると以下のようになります。

```
# 謎フォーマット
15330 36160 33220
15330 36160 11221

# CSA形式
+1533UM -3616HI +3322UM
+1533UM -3616HI +1122UM
```

このことから解答の指し手はCSA形式と考えて良いでしょう。謎フォーマットの最後は0か1ですが、これは1であれば「成」をさしていると考えられます。

```
0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_0011_3611_1602_1101_1502_1211_0011
```

次に盤面データですが、アンダーバー区切りで40つのデータから構成されていることがわかります。

将棋の盤面は9x9で81マスなのでこれは一致しません。さて、このデータは何を意味するのでしょうか。

将棋において40という数字がなにかということを考えると「駒の数」が浮かびます。盤面が81マスあっても駒が40しかないのであれば半分以上歩マスは駒が何もなく、「盤面の座標+そこにある駒」で局面を表現しようとすると効率が悪いわけです。

なので「それぞれの駒+盤面の座標」という形式にすれば駒の数しかデータが必要でないので40あれば十分というわけですね。

で、このフォーマットですがおそらく駒の価値が低い順に並んでいると思われるので、そのように表示方法を少し変えてみましょう。

```
# 歩18枚
0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011
# 香車4枚
0011 0011 0011 0011 
# 桂馬4枚
0011 0011 0011 0011
# 銀4枚
0011 0011 0011 0011
# 金4枚
0011 0011 0011 0011
# 飛車2枚
3611 1602
# 角2枚
1101 1502
# 玉2枚
1211 0011
```

> 本来は飛車の方が角より強いとみなされることが多いのですが、詰めパラの場合は逆でした

このとき、それぞれの駒は`XXYZ`という形式で表されています。CSAにおいて`XX=00`は駒台を意味します。`YZ`については、

| パラメータ | 0      | 1      | 2   | 
| :--------: | :----: | :----: | :-: | 
| Y          | 攻め方 | 受け方 | -   | 
| Z          | -      | 不成   | 成  | 

であると予想されます。ここでチュートリアルの問題を見てみると、

```
# 歩18枚
0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 3401 
# 香車4枚
0011 0011 0011 0011
# 桂馬4枚
0011 0011 0011 0011
# 銀4枚
0011 0011 0011 0011
# 金4枚
0011 0011 0001 2401
# 飛車2枚
0011 3511
# 角2枚
0011 0011
# 玉1枚
3211
```

となっており、玉の枚数が一枚少なくても攻め方の玉が存在しない(双玉詰将棋でない)場合にはフォーマットとして成立するようです。

これでフォーマットは完全に理解できたので、これをCSA形式に変換するTypeScriptのコードを書きます。

CSA形式にさえ変換できれば[tsshogi](https://github.com/sunfish-shogi/tsshogi)でデータを扱うことができます。

```
# CSA
V2.2
P1 *  *  *  *  *  *  *  * +KA
P2 *  *  *  *  *  *  *  * -OU
P3 *  *  *  *  *  *  *  *  *
P4 *  *  *  *  *  *  *  *  *
P5 *  *  *  *  *  *  *  * +UM
P6 *  *  *  *  *  * -RY * +HI
P7 *  *  *  *  *  *  *  *  *
P8 *  *  *  *  *  *  *  *  *
P9 *  *  *  *  *  *  *  *  *
P-00KI00KI00KI00KI00GI00GI00GI00GI00KE00KE00KE00KE00KY00KY00KY00KY00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU00FU
+
+1533UM
-3616RY
+1122UM
%TORYO
```

指し手の情報は要らないとはいえ、それでもちょっとめんどくさそうな気がします。