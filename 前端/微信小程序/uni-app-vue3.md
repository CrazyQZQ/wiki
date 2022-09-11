---
title: 项目搭建
description: uni-app + vue3微信小程序搭建
published: true
date: 2022-09-11T02:07:15.489Z
tags: 
editor: markdown
dateCreated: 2022-09-11T02:07:15.489Z
---

# cli 创建支持 vue3 的 uni-app 项目
1. 创建 Vue3/Vite 工程
```bash
# 创建以 javascript 开发的工程  
npx degit dcloudio/uni-preset-vue#vite my-vue3-project  

# 创建以 typescript 开发的工程  
npx degit dcloudio/uni-preset-vue#vite-ts my-vue3-project  
```
2. 进入工程目录
```bash
cd my-vue3-project 
```
3. 安装依赖
```bash
npm i  或  yarn  
```
4. 运行
```bash
# 运行到 h5   
npm run dev:h5  
# 运行到 app   
npm run dev:app  
# 运行到 微信小程序  
npm run dev:mp-weixin  
```
5. 打包
```bash
# 打包到 h5   
npm run build:h5  
# 打包到 app   
npm run build:app  
# 打包到 微信小程序  
npm run build:mp-weixin  
```