---
title: Hexo博客维护
date: 2022-07-07 14:10:52
categories:
- 博客维护
tags:
---

## Hexo博客维护

1、下载安装nodejs

https://nodejs.org/en/

2、切换npm为阿里源

```shell
 npm config set registry https://registry.npm.taobao.org
```

3、使用npm安装hexo

```shell
npm install -g hexo-cli
```

>否则会报错:
>
>$ hexo clean
>ERROR Cannot find module 'hexo' from 'C:\Users\fr724\Desktop\新建文件夹\gsproj.github.io'
>ERROR Local hexo loading failed in ~\Desktop\新建文件夹\gsproj.github.io
>ERROR Try running: 'rm -rf node_modules && npm install --force'

4、使用npm安装搜索功能依赖

```shell
npm install hexo-generator-searchdb --save
```

5、内容上线

```shell
hexo clean
hexo g
hexo d
```



