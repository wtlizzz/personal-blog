title: Vue入坑指南
author: Wtli
date: 2020-08-12 14:02:07
tags:
---
解决进入Vue的某些坑～～～
<!--more-->
#### margin-8px
浏览器自动设置body的margin。在写页面的时候，在App.vue文件style中添加
```
body {
  margin: 0;
}

```
#### 地址栏中去掉#
为了让地址栏更加好看，在router下的index.js文件中添加mode。
```
export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'TabNetMonitor',
      component: TabNetMonitor,
    },
  ],
});
```
#### @意义
在引入文件中，为了能够更加方便的引入，可以使用@标识符，@所代表的目录在webpack.base.conf.js文件中有设置。@路径代表项目中的src文件夹。
```
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
    }
```