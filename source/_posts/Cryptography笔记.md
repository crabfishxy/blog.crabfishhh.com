---
title: Cryptography笔记
date: 2020-01-19 15:47:24
tags: 密码学
mathjax: true
description: 本学期选了David Wu的密码学课程，是一门比较硬核的课，本博文用来整理一下相关的笔记内容。1月23日更新基本概念，Cipher, OTP, Perfect Secrecy。1月27日更新semantically security, reduction证明
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

而One-time Pad是符合Perfect secrecy的，证明不表。但因为One-time Pad需要至少和messages相同的长度，并不适合实际用来加密。所以我们需要一种更加可用的方式，根据香农定理，如果要实现perfect secrecy，$|K|$ ${\geq}$ $|M|$必须满足。所以我们需要将perfect secrecy这一条件放宽，转而限制住`computationally-bounded adversaries`即可。
因此我们可以通过一个较短的`seed`生成看起来随机的`key`。

## Stream cipher:
$K$ = {0,1}$^{\lambda}$, $M$ = {0,1}$^{n}$, $C$ = {0,1}$^{n}$

Encrypt($k$, $m$): $c$ = $G(k)$ ${\oplus}$ $m$

Decrypt($k$, $c$): $m$ = $G(k)$ ${\oplus}$ $c$

其中$n$ >> ${\lambda}$

因为此时已经不符合perfect secrecy的基本条件，所以我们需要一种新的security定义，做到没有efficient adversary可以分辨出PRG产生的key和实际完全随机的key。

我们需要两个实验，实验0由challenger通过PRG生成伪随机的t交给adversary来判断，实验1由challenger生成完全随机的t交给adversary来判断，而adversary输出0或1来表明是实验0还是实验1。设：
$$
W_0=Pr[adversary\;outputs\;1\;in\;Experiment\;0]\\
W_1=Pr[adversary\;outputs\;1\;in\;Experiment\;1]
$$
然后定义distinguishing advantage of A为
$$PRGAdv[A,G] = |W0-W1|$$

因此有定义：

如果对于所有的efficient adversaries A来说，$PRGAdv[A,G] = negl({\lambda})$，一个PRG是secure的

除了上述两个实验，我们还可以定义另外两个实验，adversory给challenger两个messages，$m_0$和$m_1$实验0对$m_0$进行加密，实验1对$m_1$进行加密，adversory则进行猜测，判断密文对应的明文是哪个，并输出0或1。
$$
W_0=Pr[b'=1 | b=0]\\
W_1=Pr[b'=1 | b=1]
$$
我们可以定义semantic security advantage为
$$
SSAdv[A,\pi_{se}] = |W_0-W_1|
$$

定义： 一个cipher $\pi_{se} = (Encrypt, Decrypt)$为semantically secure如果对于所有的efficient adversaries A来说， $SSAdv[A,\pi_{se}] = negl({\lambda})$

定理：如果一个cipher满足perfect secrecy，那么它也是semantically secure。

定理：G是一个secure PRG，那么G对应的sream cipher是semantically secure。

证明：考虑semantic secruity的两个实验

Experiment 0: Adversary选择$m_0$和$m_1$，并接受$c_0=G(s){\oplus}m_0$

Experiment 0: Adversary选择$m_0$和$m_1$，并接受$c_0=G(s){\oplus}m_1$

则：
$$
W_0=Pr[adversary\;outputs\;1\;in\;Experiment\;0]\\
W_1=Pr[adversary\;outputs\;1\;in\;Experiment\;1]
$$

如果G(s)是完全随机的string(one-time pad)那么$W_0=W_1$。但G(s)只能生成近似otp。定义：

Experiment 0‘’: Adversary选择$m_0$和$m_1$，并接受$c_0=t{\oplus}m_0$

Experiment 0: Adversary选择$m_0$和$m_1$，并接受$c_0=t{\oplus}m_1$

同样有对应的$W_0', W_1'$

我们可以得到$W_0'=W_1'$(因为otp是perfectly secure的)。根据secure PRG的定义，我们假设$|W_0-W_0'|=negl$和$|W_1-W_1'|=negl$

=>$|W_0-W_1| = |W_0-W_0'+W_0'-W_1'+W1'-W1| {\leq} |W_0-W0'| + |W0'-W1'| + |W1'-W1|
= negl + negl = negl$

那么只需证明G是secure PRG的情况下，对于所有的efficient adversory, $|W_0-W_0'|=negl$

这里我们可以利用reduction的方法进行证明，首先构造逆否命题，如果A可以分辨实验0和实验0‘，那么G不是一个secure PRG。

我们假设存在A可以分辨实验0和实验0'，那么我们可以构造B来break G的security。

我们只需在B中装入A，challenger和在定义secure PRG的实验中相同，我们将challenger生成的t与m进行异或交给B中的A，将A生成的结果输出就能打破PRG的security。可以参考这张图：
![reduction](Cryptography笔记/reduction.png)