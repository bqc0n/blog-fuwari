---
title: "Minecraft Modding (1.12.2) #1 - IBakedModel"
published: 2025-08-06
description: "IExtendedBlockStateやIBakedModelの使い方"
tags: ["Minecraft", "Minecraft Modding"]
category: "Minecraft Modding"
draft: false
---

## はじめに

`IBakedModel`とか`TileEntitySpecialRenderer`(TESR)などに関する解説記事があんまりないなーと思ったので、書いてみる。
アイテム/ブロックの追加などの基本的な部分については扱わない。素晴らしい記事がインターネット上にたくさんあるので、そちらを参照して欲しい。

:::tip[環境構築Tips]
今から1.12.2でModdingをする場合、[CleanroomMC/TemplateDevEnv](https://github.com/CleanroomMC/TemplateDevEnv)を使うと簡単に環境を構築できる([Kotlin用](https://github.com/CleanroomMC/TemplateDevEnvKt)もある)。
[GregTechCEu/Buildscripts](https://github.com/GregTechCEu/Buildscripts)も参考になると思う。
:::

### 前提知識

Javaの基本知識や、アイテム/ブロックの追加など基本的なModdingの知識があることを前提にする。

## `IBakedModel`とはなに?

本題に入る。

`IBakedModel`は、TESRと同じように動的なレンダリングを行うものである。
TESRが"active"にTileEntityを描画するのに対し、`IBakedModel`は"passive"に行う。
つまり、TESRの`render`メソッドは常に呼ばれ続けるが、`IBakedModel`の`getQuads`メソッドは、周りのブロックが変更された時などにのみ呼ばれる。

## 作ってみる
w
まずは、`IBakedModel`を実装して、読み込まれるようにする。

