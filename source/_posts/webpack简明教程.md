---
title: webpack简明教程
date: 2022-08-04 00:17:39

tags:
- webpack
- 打包工具

categories:
- javascript

---

# 介绍

- `webpack`是一种静态资源打包工具
- 以一个文件或多个文件作为打包的入口，将整个项目所有文件编译组合成一个或多个文件输出
    - 输出文件叫做 `bundle`
- 本身功能有限，需要额外的加载器和插件增强功能
    - 开发模式 Development：仅能编译 JS 中的 `ES Module` 语法
    - 生产模式 Production：能编译 JS 中的 `ES Module` 语法，还能压缩 JS 代码

<!-- more -->

# 前置环境

# 起步

1. 初始化

    - 来到项目根目录

    - `npm init -y`

        - 生成`package.json`

        -
      ```javascript
        {
            // name != webpack    
          "name": "webpack_template",
          "version": "1.0.0",
          "description": "",
          "main": "index.js",
              // 执行脚本 npm run xxx
          "scripts": {
            "test": "echo \"Error: no test specified\" && exit 1"
          },
          "keywords": [],
          "author": "",
          "license": "ISC"
        }
        ```

2. 构建文件

    1.
   ```
      - /
       - src
        - js
         - helloworld.js
        main.js
      ```

    2.
   ```javascript
      // helloworld.js
      export default function hello() {
          console.log("hello world")
      }
   ```

    3.
    ```javascript
    // main.js
    import hello from "./js/helloworld";
    hello()
    ```

3. 下载依赖 `npm i webpack webpack-cli - D`

    1. 生产环境使用

4. 启用 `npx webpack ./src/main.js --mode=development`

    1. `npx webpack` 运行webpack包
    2. `./src/main.js` 指定入口文件，从此文件开始打包
    3. `--mode=development` 指定打包模式
        1. development
        2. production 代码压缩，取消注释

5.
```shell
- /
 - dist
  - main.js #打包结果
```
    1.
    ```javascript
    (()=>{"use strict";console.log("hello world")})();
    ```

# 配置文件

## 5大核心概念

1. entry
2. output
3. loader
4. plugins
5. mode
    1. development
    2. production

## 处理资源

