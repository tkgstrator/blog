---
title: DockerでMongoDBにレプリカセットを設定する方法
published: 2024-06-26
description: もっと良い方法があると思うのですが、とりあえず忘れないようメモしておきます
category: Programming
tags: [Docker, MongoDB]
---

## MongoDB

MongoDBにはレプリカセットという仕組みがあります。

よくわかっていないのですが、一方がぶっ壊れても復旧させるための仕組みっぽいです。

で、ここからがめんどくさいのですが通常MongoDBのイメージをdocker composeで動かせばサービス名でつながるのですがレプリカセットには単純にはそれが反映されないという問題？があるので非常にややこしいです。

あと、ChatGPTがおかしいのかはよくわからないですが訊いても素っ頓狂な答えが返ってきます。

```yaml

```