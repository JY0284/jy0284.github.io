---
title: "Gitbook简易配置手册"
date: 2019-09-25T21:11:37+08:00
draft: false
tags: ["tool", "guide", "gitbook"]
categories: ["手册"]
---

# 安装gitbook
## 1. 首先需要安装nodejs
## 2. 使用npm安装gitbook
```shell
npm install gitbook-cli -g
```
官方页面：[npm gitbook](https://www.npmjs.com/package/gitbook)
# 版本问题及解决方案
安装后很有可能在`build`, `init`, `serve`时报错，无法完成指令，问题出在gitbook版本所需的依赖与当前node版本不匹配，安装一个名为`n`的工具包切换node版本可能可以解决。

安装n：
```shell
npm install -g n
```
切换node版本至10：
```shell
n 10
```
官方页面：[npm n](https://www.npmjs.com/package/n)

在我的环境下，切换node版本至10解决了gitbook报错的问题，并可以正常部署。
# 自动生成SUMMARY目录文件
`SUMMARY.md`是gitbook的目录文件，由于有时候文件内容很多且目录结构已经清晰，自行手写`SUMMARY.md`较为繁琐，可使用工具自动根据当前目录结构生成。

工具项目：[imfly / gitbook-summary](https://github.com/imfly/gitbook-summary)

基本用法：
```shell
# 安装
npm install -g gitbook-summary

# 生成SUMMARY.md
book sm
```

支持包括指定排序方法等多种功能，具体可查看上面的项目链接。

# 使用支持中文检索的插件
插件：[gitbook-plugin-search-pro](https://www.npmjs.com/package/gitbook-plugin-search-pro)

## 1. 安装插件
```shell
npm install gitbook-plugin-search-pro -g
```
## 2. 配置book.json的plugin
```json
{
    "title": "资治通鉴",
    "author": "司马光 及诸位译者",
    "description": "《资治通鉴》文言文及白话文对照文本",
    "language": "zh-hans",
    "plugins": [
	    "-lunr",
        "-search",
        "search-pro"
    ]
}
```
注意，"-lunr","-search","search-pro" 需要一起写入，否则可能会引起冲突。

# 配置nginx
使用`git build`指令后，在`gitbook init`目录下会产生`_book`目录，其中为静态站点的全部资源，设置nginx反向解析至此即可。
```
server{
        listen 80;
        server_name $host_address;
        access_log $path-to-log;

        charset utf-8;

        location / {
                root path_to_your_book_directory/_book;
        }
}
```

# 样例
[资治通鉴](http://book.nnzxzb.cn)
