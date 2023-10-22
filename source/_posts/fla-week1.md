---
title: FL&A 笔记一
date: 2023-10-19 10:30:49
tags: 
    - FL&A
    - note
categories: 
    - 【学习笔记】计算机相关
mathjax: true
---

## 介绍

***FL&A*** ，全称为 *Formal Language and Automata* ，中文译名为 *形式语言与自动机* ，本系列文章源于所学必修课，在整理知识点基础上对经典的解题思路进行归纳总结

## 概念介绍

- *字母表 (Alphabet)*

    形式符号的 **非空有限** 集合，常用 $\Sigma$ 表示

    *eg.* 英文小写字母表 {a, b, c, d, ..., z}

- *字符串* or *串* 、*字 (word)*

    定义需要 **基于字母表** ，空串使用 $\epsilon$ 表示

    字符串 $w$ 的长度记为 $\left|w\right|$ ，表示字符串中字符的数量

    *eg.* 设 $\Sigma=\{a, b\}$ ，则 $\epsilon, a, ab$ 等都是串
    
- *相关运算*
        
    - *连接 (connection)*

        设 $x$、$y$ 为串，且 $x=a_{1}a_{2}\cdots a_{n}$ ，$y=b_{1}b_{2}\cdots b_{m}$ ，则 $x$ 与 $y$ 的连接为 $xy=a_{1}a_{2}\cdots a_{n}b_{1}b_{2}\cdots b_{m}$

        字符串的连接满足 **结合律**，且 $\left|xy\right|=\left|x\right|+\left|y\right|$

    - *幂运算*

        设 $\Sigma$ 为字母表，$n$ 为任意自然数，定义

        1. $\Sigma^{0}=\{\epsilon\}$
        2. 设 $x\in\Sigma^{n-1}, a\in\Sigma$ ，则 $ax\in\Sigma^{n}$
        3. $\Sigma^{n}$ 中的元素只能有以上两种方式生成
   
    - *$*$闭包*

        定义 $\Sigma^{*}=\Sigma^{0}\cup\Sigma^{1}\cup\Sigma^{2}\cup\cdots$

    - *+闭包*

        定义 $\Sigma^{+}=\Sigma^{1}\cup\Sigma^{2}\cup\Sigma^{3}\cup\cdots$

- *语言 (language)*

    设 $\Sigma$ 为字母表，则 **任何** 集合 $L\subset\Sigma^{*}$ 是字母表 $\Sigma$ 上的一个语言

    - *连接 (concatenation)*

        定义 $LM=\{w_{1}w_{2}|w_{1}\in L\land w_{2}\in M\}$

    - *闭包 (closure)*

        定义 $L^{*}=L^{0}\cup L^{1}\cup L^{2}\cup\cdots$ ，其中 $L^{0}=\{\epsilon\}$ 且 $L^{n}=L^{n-1}L $

        