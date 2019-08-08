---
layout: post
title:  "Welcome to sona!"
date:   2017-11-14 16:52:07
categories: sona update
tags: sona update
---
##  支付路由遇到的坑

最近在做支付路由,系统架构如下 N:1:N关系


                [下游系统]  ️                       [第三方]

                [下游系统]  ️       [支付路由]        [第三方]

                [下游系统]                           [第三方]


下游系统订单设计出现了分歧