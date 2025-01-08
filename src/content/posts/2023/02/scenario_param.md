---
title: シナリオコードのロジックについて
published: 2023-02-13
description: スプラトゥーン3で利用されているシナリオコードについて解説します
category: Nintendo
tags: [Splatoon3]
---

## シナリオコード

シナリオコードとはオンラインまたはプライベートで遊んだサーモンランのリザルトを任天堂のサーバーに送ることによって得られる 16 桁のコードのことです。

このコードを共有することで、送信したサーモンランのリザルトと同じ内容のゲームを遊ぶことができます。

### シナリオコードの仕組み

そもそも、サーモンランには全部で五つのバイトタイプが設定されています。これらのうち通常プレイで遊べるのは「いつものバイト」「プライベート」「ビッグラン」の三つです。残りの二つは内部的には実装されてはいるもののまだリリースされていません。

|                  | いつものバイト |  プライベート  | ビッグラン |
| :--------------: | :------------: | :------------: | :--------: |
|     内部 ID      |       0        |       1        |     4      |
|       保存       |      可能      |      可能      |    不可    |
|       ブキ       |      固定      |    自由選択    |     -      |
|     キケン度     |  シナリオ依存  |  シナリオ依存  |     -      |
|     ステージ     |  シナリオ依存  |  シナリオ依存  |     -      |
| オカシラメーター |  シナリオ依存  |  シナリオ依存  |     -      |
|  オカシラシャケ  |  シナリオ依存  | -1(出現しない) |     -      |
|   ランダムブキ   |  シナリオ依存  | -1(出現しない) |     -      |
|      シード      |      固定      |      固定      |     -      |
|     シーズン     |  シナリオ依存  |  シナリオ依存  |     -      |
|  タイムスタンプ  |  シナリオ依存  |  シナリオ依存  |     -      |
|  シナリオコード  |      固定      |      固定      |     -      |

ビッグランはシナリオコードを保存できないので、実質的にシナリオコードで遊べるのは「いつものバイト」と「プライベート」の二つです。

これらの違いですが、簡単に言うと以下の

- ブキ変更ができない
  - プライベートであれば、固定されているように見えてもブキ変更ができます
  - いつものバイトは変更することができません
- ブキがローテーションしない
  - プライベートではランダム選択時以外はブキが常に固定されます
  - いつものバイトでは WAVE 開始時に使っていないブキからランダムに支給されます
- オカシラシャケが出現しない
  - 内部的にいじってもオカシラシャケが出現しません
- 緑ランダムからレアブキが支給される
  - プライベートではレアブキは支給されません
  - いつものバイトではそのバイトで支給されていたレアブキ枠が支給されます