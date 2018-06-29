---
title: <译> Some mistakes in RxSwift you want to avoid
date: 2017-11-24 15:25:51
tags:
---

[原文地址](http://adamborek.com/top-7-rxswift-mistakes/)

<!-- more -->

## 0x01 combineLatest vs withLatestFrom

Observable 通常情况下是 lazy 的。意思是它在被订阅前不做任何事情的。有时候你会像下面这样用 Observable 包装一层：

<script src="https://gist.github.com/TheAdamBorek/7d9f1d24c1ce5e5a257a13f17e5bc8ff.js"></script>

你可以用 `just` 直接将某一个值包装成 Observable。但是如果这个值需要大量复杂计算呢？你可以用 `deferred` 来将这个值的计算推迟到当有新订阅者订阅的时候。

<script src="https://gist.github.com/TheAdamBorek/6eca215185c0c269682f8d1f68e35d55.js"></script>

还有像 `create`, `just` 

## 0x02 Observable should be lazy

## 0x03 Using wrong DisposeBag

## 0x04 Not using drivers on UI

## 0x05 Error handling

## 0x06 Subscribing multiple times into 1 Observable

## 0x07 Overusing subjects & variables