[Webpack 官方 Loader 文档](https://webpack.docschina.org/loaders/)

### 修改输出资源名称及路径

```javascript
output: {
    path: path.resolve(__dirname, "dist"),
        // 将 js 文件输出到 static/js 目录中,
    filename: "static/js/main.js", 
    clean: true, // 自动将上次打包目录资源清空
  }

module: {
    rules: [
        {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
          },
        },
        generator: {
          // 将图片文件输出到 static/imgs 目录中
          // 将图片文件命名 [hash:8][ext][query]
          // [hash:8]: hash值取8位
          // [ext]: 使用之前的文件扩展名
          // [query]: 添加之前的query参数
          filename: "static/imgs/[hash:8][ext][query]",
        }, 
        }
    ]
}
```



### html

1. `npm i html-webpack-plugin -D`

2.
```javascript
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "src"),
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "public/index.html"),
    }),
  ]
```



### CSS

- CSS
    - `npm i css-loader style-loader -D`
        - `css-loader`：负责将 CSS 文件编译成 Webpack 能识别的模块
        - `style-loader`：会动态创建一个 Style 标签，里面放置 Webpack 中 CSS 模块内容
- Less
    - `npm i less-loader -D`
        - 负责将 Less 文件编译成 CSS 文件
- Sass and Scss
    - `npm i sass-loader sass -D`
        - `sass-loader`：负责将 Sass 文件编译成 CSS 文件
        - `sass`：`sass-loader` 依赖 `sass` 进行编译
- Styl
    - `npm i stylus-loader -D`
        - `stylus-loader`：负责将 Styl 文件编译成 CSS 文件

##### 提取CSS成单独的文件

1. `npm i mini-css-extract-plugin -D`

2. ```javascript
   const MiniCssExtractPlugin = require("mini-css-extract-plugin");
   
        {
           // 用来匹配 .css 结尾的文件
           test: /\.css$/,
           // use 数组里面 Loader 执行顺序是从右到左
           use: [MiniCssExtractPlugin.loader, "css-loader"],
         },
             
   plugins: [
       // 提取css成单独文件
       new MiniCssExtractPlugin({
         // 定义输出文件名和目录
         filename: "static/css/main.css",
       }),
     ],
   ```

##### CSS兼容性处理

1. `npm i postcss-loader postcss postcss-preset-env -D`

2. ```javascript
        {
           // 用来匹配 .css 结尾的文件
           test: /\.css$/,
           // use 数组里面 Loader 执行顺序是从右到左
           use: [
             MiniCssExtractPlugin.loader,
             "css-loader",
             {
               loader: "postcss-loader",
               options: {
                 postcssOptions: {
                   plugins: [
                     "postcss-preset-env", // 能解决大多数样式兼容性问题
                   ],
                 },
               },
             },
           ],
         },
   ```

3. 控制兼容性
      ```javascript
      // package.json
      {
        // 交集
        "browserslist": ["last 2 version", "> 1%", "not dead"]
      }
      ```

##### 代码复用

```javascript
    // 获取处理样式的Loaders
    const getStyleLoaders = (preProcessor) => {
      return [
        MiniCssExtractPlugin.loader,
        "css-loader",
        {
          loader: "postcss-loader",
          options: {
            postcssOptions: {
              plugins: [
                "postcss-preset-env", // 能解决大多数样式兼容性问题
              ],
            },
          },
        },
        preProcessor,
      ].filter(Boolean);
    };
```



##### CSS压缩

1. `npm i css-minimizer-webpack-plugin -D`

2.
```javascript
plugins: [
    // css压缩
    new CssMinimizerPlugin(),
  ],
```

### js

#### Eslint

1. 可组装的 JavaScript 和 JSX 语法检查工具

2. 配置文件

    1. `.eslintrc.*`：新建文件，位于项目根目录
        - `.eslintrc`
        - `.eslintrc.js`
        - `.eslintrc.json`
        - 区别在于配置格式不一样
    2. `package.json` 中 `eslintConfig`：不需要创建文件，在原有文件基础上写
    3. ESLint 会查找和自动读取它们，所以以上配置文件只需要存在一个即可

3.
```javascript
// .eslintrc.js
module.exports = {
  // 解析选项
  parserOptions: {},
  // 具体检查规则
  rules: {},
  // 继承其他规则
  extends: [],
  // ...
  // 其他规则详见：https://eslint.bootcss.com/docs/user-guide/configuring
};
```

    1.
    ```javascript
    parserOptions: {
      ecmaVersion: 6, // ES 语法版本
      sourceType: "module", // ES 模块化
      ecmaFeatures: { // ES 其他特性
        jsx: true // 如果是 React 项目，就需要开启 jsx 语法
      }
    }
    ```

    2.
    ```javascript
    rules: {
      // "off", "warn", "error"
      // "error" (当被触发的时候，程序会退出)
      semi: "error", // 禁止使用分号
      'array-callback-return': 'warn', // 强制数组方法的回调函数中有 return 语句，否则警告
      'default-case': [
        'warn', // 要求 switch 语句中有 default 分支，否则警告
        { commentPattern: '^no default$' } // 允许在最后注释 no default, 就不会有警告了
      ],
      eqeqeq: [
        'warn', // 强制使用 === 和 !==，否则警告
        'smart' // https://eslint.bootcss.com/docs/rules/eqeqeq#smart 除了少数情况下不会有警告
      ],
    }
    ```

    3.
    ```javascript
    - [Eslint 官方的规则](https://eslint.bootcss.com/docs/rules/)：`eslint:recommended`
    - [Vue Cli 官方的规则](https://github.com/vuejs/vue-cli/tree/dev/packages/@vue/cli-plugin-eslint)：`plugin:vue/essential`
    - [React Cli 官方的规则](https://github.com/facebook/create-react-app/tree/main/packages/eslint-config-react-app)：`react-app
    
    module.exports = {
      extends: ["eslint:recommended"],
      rules: {
        // 我们的规则会覆盖掉extends的规则
        eqeqeq: ["warn", "smart"],
      },
    };
    ```

    4. `npm i eslint-webpack-plugin eslint -D`

    5.
    ```javascript
    // .eslintrc.js
    module.exports = {
      // 继承 Eslint 规则
      extends: ["eslint:recommended"],
      env: {
        node: true, // 启用node中全局变量
        browser: true, // 启用浏览器中全局变量
      },
      parserOptions: {
        ecmaVersion: 6,
        sourceType: "module",
      },
      rules: {
        "no-var": 2, // 不能使用 var 定义变量
      },
    };
    ```

    6.
     ```javascript
    // webpack.config.js
    const ESLintWebpackPlugin = require("eslint-webpack-plugin");
     plugins: [
        new ESLintWebpackPlugin({
          // 指定检查文件的根目录
          context: path.resolve(__dirname, "src"),
        }),
      ],
    ```



#### Babel

JavaScript 编译器。

主要用于将 ES6 语法编写的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。



1. 配置文件

    1. `babel.config.*`：新建文件，位于项目根目录
        - `babel.config.js`
        - `babel.config.json`
    2. `.babelrc.*`：新建文件，位于项目根目录
        - `.babelrc`
        - `.babelrc.js`
        - `.babelrc.json`
    3. `package.json` 中 `babel`：不需要创建文件，在原有文件基础上写

2.
```javascript
// babel.config.js
module.exports = {
  // 预设
  presets: [],
};
```

    1. 简单理解：就是一组 Babel 插件, 扩展 Babel 功能

        - `@babel/preset-env`: 一个智能预设，允许您使用最新的 JavaScript。
        - `@babel/preset-react`：一个用来编译 React jsx 语法的预设
        - `@babel/preset-typescript`：一个用来编译 TypeScript 语法的预设

    2. `npm i babel-loader @babel/core @babel/preset-env -D`

    3.
    ```javascript
    // babel.config.js
    module.exports = {
      presets: ["@babel/preset-env"],
    };
    ```

    4.
    ```javascript
    // webpack.config.js
        {
            test: /\.js$/,
            exclude: /node_modules/, // 排除node_modules代码不编译
            loader: "babel-loader",
          },
    ```



### others

webpack 5 内置`file-loader` 和 `url-loader`

将小于某个大小的图片转化成 data URI 形式（Base64 格式）

减少请求数量，但图片资源体积变大

```javascript
	{
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024 // 小于10kb的图片会被base64处理
          }
        }
      },
