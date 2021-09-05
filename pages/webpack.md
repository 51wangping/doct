##  webpack

> webpack: 用来为现代 JavaScript 应用提供静态模块打包
> 打包： 将不同类型的资源按照模块进行打包 
> 静态： 打包后产出的静态资源

#### webpack 初体验


```bash
 # 全局安装webpack webpack-cli

 npm i -g webpack webpack-cli

 # 查看版本

 webpack --version
```

#### webpack config 

项目下新建webpack 配置文件 `webpack.config.js` 配置webpack 

``` bash
# webpack.config.js

const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'build.js',
    path: path.resolve(__dirname,'dist')
  },
  mode: 'development' # production
}

```
 - 如果想根据不同环境执行不同的 webpack 进行打包，可以配置多个 webpack config文件 。 
 - 例如: webpack.dev.config.js webpack.prod.config.js

 ```
 # package.json
 
 "scripts" : {
   ...
   "build": "webpack --config webpack.dev.config.js"
 }
 ```

 #### 解析 css 文件
 > 默认 js 代码不能解析 css 文件，所以需要 loader 对 css 文件进行解析

``` js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'build.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/, // 一般写一个正则来匹配需要处理的文件
        use: ['css-loader'] // 需要使用的loader， 顺序从后向前依次执行
      }
    ]
  },
  mode: 'development'
}
```
> css-loader 只是用来对 css 进行解析，但是不能把处理好的 css 进行展示
> 需要使用 style-loader 对 处理好的 css 文件插入到文件中

```
npm i style-loader --save-D
```

``` js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'build.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/, // 一般写一个正则来匹配需要处理的文件
        use: ['style-loader','css-loader'] // 需要使用的loader, 顺序从右往左依次执行
      }
    ]
  },
  mode: 'development'
}
```
> 项目中使用 less 需要先解析为css 
> 所以使用 less-loader 对less 文件解析

 如果项目中使用了 less ， 需要安装less ，less-loader
```
  npm i less less-loader -D
```
配置webpack config

``` js
const path = require('path')

module.exports = {
...
  module: {
    rules: [
      ...
      {
        test: /\.less$/,
        use: ['style-loader', 'css-loader','less-loader']
      }
    ]
  }
...
}
```

#### Browserslistrc 工作流程
> 在不同前端工具之间共享目标浏览器和 Node.js 版本的配置。主要用于: Autoprefixer
Babel
postcss-preset-env
...


安装
``` bash
npm i browserslist
```


配置方式: package.json 或者 .browserslistrc 配置

``` bash
 # .browserslistrc

defaults
not IE 11
maintained node versions
last 2 version # 最新的两个版本
>1%   

```

#### 如何兼容 css js

##### postcss  
> 利用 JavaScript 转换样式的工具

安装 

```bash
npm i postcss-loader -D

npm i autoprefixer -D
```

默认 postcss-loader 是使用 postcss 只是将 css 进行拷贝，不能做兼容处理，需要设置option 才可以

```js
...
rules: [
      {
        test: /\.css$/,
        use: ['style-loader','css-loader', {
          loader: 'postcss-loader',
          options: {
            postcssOptions: {
              plugins: [
                require('autoprefixer')
              ]
            }
          }
        }]
      }
    ]
...
```

###### postcss-preset-env
> 预设 插件的集合
postcss-preset-env 中已经包含 autoprefixer 插件，所以可以不使用 autoprefixer

```js
...
rules: [
      {
        test: /\.css$/,
        use: ['style-loader','css-loader', {
          loader: 'postcss-loader',
          options: {
            postcssOptions: {
              plugins: ['postcss-preset-env']
            }
          }
        }]
      }
    ]
...
```

或者 创建 postcss.config.js 配置

```js
 // postcss.config.js
module.exports = {
  plugins: [
    require('postcss-preset-env')
  ]
}
```
这样在webpack 中就不需要写那么复杂了

```js
...
module: {
    rules: [
      {
        test: /\.css$/, // 一般写一个正则来匹配需要处理的文件
        use: ['style-loader','css-loader','postcss-loader'] // 需要使用的loader， 顺序从后向前依次执行
      },
      {
        test: /\.less$/, // 一般写一个正则来匹配需要处理的文件
        use: ['style-loader', 'css-loader','postcss-loader','less-loader'] // 需要使用的loader， 顺序从后向前依次执行
      }
    ]
  },
...
```

