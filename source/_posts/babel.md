---
title: babel
date: 2020-06-17 16:18:58
tags:
---

### @babel/core
Babel的核心模块
- 安装
``` js
npm i -D @babel/core
```
<!-- more -->
- 使用

```
const babel = require('@babel/core');
babel.transform("code", options);
```

### @babel/cli
终端运行工具, 内置的插件, 可以在终端使用babel的工具
- 安装
``` js
npm i -D @babel/cli
```

#### 用法一

``` js
./node_modules/.bin/babel src --out-dir lib
```
这段语句的意思是: 它使用我们设置的解析方式来解析src目录下的所有JS文件, 并将转换后的每个文件都输出到lib目录下.

如果是 `npm@5.2.0` 就可以使用：
``` js
npx babel src --out-dir lib
```

#### 用法二
给package.json中配置一段脚本命令:

``` js
{
    "name": "babel-basic",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
+       "build": "babel src -d lib"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
+       "@babel/cli": "^7.8.4",
+       "@babel/core": "^7.8.4"
    }
}
```

现在运行npm run build效果也是一样的, -d是--out-dir的缩写...
(使用上面的 --out-dir 选项。也可以通过使用 --help 运行它来查看 cli 工具接受的其余选项。但对我们来说最重要的是 --plugins 和 --presets。)

``` js
npx babel --help
```

### 插件plugins
- 基本概念
本质就是一个JS程序, 指示着Babel如何对代码进行转换.
如果你是要将ES6+转成ES5, 可以依赖官方插件, 例如:
@babel/plugin-transform-arrow-functions

``` js
cnpm i --save-dev @babel/plugin-transform-arrow-functions
npx babel src --out-dir lib --plugins=@babel/plugin-transform-arrow-functions
```

插件的作用是将箭头函数转换为ES5兼容的函数

### Presets:
- 基本概念
如果想要转换ES6+的其它代码为ES5, 可以使用"preset"来代替预先设定的一组插件, 而不是逐一添加我们想要的所有插件.
`可以理解为一个preset就是一组插件的集合`

- @babel/preset-env
``` js
npm i --save-dev @babel/preset-env
```

envpreset这个preset包括支持现代JavaScript(ES6+)的所有插件.

``` js
npx babel src --out-dir lib --presets=@babel/preset-env
```

### 配置

创建一个babel.config.js文件:
``` js
const presets = [
	[
    "@babel/env",
    {
      targets: {
        edge: "17",
        chrome: "64",
        firefox: "60",
        safari: "11.1"
      }
    }
  ]	
]

module.exports = { presets };
```

加上这个配置的作用是:
- 使用了envpreset这个preset
- envpreset只会为目标浏览器中没有的功能加载转换插件

package.json:

``` js
{
	"scripts": {
		"build": "babel src -d lib"
	}
}
```
执行 npm run build 即可


这个命令行语句看起来并没有修改, 那是因为它默认会去寻找跟根目录下的一个名为babel.config.js的文件(或者babelrc.js也可以), 所以其实就相当于以下这个配置:

``` js
{
	"scripts": {
		"build": "babel src -d lib --config-file ./babel.config.js"
	}
}
```
(--config-file指令就类似于webpack中的--config, 用于指定以哪个配置文件构建)

targets: 只会为目标浏览器中没有的功能加载转换插件

例如我这里配置的其中一项是edge: "17", 那就表示它转换之后的代码支持到edge17.

所以上面babel.config.js的配置之后生成的lib文件夹下的代码好像并没有发生什么改变, 也就是它并没有被转换成ES5的代码

``` 这是因为在Edge17浏览器中支持ES7的这些功能, 所以它就没有必要将其转换了, 它只会为目标浏览器中没有的功能加载转换插件!!! ```

将edge17改成edge10：

babel.config.js:
``` js
const presets = [
    [
        "@babel/env",
        {
            targets: {
-               edge: "17",
+               edge: "10",
                firefox: "60",
                chrome: "67",
                safari: "11.1",
            },
        },
    ],
];

module.exports = { presets };
```
保存重新运行npm run build, 会发现lib/index.js现在有所改变了

