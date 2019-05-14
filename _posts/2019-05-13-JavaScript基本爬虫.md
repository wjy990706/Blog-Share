---
layout:     post
title:      "JavaScript 基础爬虫"
date:       2019-05-12
author:     "李楸柯"
header-img: "img/post-bg-2015.jpg"
tags:
    - 前端
    - JavaScript
    - node.js
---

此处的 JavaScript 指的是 node.js 环境，主要使用 axios 和 cheerio 两个库。

本文的目标是爬取 [it.swufe.edu.cn](https://it.swufe.edu.cn) 上的首页某一栏（如：学院新闻、通知公告）的所有文章的 `{标题, 链接, 日期, 正文}` 数据。

> axios 是一个基于 promise 的 HTTP 库，可以用在浏览器和 node.js 中。
>
> cheerio 是 jquery 核心功能的一个快速灵活而又简洁的实现，主要是为了用在服务器端需要对DOM进行操作的地方

## 环境配置

1. 安装 node.js、npm（或 Yarn ）

   - Windows：在 node.js 官网下载 installer 程序
   - Mac：使用 `brew install node` ，安装 Yarn 可用 `brew install yarn`

   安装完成后使用以下命令查看是否安装成功

   ```bash
   node --version
   
   npm --version
   
   yarn --version
   ```

2. 更换 npm 镜像为淘宝镜像

   ```bash
   npm install -g cnpm --registry=https://registry.npm.taobao.org
   ```

3. 安装 axios 和 cheerio

   使用命令行工具（此处使用了 yarn，也使用 npm 也可）：

   ```bash
   yarn install axios
   
   yarn install cheerio
   ```

## 基础知识

### 1. axios 执行 `get` 请求

```javascript
// 此处是 promise 异步方法
axios.get(url)
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

### 2. cheerio 解析 html 内容

cheerio 语法与 jQuery 几乎相同，会使用 jQuery 就能使用 cheerio

```javascript
$ = cheerio.load('<h2 class="title">Hello world</h2>');

$('h2.title').text('Hello there!');
$('h2').addClass('welcome');

$.html();
//=> <h2 class="title welcome">Hello there!</h2>
```

## 开始使用

```javascript
const axios = require('axios');
const cheerio = require('cheerio');

const listUrl = 'https://it.swufe.edu.cn/index/';
const baseUrl = 'https://it.swufe.edu.cn/';

const map = new Map([['xyxw', { title: '西南财经大学经济信息工程学院 -- 学院新闻', suffix: 'xyxw.htm' }], ['tzgg', { title: '西南财经大学经济信息工程学院 -- 通知公告', suffix: 'tzgg.htm' }]]);

// 此处使用 async/await 的异步方式
async function run(column){
    const type = column || 'tzgg';
    const suffix = map.get(type).suffix;

    const response = await axios.get(link);
    const data = response.data;

    const $ = cheerio.load(data);

    const list = $('div[class="article-list"] ul li a')
        .slice(0, 10)
        .map(function() {
            const info = {
                link: $(this)
                    .attr('href')
                    .slice(2),
                title: $(this)
                    .children('span')
                    .first()
                    .text(),
                date: $(this)
                    .children('span')
                    .last()
                    .text(),
            };
            return info;
        })
        .get();
  
  	const out = await Promise.all(
        list.map(async (info) => {
            const title = info.title;
            const date = info.date;
            const itemUrl = url.resolve(baseUrl, info.link);

            const cache = await ctx.cache.get(itemUrl);
            if (cache) {
                return Promise.resolve(JSON.parse(cache));
            }

            const response = await axios.get(itemUrl);
            const $ = cheerio.load(response.data);

            const single = {
                title: title,
                link: itemUrl,
                description: $('.article-main')
                    .html()
                    .replace(/src="\//g, `src="${url.resolve(baseUrl, '.')}`)
                    .trim(),
                pubDate: new Date(date),
            };
            ctx.cache.set(itemUrl, JSON.stringify(single));
            return Promise.resolve(single);
        })
    );
}
```

## 参考资料

1. [Axios 中文说明](https://www.kancloud.cn/yunye/axios/234845)
2. [cheerio.js.org](https://cheerio.js.org/)

