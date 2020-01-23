---
title: Cryptography笔记
date: 2020-01-19 15:47:24
tags: 密码学
mathjax: true
description: 本学期选了David Wu的密码学课程，是一门比较硬核的课，本博文用来整理一下相关的笔记内容。
---
## Overview
Cipher定义： ($K$, $M$, $C$)，其中$K$是key-space，$M$是message-space，$C$是cyphertext-space。并且包含了`Encrypt`, `Decrypt`即加密与解密算法。

Encrypt: $K$ x $M$ -> $C$

Decrypt: $K$ x $C$ -> $M$

两个算法必须`efficiently-computable`即多项式时间内完成。

正确性：
${\forall}$ $k$ ${\in}$ $K$, ${\forall}$ $m$ ${\in}$ $M$: Decrypt($k$, Encrypt($k$, $m$)) = $m$

One-time pad:

$K$ = {0,1}$^{n}$, $M$ = {0,1}$^{n}$, $C$ = {0,1}$^{n}$

Encrypt($k$, $m$): $c$ = $k$ ${\oplus}$ $m$

Decrypt($k$, $c$): $m$ = $k$ ${\oplus}$ $c$

Perfect secrecy定义：一个cipher(Encrypt, Decrypt)满足perfect secrecy如果对于所有的messages, $m0$,$m1$ ${\in}$ $M$, 所有的ciphertext $c$ ${\in}$ $C$:
$$ Pr[k \xleftarrow{R} K: Encrypt(k, m0) = c] = Pr[k \xleftarrow{R} K: Encrypt(k, m1) = c]$$ 

这里可以直观的理解为在key随机的情况下，不同的message加密成为同一个ciphertext的几率都是相同的。