### Polyfill

Plugins是提供的插件, 例如箭头函数转普通函数@babel/plugin-transform-arrow-functions

Presets是一组Plugins的集合.

``` 而Polyfill是对执行环境或者其它功能的一个补充.```
就像现在你想在edge10浏览器中使用ES7中的方法includes(), 但是我们知道这个版本的浏览器环境是不支持你使用这个方法的, 所以如果你强行使用并不能达到预期的效果.
而polyfill的作用正是如此, 知道你的环境不允许, 那就帮你引用一个这个环境, 也就是说此时编译后的代码就会变成这样:

``` js
// 原来的代码
var hasTwo = [1, 2, 3].includes(2);

// 加了polyfill之后的代码
require("core-js/modules/es7.array.includes");
require("core-js/modules/es6.string.includes");
var hasTwo = [1, 2, 3].includes(2);
```

现在就让我们来学习一个重要的polyfill, 它就是@babel/polyfill.

@babel/polyfill用来模拟完成ES6+环境:
- 可以使用像Promise或者WeakMap这样的新内置函数
- 可以使用像Array.from或者Object.assign这样的静态方法
- 可以使用像Array.prototype.includes这样的实例方法
- generator函数

为了实现这一点, Polyfill增加了全局范围以及像String这样的原生原型.

而@babel/polyfill模块包括了core-js和自定义regenerator runtime

对于库/工具来说, 如果你不需要像Array.prototype.includes这样的实例方法, 可以使用transform runtime插件, 而不是使用污染全局的@babel/polyfill.

对于应用程序, 我们建议安装使用@babel/polyfill

``` js
cnpm i --save @babel/polyfill
```
(注意 --save 选项而不是 --save-dev，因为这是一个需要在源代码之前运行的 polyfill。)

但是由于我们使用的是envpreset, 这里个配置中有一个叫做 "useBuiltIns"的选项

如果将这个选择设置为"usage", 就只包括你需要的polyfill

此时的babel.config.js调整为:

``` js
const presets = [
	[
		"@babel/env",
		{
			targets: {
				edge: "17",
				chrome: "64",
				firefox: "67",
				safari: '11.1'
			},
+			useBuiltIns: "usage"
		}
	]
]

module.exports = { presets }
```

安装配置了@babel/polyfill, Babel将检查你的所有代码, 然后查找目标环境中缺少的功能, 并引入仅包含所需的polyfill

(如果我们没有将 env preset 的 "useBuiltIns" 选项的设置为 "usage" ，就必须在其他代码之前 require 一次完整的 polyfill。)

### corejs
被deprecated的@babel/polyfill

上面我介绍了一种名为@babel/polyfill的polypill, 其实它在Babel7.4.0以上已经不被推荐使用了.

而是推荐使用core-js@3+@babel/preset-env然后设置@babel/preset-env的corejs选项为3.

因此如果你按着我文章中讲方式使用@babel/polyfill, 是可以实现的, 不过控制台中会抛出一个警告⚠️

解决办法是卸载掉@babel/polyfill, 然后重新安装core-js@版本号, 然后重新配置一些babel.config.js文件.

1. 安装core-js@3:
``` js
cnpm i --save core-js@3
```
2. 添加corejs选项:
``` js
const presets = [
[
  "@babel/env",
      {
        targets: {
        edge: "17",
        chrome: "64",
        firefox: "67",
        safari: '11.1'
      },
      useBuiltIns: "usage",
+     corejs: 3
    }
  ]
]

module.exports = { presets }
```
现在重新npm run build之后就不会有这个警告了, 而且生成的lib也是正确的.

### 小结
- babel/cli 允许我们从终端运行Babel
- envpreset 只包含我们使用的功能的转换,实现我们的目标浏览器中缺少的功能
- @babel/polyfill实现所有新的JS功能, 为目标浏览器引入缺少的环境(但是Babel7.4.0以上推荐使用corejs)

[参考](https://juejin.im/post/5e477139f265da574c566dda)