##### importloaders 属性
> 当css 文件中 引用了其他 css 文档。 webpack 匹配 css 按照 loader 顺序进行处理css 文件
> 这时就算遇到了新的css 文件需要 postcss-loader 文件处理也不再返回上一步进行处理
> 这时就需要使用 importloaders 返回 上一步进行处理

```js
rules: [
      {
        test: /\.css$/, // 一般写一个正则来匹配需要处理的文件
        use: ['style-loader',
        {
          loader: 'css-loader',
          options: {
            importloaders: 1, // 返回上一步进行处理
          }
        },
        'postcss-loader'] // 需要使用的loader， 顺序从后向前依次执行
      },
```

##### file-loader 

file-loader 默认使用 esModule 
1. 如果使用require 但是进行引入 那么就需要  `require('ddd').default`
2. 设置`file-loader => options  esModule: false`
3. 使用`import img from ’./dd‘` 进行使用

```js
 {
    test: /\.(png|svg|gif|jpe?g)$/,
    use:[{
      loader: 'file-loader',
      options: {
        esMoule: false
      }
    }]
  }

```

###### 配置文件名称
- [ext] 文件后缀类型
- [name] 文件名
- [hash] 文件hash值，webpack 每次打包都会产生一个hash 值
- [contenthash] 文件内容hash
- [hash：length] 文件hash长度

```js
...
{
    test: /\.(png|svg|gif|jpe?g)$/,
    use:[{
      loader: 'file-loader',
      options: {
        esModule: false, // 不转化为 esModule
        name: '[name].[hash:10].[ext]',
        outputPath: 'img'
      }
    }]
  }
...
```

##### url-loader 

使用
```js
...
{
    test: /\.(png|svg|gif|jpe?g)$/,
    use:[{
      loader: 'url-loader',
      options: {
        name: '[name].[hash:10].[ext]',
        limit: 8 * 1024 // 下小于 8kb 转base64
      }
    }]
  }
...
```

url-loader 和 file-loader 之间的区别
1. url-loader 默认转为base64 url的方式加载到文件中，减少请求次数
2. file-loader 将资源拷贝到指定的目录，分开请求。
3. limit 设置文件最大 限制，超过就拷贝

#### webpack 5 使用 asset 处理图片

webpack 5 中内置 asset 模块
1. asset/resource ----> file-loader   拷贝
2. asset/inline ---- >  url-loader
3. asset/source  ----> raw-loader

``` js
{
    test: /\.(png|svg|gif|jpe?g)$/,
    type: 'asset/resource',
    generator: {
      filename: 'img/[name].[hash:10][ext]'
    }
  }
```
设置asset 值
``` js
{
    test: /\.(png|svg|gif|jpe?g)$/,
    type: 'asset',
    generator: {
      filename: 'img/[name].[hash:10][ext]'
    },
    parser: {
      dataUrlCondition: {
        maxSize: 30 * 1024
      }
    }
  }
```

###### asset 处理 图标 字体

对数据进行拷贝

```js
{
  test: /\.(ttf|woff2?)$/,
  type: 'asset/resource',
  generator: {
    filename: 'font/[name].[hash:6][ext] '
  }
}
```

##### loader vs plugins

1. loader: 对特定类型数据进行转换
2. plugins： 做更多的事情， 可以在webpack 任意生命周期中做事情

### webpack plugins

#### clean-webpack-plugin
使用 clean-webpack-plugin 打包时自动删除 文件

安装
```
npm i clean-webpack-plugin -D
```

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

···
plugins: [
    new CleanWebpackPlugin()
  ],
···
```
#### html-webpack-plugin
使用 html-webpack-plugin 打包时自动删除 文件

安装
```
npm i html-webpack-plugin -D
```

```js
const  HtmlWebpackPlugin  = require('html-webpack-plugin')

···
plugins: [
     new HtmlWebpackPlugin({
      title: 'webpack ',
      template : "./public/index.html"
    })
  ],
···
```

