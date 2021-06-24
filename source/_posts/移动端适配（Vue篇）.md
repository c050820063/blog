---
title: 移动端适配（Vue篇）
date: 2020-08-07 17:09:57
tags:
---

### vw适配
1. 安装依赖
  ``` js
  npm i postcss-aspect-ratio-mini postcss-px-to-viewport postcss-write-svg postcss-cssnext postcss-viewport-units cssnano --save
  ```
<!-- more -->
2. 在.postcssrc.js中进行配置
  ``` js
  module.exports = {
    "plugins": {
      "postcss-import": {},
      "postcss-url": {},
      "postcss-aspect-ratio-mini": {},
      "postcss-write-svg": {
        utf8: false
      },
      "postcss-cssnext": {},
      "postcss-px-to-viewport": {
        viewportWidth: 750, // 视窗的宽度，对应的是我们设计稿的宽度，一般是750
        viewportHeight: 1334, // 视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置
        unitPrecision: 3, // 指定`px`转换为视窗单位值的小数位数
        viewportUnit: "vw", //指定需要转换成的视窗单位，建议使用vw
        selectorBlackList: ['.ignore', '.hairlines'],// 指定不转换为视窗单位的类，可以自定义，可以无限添加,建议定义一至两个通用的类名
        minPixelValue: 1, // 小于或等于`1px`不转换为视窗单位，你也可以设置为你想要的值
        mediaQuery: false // 允许在媒体查询中转换`px`
      },
      "postcss-viewport-units": {},
      "cssnano": {
        preset: "advanced",
        autoprefixer: false,
        "postcss-zindex": false
      }
    }
  }
  ```

* postcss-cssnext：其实就是cssnext。该插件可以让我们使用CSS未来的特性，其会对这些特性做相关的兼容性处理。有关于cssnext的每个特性的操作文档，可以点击这里浏览。

* cssnano：主要用来压缩和清理CSS代码。在Webpack中，cssnano和css-loader捆绑在一起，所以不需要自己加载它。详细文档可以点击这里获取。在cssnano的配置中，使用了preset: "advanced"，所以我们需要另外安装：

  ``` js
  npm i cssnano-preset-advanced --save-dev
  "cssnano": {
    "autoprefixer": false,
    "postcss-zindex": false
  }
  ```
* 上面的代码把autoprefixer和postcss-zindex禁掉了。前者是有重复调用，后者是一个讨厌的东东。只要启用了这个插件，z-index的值就会重置为1。这是一个天坑，千万记得将postcss-zindex设置为false。

* postcss-px-to-viewport：要用来把px单位转换为vw、vh、vmin或者vmax这样的视窗单位，也是vw适配方案的核心插件之一。在配置中需要配置相关的几个关键参数：

  ``` js
  "postcss-px-to-viewport": {
    viewportWidth: 750,   // 视窗的宽度，对应的是我们设计稿的宽度，一般是750 （如果我们设置的宽度是300px，那么编译之后的宽度为(300/750*100)=40vw,如果频宽实际为375px，那么该元素的宽度为（375*0.4）= 150px）
    viewportHeight: 1334, // 视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置
    unitPrecision: 3,     // 指定`px`转换为视窗单位值的小数位数（很多时候无法整除）
    viewportUnit: "vw",   // 指定需要转换成的视窗单位，建议使用vw
    selectorBlackList: ['.ignore', '.hairlines'],// 指定不转换为视窗单位的类，可以自定义，可以无限添加,建议定义一至两个通用的类名
    minPixelValue: 1,     // 小于或等于`1px`不转换为视窗单位，你也可以设置为你想要的值
    mediaQuery: false     // 允许在媒体查询中转换`px`
  },
  ```
* postcss-aspect-ratio-mini：主要用来处理元素容器宽高比。

* postcss-write-svg：插件主要用来处理移动端1px的解决方案。该插件主要使用的是border-image和background来做1px的相关处理。

[参考](https://www.jianshu.com/p/686b61c4a43d)

### rem
1. 在main中添加代码
  ``` js
  (function () {
    // 在标准 375px 适配下，100px = 1rem;
    var baseFontSize = 100;  
    var baseWidth = 375;

    var set = function () {
      var clientWidth = document.documentElement.clientWidth || window.innerWidth;

      var rem = 100;
      if (clientWidth != baseWidth) {
        rem = Math.floor(clientWidth / baseWidth * baseFontSize);
      }

      document.querySelector('html').style.fontSize = rem + 'px';
    }
    set();

    window.addEventListener('resize', set);
  }());
  ```
2. 修改.postcssrc.js配置
  ``` js
  npm i postcss-pxtorem --sav-dev // 安装pxtorem插件
  "postcss-pxtorem": {
    "rootValue": 100,
    "propList": ['font', 'font-size', 'line-height', 'letter-spacing']
  }
  ```
  * rootValue 基数； 例：rootValue为100 则1px转换为0.01rem
  * proplist就是那些属性需要转换成rem。 倘若默认为整个项目，则值为：propList:["*"]


### lib-flexible

1. 安装插件
  ``` js
  npm i lib-flexible --save // 安装lib-flexible
  ```

2. main.js 引入
  ``` js
  import 'lib-flexible/flexible'
  ```

3. 在 index.html 中添加：移动适配 meta标签
  ``` js
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  // 注意这两个的区别，建议添加下面的meta，反正点击输入框，页面自动缩放
  <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
  ```
  4. 修改.postcssrc.js配置
  ``` js
  npm i postcss-pxtorem --sav-dev // 安装pxtorem插件
  "postcss-pxtorem": {
    "rootValue": 75, //设计稿除以10得到的值
    "propList": ["*"]
  }
  ```