```



```javascript
      {
        test: /\.(ttf|woff2?|map4|map3|avi)$/,
        type: "asset/resource",
        generator: {
          filename: "static/media/[hash:8][ext][query]",
        },
      },
```

1. `type: "asset/resource"` 相当于`file-loader`, 将文件转化成 Webpack 能识别的资源，其他不做处理
2. `type: "asset"` 相当于`url-loader`, 将文件转化成 Webpack 能识别的资源，同时小于某个大小的资源会处理成 data URI 形式





### 开发服务器

1. `npm i webpack-dev-server -D`

2.
```javascript
devServer: {
    host: "localhost", // 启动服务器域名
    port: "3000", // 启动服务器端口号
    open: true, // 是否自动打开浏览器
  },
```

4. `npx webpack serve`
5. 代码只会在内存中编译打包，不会输出到dist目录下



## 代码优化

### 文件准备

```
├── webpack-template (项目根目录)
    ├── config (Webpack配置文件目录)
    │    ├── webpack.dev.js(开发模式配置文件)
    │    └── webpack.prod.js(生产模式配置文件)
    ├── node_modules (下载包存放目录)
    ├── src (项目源码目录，除了html其他都在src里面)
    │    └── 略
    ├── public (项目html文件)
    │    └── index.html
    ├── .eslintrc.js(Eslint配置文件)
    ├── babel.config.js(Babel配置文件)
    └── package.json (包的依赖管理配置文件)
```

```javascript
// package.json
{
  // 其他省略
  "scripts": {
    "start": "npm run dev",
    "dev": "npx webpack serve --config ./config/webpack.dev.js",
    "build": "npx webpack --config ./config/webpack.prod.js"
  }
}
```



#### webpack.dev.js



#### webpack.prod.js



### 提升开发体验

#### SourceMap

[Webpack DevTool 文档](https://webpack.docschina.org/configuration/devtool/)

SourceMap（源代码映射）是一个用来生成源代码与构建后代码一一映射的文件的方案。

它会生成一个 xxx.map 文件，里面包含源代码和构建后代码每一行、每一列的映射关系。当构建后代码出错了，会通过 xxx.map 文件，从构建后代码出错位置找到映射后源代码出错位置，从而让浏览器提示源代码文件出错位置，帮助我们更快的找到错误根源。

1. 开发模式：`cheap-module-source-map`

    1. 优点：打包编译速度快，只包含行映射

    2. 缺点：没有列映射

    3.
   ```javascript
      // webpack.dev.js 
      devtool: "cheap-module-source-map",
      ```

2. 生产模式：`source-map`

    1. 优点：包含行/列映射

    2. 缺点：打包编译速度更慢

    3. ```javascript
      // webpack.prod.js
      devtool: "source-map",
      ```



### 提升打包构建速度

#### HotModuleReplacement

HotModuleReplacement（HMR/热模块替换）：在程序运行中，替换、添加或删除模块，而无需重新加载整个页面。

```javascript
// webpack.dev.js
devServer: {
    host: "localhost", // 启动服务器域名
    port: "3000", // 启动服务器端口号
    open: true, // 是否自动打开浏览器
    hot: true, // 开启HMR功能（只能用于开发环境，生产环境不需要了）
  },
