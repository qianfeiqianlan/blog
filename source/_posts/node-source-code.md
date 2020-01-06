---
title: Node.js源码分析
categories: 
- Node.js
tags:
- Node.js
- 源码分析
---

## 缘起

去年年底换新项目，项目方向虽然偏业务计算，但是前后端框架使用的是react + express的node套餐，因为之前从来没有仔细了解过node比较细节部分的技术，上了项目之后花了一些时间看了看Node.js部分源代码，2020年伊始，也算是给自己立个flag吧，尝试着把自己学到的东西分享出来，输出一系列的文章，记录下自己学到的东西，也希望帮到更多的人。

**同时也因为不管是网上还是书上，有一些源码分析类文章是有一些过时的，该系列文章基于v13.3.0版本源代码，从main方法开始，逐步分析Node.js的架构、模块系统、V8、libuv、及一些内置模块等等，flag先立下来，可以等年底的时候看一下有没有完成。**

## 前言

node生于2009年，天才少年Ryan Dahl利用了Google的V8引擎打造了基于事件循环的异步I/O框架。也许Ryan当时选择JavaScript作为服务端开发语言，只是因为V8的性能远超他脚本语言，但是这却成为node成功的极其重要的因素。不仅仅是JavaScript巨大的用户群，更重要的是JavaScript之前没有任何I/O库，这使Node在开发异步I/O时不会像EventMachine、Twisted那样因与同步I/O混用而导致的问题  
<div style="text-align: right"> —— 《深入浅出NodeJS》序 II </div>

Node.js的第一个版本发布于2009年，迄今已经十余年之久，得益于Web领域蓬勃的发展，Node.js也展现了勃勃的生命力，周边工具不断完善，node的版本也来到了13.0，下面我们就来揭开node神秘的面纱，尝试走进node源码中去看一下，ndoe到底为我们做了哪些事情。

## 架构

下面是一张node的architecture图

![](/images/node-source-code/node-architecture.jpeg)

通过这张图可以很清楚的看到，node通过C/C++封装系统相关的功能，包括Event Loop、DNS、http、zlib等等一系列功能，然后通过node自己的binding系统，将C/C++所提供的能力赋予系统库(也就是`Node standard library`)，也就是我们代码里可以调用部分。