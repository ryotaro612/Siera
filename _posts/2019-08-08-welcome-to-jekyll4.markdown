---
layout: post
title:  "支付路由"
date:   2017-11-14 16:52:07
categories: sona update
tags: sona update
---
##  支付路由遇到的坑

![avatar](/images/logo.png)

最近在做支付路由,系统架构如下 N:1:N关系


                [下游系统]  ️                       [第三方]

                [下游系统]  ️       [支付路由]        [第三方]

                [下游系统]                           [第三方]


下游系统订单设计出现了分歧