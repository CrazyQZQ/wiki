---
title: vuex持久化问题
description: 二次开发vue-typescript-admin问题
published: true
date: 2022-07-21T07:13:09.829Z
tags: 
editor: markdown
dateCreated: 2022-07-21T07:13:09.829Z
---

# vuex持久化问题
原项目使用[vuex-module-decorators](),且只有token放在cookie中，其他数据页面刷新后丢失

## 解决方法：使用vuex-persist
1. 安装vuex-persist
```bash
yarn add vuex-persist
```
2. 具体module加上`preserveState: localStorage.getItem('vuex') !== null`
```javascript
@Module({ dynamic: true, store, name: 'user', preserveState: localStorage.getItem('vuex') !== null })
class User extends VuexModule implements IUserState {
  public id= -1
  public token = ''
  public name = ''
  public avatar = ''
  public introduction = ''
  public roles: string[] = []
  public email = ''

  @Mutation
  private SET_ID(id: number) {
    this.id = id
  }
}
```
3. 设置存储方式
```javascript
// 引入
import VuexPersistence from 'vuex-persist'

// 存储方式
const vuexLocal = new VuexPersistence<IRootState>({
  storage: window.localStorage
})

// 设置插件
export default new Vuex.Store<IRootState>({
  plugins: [vuexLocal.plugin]
})
```
