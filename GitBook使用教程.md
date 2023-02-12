---
title: GitBook使用教程
date: 2020-12-07 18:36:03
tags:
- gitbook
categories:
---

## 通过npm安装

```shell
npm install gitbook-cli -g
```

## 创建两个文件

README.md

```
# Introduction
说明文档
```

**SUMMARY.md**

```
# Summary

* [Introduction](README.md)
* [section1](section1/README.md)
    * [example1](section1/example1.md)
    * [example2](section1/example2.md)
* [section2](section2/README.md)
    * [example1](section2/example1.md)
    * [example2](section2/example2.md)
```

## 初始化目录文件

```
gitbook init
```

## 预览书籍

```
gitbook serve
```

## 编译文件

```
gitbook build
```

## 使用插件

创建 book.json 文件

```
{
    "title": "GitBook",
    "author" : "echo",
	"output.name": "site",
    "description" : "电子书",
    "language" : "zh-hans",
	"gitbook" : "3.2.3",
	"root": ".",
	"links" : {
        "sidebar" : {
            "Home" : "https://zhx2020.github.io"
        }
    },
    "styles": {
		"website": "styles/website.css",
		"ebook": "styles/ebook.css",
		"pdf": "styles/pdf.css",
		"mobi": "styles/mobi.css",
		"epub": "styles/epub.css"
	},
    "plugins": [
        "-lunr",
		"-search",
		"back-to-top-button",
		"highlight",
		"search-plus",
		"expandable-chapters",
		"chapter-fold",
		"splitter",
		"-prism",
		"github",
		"github-buttons",
		"tbfed-pagefooter",
		"edit-link",
		"copy-code-button",
		"simple-page-toc",
		"anchor-navigation-ex"
    ],
    "pluginsConfig": {
		"theme-default": {
			"showLevel": false
		},
		"prism": {
            "css": [
                "prism-themes/themes/prism-base16-ateliersulphurpool.light.css"
            ]
        },
		"github": {
			"url": "https://github.com/zhx2020"
		},
		"github-buttons": {
            "repo": "zhx2020/gitbook",
            "types": [
                "star",
                "watch",
                "fork"
            ],
            "size": "small"
        },
		"tbfed-pagefooter": {
			"copyright":"Copyright &copy zhx2020.github.io 2020",
			"modify_label": "该文件修订时间：",
			"modify_format": "YYYY-MM-DD HH:mm:ss"
		},
		"edit-link": {
			"base": "https://github.com/zhx2020/gitbook/edit/master",
			"label": "Edit This Page"
		},
		"simple-page-toc": {
            "maxDepth": 3,
            "skipFirstH1": true
        },
		"anchor-navigation-ex": {
            "showLevel": false,
            "showGoTop": false
        }
    }
}
```

安装插件

```
gitbook install
```

## 书籍部署

将编译后的 _book 文件部署到 github ，配置 page 。

## 在线预览

[GitBook电子书](https://zhx2020.github.io/book/)