```

js 处理判断是否支持，支持用模块替换

```javascript
// 判断是否支持HMR功能
if (module.hot) {
  module.hot.accept("./js/example.js", function (example) {
    const result = example(2, 1);
    console.log(result);
  });
```

要跟 [vue-loader](https://github.com/vuejs/vue-loader), [react-hot-loader](https://github.com/gaearon/react-hot-loader) 结合使用，自动执行



#### One of

打包时每个文件都会经过所有 loader 处理，虽然因为 `test` 正则原因实际没有处理上，但是都要过一遍。比较慢。

顾名思义就是只能匹配上一个 loader, 剩下的就不匹配了。

```javascript
rules: [
      {
        oneOf: [
            ......
        ]
      }
    ]
```



#### Include/Exclude

开发时我们需要使用第三方的库或插件，所有文件都下载到 node_modules 中了。而这些文件是不需要编译可以直接使用的。

所以我们在对 js 文件处理时，要排除 node_modules 下面的文件。

```javascript
{
       test: /\.js$/,
       // 排除node_modules代码不编译
       // exclude: /node_modules/, 
       // 两个只能同时用一个
       include: path.resolve(__dirname, "../src"), // 也可以用包含
       loader: "babel-loader",
 },
```



#### Cache

每次打包时 js 文件都要经过 Eslint 检查 和 Babel 编译，速度比较慢。

我们可以缓存之前的 Eslint 检查 和 Babel 编译结果，这样第二次打包时速度就会更快了。

对 Eslint 检查 和 Babel 编译结果进行缓存。

```javascript
{
       test: /\.js$/,
       // 排除node_modules代码不编译
       // exclude: /node_modules/, 
       // 两个只能同时用一个
       include: path.resolve(__dirname, "../src"), // 也可以用包含
       loader: "babel-loader",
       options: {
               cacheDirectory: true, // 开启babel编译缓存
                   cacheCompression: false, // 缓存文件不要压缩
       },
 },
     
 plugins: [
        new ESLintWebpackPlugin({
            // 指定检查文件的根目录
            context: path.resolve(__dirname, "../src"),
            cache: true, // 开启缓存
            // 缓存目录
            cacheLocation: path.resolve(__dirname, "../node_modules/.cache/.eslintcache")
        }),
     ]
```

#### Thread



当项目越来越庞大时，打包速度越来越慢，甚至于需要一个下午才能打包出来代码。这个速度是比较慢的。

我们想要继续提升打包速度，其实就是要提升 js 的打包速度，因为其他文件都比较少。

而对 js 文件处理主要就是 eslint 、babel、Terser 三个工具，所以我们要提升它们的运行速度。

我们可以开启多进程同时处理 js 文件，这样速度就比之前的单进程打包更快了。

多进程打包：开启电脑的多个进程同时干一件事，速度更快。

**需要注意：请仅在特别耗时的操作中使用，因为每个进程启动就有大约为 600ms 左右开销。**

1. `npm i thread-loader -D`

2.
```javascript
// webpack.config.js
const os = require("os");
const TerserPlugin = require("terser-webpack-plugin");

// cpu核数
const threads = os.cpus().length;

{
            test: /\.js$/,
            include: path.resolve(__dirname, "../src"), 
            use: [
              {
                loader: "thread-loader", // 开启多进程
                options: {
                  workers: threads, // 数量
                },
              },
              {
                loader: "babel-loader",
                options: {
                  cacheDirectory: true, // 开启babel编译缓存
                },
              },
            ],
          },
// 插件
    plugins: [
        new ESLintWebpackPlugin({
            // 指定检查文件的根目录
            context: path.resolve(__dirname, "../src"),
            cache: true, // 开启缓存
            // 缓存目录
            cacheLocation: path.resolve(__dirname, "../node_modules/.cache/.eslintcache"),
            threads, // 开启多进程
        }),
        new TerserPlugin({
            parallel: threads,
        })
    ],
```



### 减少代码体积

#### Tree Shaking

开发时我们定义了一些工具函数库，或者引用第三方工具函数库或组件库。

如果没有特殊处理的话我们打包时会引入整个库，但是实际上可能我们可能只用上极小部分的功能。

这样将整个库都打包进来，体积就太大了

`Tree Shaking` 是一个术语，通常用于描述移除 JavaScript 中的没有使用上的代码。

**注意：它依赖 `ES Module`。**

Webpack 已经默认开启了这个功能，无需其他配置。



#### Babel 辅助代码

Babel 为编译的每个文件都插入了辅助代码，使代码体积过大！

Babel 对一些公共方法使用了非常小的辅助代码，比如 `_extend`。默认情况下会被添加到每一个需要它的文件中。

你可以将这些辅助代码作为一个独立模块，来避免重复引入。

`@babel/plugin-transform-runtime`: 禁用了 Babel 自动对每个文件的 runtime 注入，而是引入 `@babel/plugin-transform-runtime` 并且使所有辅助代码从这里引用。

1. `npm i @babel/plugin-transform-runtime -D`

2.
```javascript
{
                        test: /\.js$/,
                        // 排除node_modules代码不编译
                        // exclude: /node_modules/,
                        // 两个只能同时用一个
                        include: path.resolve(__dirname, "../src"), // 也可以用包含
                        use: [
                            {
                                loader: "thread-loader", // 开启多进程
                                options: {
                                    workers: threads, // 数量
                                },
                            },
                            {
                                loader: "babel-loader",
                                options: {
                                    cacheDirectory: true, // 开启babel编译缓存
                                    cacheCompression: false, // 缓存文件不要压缩
                                    plugins: [
                                        "@babel/plugin-transform-runtime" // 减少代码体积
                                    ]
                                },
                            }
                        ]
                    },
```



#### Image Minimizer

开发如果项目中引用了较多图片，那么图片体积会比较大，将来请求速度比较慢。

我们可以对图片进行压缩，减少图片体积。

**注意：如果项目中图片都是在线链接，那么就不需要了。本地项目静态图片才需要进行压缩。**

1. `npm i image-minimizer-webpack-plugin imagemin -D`

    1. 无损 `npm install imagemin-gifsicle imagemin-jpegtran imagemin-optipng imagemin-svgo -D`
    2. 有损 `npm install imagemin-gifsicle imagemin-mozjpeg imagemin-pngquant imagemin-svgo -D`

2. 无损

    1.
    ```javascript
    // webpack.config.js
    const ImageMinimizerPlugin = require("image-minimizer-webpack-plugin");
    
    // 压缩图片
          new ImageMinimizerPlugin({
            minimizer: {
              implementation: ImageMinimizerPlugin.imageminGenerate,
              options: {
                plugins: [
                  ["gifsicle", { interlaced: true }],
                  ["jpegtran", { progressive: true }],
                  ["optipng", { optimizationLevel: 5 }],
                  [
                    "svgo",
                    {
                      plugins: [
                        "preset-default",
                        "prefixIds",
                        {
                          name: "sortAttrs",
                          params: {
                            xmlnsOrder: "alphabetical",
                          },
                        },
                      ],
                    },
                  ],
                ],
              },
            },
          }),
    ```



打包时会出现报错：

```
Error: Error with 'src\images\1.jpeg': '"C:\Users\86176\Desktop\webpack\webpack_code\node_modules\jpegtran-bin\vendor\jpegtran.exe"'
Error with 'src\images\3.gif': spawn C:\Users\86176\Desktop\webpack\webpack_code\node_modules\optipng-bin\vendor\optipng.exe ENOENT
```

我们需要安装两个文件到 node_modules 中才能解决, 文件可以从课件中找到：

- jpegtran.exe

需要复制到 `node_modules\jpegtran-bin\vendor` 下面

> [jpegtran 官网地址](http://jpegclub.org/jpegtran/)

- optipng.exe

需要复制到 `node_modules\optipng-bin\vendor` 下面

> [OptiPNG 官网地址](http://optipng.sourceforge.net/)



### 优化代码运行性能

####  Code Split

打包代码时会将所有 js 文件打包到一个文件中，体积太大了。我们如果只要渲染首页，就应该只加载首页的 js 文件，其他文件不应该加载。

所以我们需要将打包生成的文件进行代码分割，生成多个 js 文件，渲染哪个页面就只加载某个 js 文件，这样加载的资源就少，速度就更快。

代码分割（Code Split）主要做了两件事：

1.  分割文件：将打包生成的文件进行分割，生成多个 js 文件。
2.  按需加载：需要哪个文件就加载哪个文件。



##### 多入口

`npm i webpack webpack-cli html-webpack-plugin -D`

```javascript
// webpack.config.js

// 多入口
  entry: {
    main: "./src/main.js",
    app: "./src/app.js",
  },
  output: {
    path: path.resolve(__dirname, "./dist"),
    // [name]是webpack命名规则，使用chunk的name作为输出的文件名。
    // 什么是chunk？打包的资源就是chunk，输出出去叫bundle。
    // chunk的name是啥呢？ 比如： entry中xxx: "./src/xxx.js", name就是xxx。注意是前面的xxx，和文件名无关。
    // 为什么需要这样命名呢？如果还是之前写法main.js，那么打包生成两个js文件都会叫做main.js会发生覆盖。(实际上会直接报错的)
    filename: "js/[name].js",
    clear: true,
  }

```



##### 提取重复代码

```javascript
optimization: {
        // 代码分割配置
        splitChunks: {
            chunks: "all", // 对所有模块都进行分割
            // 以下是默认值
            // minSize: 20000, // 分割代码最小的大小
            // minRemainingSize: 0, // 类似于minSize，最后确保提取的文件大小不能为0
            // minChunks: 1, // 至少被引用的次数，满足条件才会代码分割
            // maxAsyncRequests: 30, // 按需加载时并行加载的文件的最大数量
            // maxInitialRequests: 30, // 入口js文件最大并行请求数量
            // enforceSizeThreshold: 50000, // 超过50kb一定会单独打包（此时会忽略minRemainingSize、maxAsyncRequests、maxInitialRequests）
            // cacheGroups: { // 组，哪些模块要打包到一个组
            //   defaultVendors: { // 组名
            //     test: /[\\/]node_modules[\\/]/, // 需要打包到一起的模块
            //     priority: -10, // 权重（越大越高）
            //     reuseExistingChunk: true, // 如果当前 chunk 包含已从主 bundle 中拆分出的模块，则它将被重用，而不是生成新的模块
            //   },
            //   default: { // 其他没有写的配置会使用上面的默认值
            //     minChunks: 2, // 这里的minChunks权重更大
            //     priority: -20,
            //     reuseExistingChunk: true,
            //   },
            // },
            // 修改配置
            cacheGroups: {
                // 组，哪些模块要打包到一个组
                // defaultVendors: { // 组名
                //   test: /[\\/]node_modules[\\/]/, // 需要打包到一起的模块
                //   priority: -10, // 权重（越大越高）
                //   reuseExistingChunk: true, // 如果当前 chunk 包含已从主 bundle 中拆分出的模块，则它将被重用，而不是生成新的模块
                // },
                default: {
                    // 其他没有写的配置会使用上面的默认值
                    minSize: 0, // 我们定义的文件体积太小了，所以要改打包的最小文件体积
                    minChunks: 2,
                    priority: -20,
                    reuseExistingChunk: true,
                },
            },
        },
```













# webpack.config.js

```javascript
// Node.js的核心模块，专门用来处理文件路径
// 相当于 python os
const path = require("path");

module.exports = {
  // 入口
  // 相对路径和绝对路径都行
  entry: "./src/main.js",
  // 输出
  output: {
    // path: 文件输出目录，必须是绝对路径
    // path.resolve()方法返回一个绝对路径
    // __dirname 当前文件的文件夹绝对路径
    path: path.resolve(__dirname, "dist"),
    // filename: 输出文件名
    filename: "main.js",
  },
  // 加载器
  module: {
    rules: [
        {
            // 文件匹配规则
            test: /\.css$/,
            // use 处理顺序 从右到左，从左到右
            use: [
                "style-loader",
                "css-loader"
            ]
        }
    ],
  },
  // 插件
  plugins: [],
  // 模式
  mode: "",
};
```
